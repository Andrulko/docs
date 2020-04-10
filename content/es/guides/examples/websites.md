---
title: IPFS for Websites
menu:
  guides:
    parent: examples
weight: 4
beta_equivalent: how-to/host-single-page-site
summary: A short guide to hosting your site on IPFS
---

### A short guide to hosting your site on IPFS

<div class="alert alert-info">
Our interactive tutorials help you learn about the the decentralized web by writing code and solving challenges:
<a class="button button-primary" href="https://proto.school/#/tutorials" role="button" target="_blank">Open Tutorials at ProtoSchool</a> &nbsp;<i class="fa fa-external-link-square-alt"></i>
</div>

**Note:** [@agentofuser](https://github.com/agentofuser/)'s "Complete Beginner's Guide to Deploying Your First Static Website to IPFS" (see the [writeup](https://interplanetarygatsby.com/ipfs-deploy/) and [repo](https://github.com/agentofuser/ipfs-deploy)) is another useful resource for learning more about hosting your site on IPFS.

#### Create your site

Assume you have a static website in a directory `mysite`.

In order to publish it as a site, [install ipfs](https://docs.ipfs.io/guides/guides/install/) and make sure your ipfs daemon is running:

```bash
$ ipfs daemon
```

Then add the directory with your website:

```bash
$ ls mysite
img index.html
$ ipfs add -r mysite
added QmcMN2wqoun88SVF5own7D5LUpnHwDA6ALZnVdFXhnYhAs mysite/img/spacecat.jpg
added QmS8tC5NJqajBB5qFhcA1auav14iHMnoMZJWfmr4k3EY6w mysite/img
added QmYh6HbZhHABQXrkQZ4aRRSoSa6bb9vaKoHeumWex6HRsT mysite/index.html
added QmYeAiiK1UfB8MGLRefok1N7vBTyX8hGPuMXZ4Xq1DPyt7 mysite/
```

The last hash, next to the folder name, `mysite/` is the one to remember, call it `$SITE_CID` for now.

You can test it out locally by opening `http://localhost:8080/ipfs/$SITE_CID` in a browser or with `wget` or `curl` from the command line.

To view it from another ipfs node, you can try `http://gateway.ipfs.io/ipfs/$SITE_CID` in a browser. This will work from a browser on another device, inside or outside the network where you added the site's file.

Those hashes are difficult to remember. Let's look at some ways to get rid of them.

#### Using a DNS TXT record as a shortcut

Assume you have the domain name `your.domain` and can access your registrar's control panel to manage DNS entries for it.

Create a DNS TXT record ([DNSLink](https://docs.ipfs.io/guides/concepts/dnslink/)), with the key `your.domain.` and the value `dnslink=/ipfs/$SITE_CID` where `$SITE_CID` is the value from the section above.

Once you've created that record, and it has propagated you should be able to find it.

```bash
$ dig +noall +answer TXT your.domain
your.domain.            60      IN      TXT     "dnslink=/ipfs/$SITE_CID"
```
Now you can view your site at `http://localhost:8080/ipns/your.domain`.

You can also try this on the gateway at `http://gateway.ipfs.io/ipns/your.domain`.

More questions about DNSLink? Check out the website for tutorials, examples, and FAQ: http://dnslink.io/

#### Using the Interplanetary Naming System

Each time you change your website, you will have to republish it, update the DNS TXT record with the new value of `$SITE_CID` and wait for it to propagate.

You can get around that limitation by using IPNS, the [InterPlanetary Naming System](https://docs.ipfs.io/guides/concepts/ipns/).

You might have noticed `/ipns/` instead of `/ipfs/` in the updated links in the previous section.

The IPNS  is used for mutable content in the ipfs network. It's relatively easy to use, and will allow you to change your website without updating the dns record every time.

To enable the IPNS for your content run the following command where `$SITE_CID` is the hash value from the first step.

```bash
$ ipfs name publish $SITE_CID
Published to $PEER_ID: /ipfs/$SITE_CID
```
You will need to note and save that value of `$PEER_ID` for the next steps.

Load the urls `http://localhost:8080/ipns/$PEER_ID` and `http://gateway.ipfs.io/ipns/$PEER_ID` to confirm this step.

Return to your registrar's control panel, change the DNS TXT record with the key of `your.domain` to `dnslink=/ipns/$PEER_ID`, wait for that record to propagate, and then try the urls  `http://localhost:8080/ipns/your.domain` and `http://gateway.ipfs.io/ipns/your.domain`.

**Note:** When using IPNS to update websites, assets may be loaded from two different resolved hashes while the update propagates. This may result in outdated URLs or missing assets until the update has completely propagated.

#### Pointing your.domain to IPFS

You now have a website on ipfs/ipns, but your visitors can't access it at `http://your.domain`.

What we can do is have requests for `http://your.domain` resolved by an IPFS gateway daemon.

Return to your registrar's control panel and add an A record with key of `your.domain` and value of the IP address of an ipfs daemon that listens on port 80 for HTTP requests (such as `gateway.ipfs.io`). If you don't know the IP address of the daemon you plan to use, you can find it using the command like:

```bash
$ nslookup gateway.ipfs.io
```
1. Note the IP addresses returned.
2. Create an A record for each IPv4 address (e.g. `209.94.90.1` for ipfs.io).
3. Create an AAAA record for each IPv6 address (e.g. use `2602:fea2:2::1` for ipfs.io).

Note: The ipfs.io gateway IP addresses won't change, so you can set them and forget them. If you use a custom gateway where you don't control the IP address and they could change you may need to re-check them periodically and update your DNS records if they do.

Visitors' browsers will send `your.domain` in the Host header of their requests. The ipfs gateway will recognize `your.domain`, look up the value of the DNS TXT for your domain, then serve the files in `/ipns/your.domain/` instead of `/`.

If you point `your.domain`'s A and AAAA record to the IP addreses of `gateway.ipfs.io`, and then wait for the DNS to propagate, then anyone should be able to access your ipfs-hosted site without any extra configuration at `http://your.domain`.

#### Using CNAMES

Alternatively, it is possible to use CNAME records to point at the DNS records of the gateway. This way, IP addresses of the gateway are automatically updated.

However you will need to change the key for the TXT record from `your.domain` to `_dnslink.your.domain`.

So by creating a CNAME for `your.domain` to `gateway.ipfs.io` and adding a `_dnslink.your.domain` record with `dnslink=/ipns/<your peer id>` you can host your website without explicitly referring to IP addresses of the ipfs gateway.

Happy Hacking!

By [Whyrusleeping](https://github.com/whyrusleeping) and [Emma Humphries](https://github.com/emceeaich)
