---
title: Making Your Own IPFS Service
beta_equivalent: how-to/make-service
<!-- menu:
  guides:
    parent: examples
weight: >

summary: IPFS has a few default services that it runs by default, such as the dht, bitswap, and the diagnostics service. Each of these simply registers a handler on the IPFS PeerHost, and listens on it for new connections. The `corenet` package has a very clean interface to this functionality. So lets try building an easy demo service to try this out…
---

<div class="alert alert-info">
Our interactive tutorials help you learn about the the decentralized web by writing code and solving challenges:
<a class="button button-primary" href="https://proto.school/#/tutorials" role="button" target="_blank">Open Tutorials at ProtoSchool</a> &nbsp;<i class="fa fa-external-link-square-alt"></i>
</div>

IPFS has a few default services that it runs by default, such as the dht, bitswap, and the diagnostics service. Each of these simply registers a handler on the IPFS PeerHost, and listens on it for new connections.  The `corenet` package has a very clean interface to this functionality. So lets try building an easy demo service to try this out!

Lets start by building the service host:

```
package main

import (
    "fmt"

    core "github.com/ipfs/go-ipfs/core"
    corenet "github.com/ipfs/go-ipfs/core/corenet"
    fsrepo "github.com/ipfs/go-ipfs/repo/fsrepo"

    "code.google.com/p/go.net/context"
)
```

We don't need too many imports for this. Now, the only other thing we need is our main function:

Set up an ipfsnode.

```
func main() {
    // Basic ipfsnode setup
    r, err := fsrepo.Open("~/.ipfs")
    if err != nil {
        panic(err)
    }

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    cfg := &core.BuildCfg{
        Repo:   r,
        Online: true,
    }

    nd, err := core.NewNode(ctx, cfg)

    if err != nil {
        panic(err)
    }
```

That's just the basic template of code to initiate a default ipfsnode from the config in the users `~/.ipfs` directory.

Next, we are going to build our service.

```
    list, err := corenet.Listen(nd, "/app/whyrusleeping")
    if err != nil {
        panic(err)
    }

    fmt.Printf("I am peer: %s\n", nd.Identity.Pretty())

    for {
        con, err := list.Accept()
        if err != nil {
            fmt.Println(err)
            return
        }

        defer con.Close()

        fmt.Fprintln(con, "Hello! This is whyrusleepings awesome ipfs service")
        fmt.Printf("Connection from: %s\n", con.Conn().RemotePeer())
    }
}
```

And thats really all you need to write a service on top of IPFS. When a client connects, we send them our greeting, print their peer ID to our log, and close the session. This is the simplest possible service, and you can really write anything you want to handle the connection.

Now we need a client to connect to us:

```
package main

import (
    "fmt"
    "io"
    "os"

    core "github.com/ipfs/go-ipfs/core"
    corenet "github.com/ipfs/go-ipfs/core/corenet"
    peer "github.com/ipfs/go-ipfs/p2p/peer"
    fsrepo "github.com/ipfs/go-ipfs/repo/fsrepo"

    "golang.org/x/net/context"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Please give a peer ID as an argument")
        return
    }
    target, err := peer.IDB58Decode(os.Args[1])
    if err != nil {
        panic(err)
    }

    // Basic ipfsnode setup
    r, err := fsrepo.Open("~/.ipfs")
    if err != nil {
        panic(err)
    }

    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    cfg := &core.BuildCfg{
        Repo:   r,
        Online: true,
    }

    nd, err := core.NewNode(ctx, cfg)

    if err != nil {
        panic(err)
    }

    fmt.Printf("I am peer %s dialing %s\n", nd.Identity, target)

    con, err := corenet.Dial(nd, target, "/app/whyrusleeping")
    if err != nil {
        fmt.Println(err)
        return
    }

    io.Copy(os.Stdout, con)
}
```

This client will set up their IPFS node (note: this is moderately expensive and you normally wont just spin up an instance for a single connection) and dial the service we just created.

To try it out, run the following on one computer:
```
$ ipfs init # if you havent already
$ go run host.go
```

That should print out that peers ID, copy it and use it on a second machine:
```
$ ipfs init # if you havent already
$ go run client.go <peerID>
```

It should print out `Hello! This is whyrusleepings awesome ipfs service`

Now, you might be asking yourself: "Why would I use this? How is it better than the `net` package?". Well, here are the advantages:

1. You dial a specific peerID, no matter what their IP address happens to be at the moment.
2. You take advantage of the NAT traversal built into our net package.
3. Instead of a 'port' number, you get a much more meaningful protocol ID string.

By [whyrusleeping](http://github.com/whyrusleeping)
