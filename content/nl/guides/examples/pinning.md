---
title: Pinning Files
menu:
  guides:
    parent: examples
weight: 7
beta_equivalent: how-to/pin-files
summary: A quick guide to pinning files
---

<div class="alert alert-info">
Our interactive tutorials help you learn about the the decentralized web by writing code and solving challenges:
<a class="button button-primary" href="https://proto.school/#/tutorials" role="button" target="_blank">Open Tutorials at ProtoSchool</a> &nbsp;<i class="fa fa-external-link-square-alt"></i>
</div>

Pinning is a very important concept in ipfs. ipfs semantics try to make it feel like every single object is local, there is no "retrieve this file for me from a remote server", just `ipfs cat` or `ipfs get` which act the same way no matter where the actual object is located. While this is nice, sometimes you want to be able to control what you keep around. Pinning is the mechanism that allows you to tell ipfs to always keep a given object local. ipfs has a fairly aggressive caching mechanism that will keep an object local for a short time after you perform any ipfs operation on it, but these objects may get garbage collected fairly regularly. To prevent that garbage collection simply pin the hash you care about. Objects added through `ipfs add` are pinned recursively by default.
```
echo "ipfs rocks" > foo
ipfs add foo
ipfs pin ls --type=all
ipfs pin rm <foo hash>
ipfs pin rm -r <foo hash>
ipfs pin ls --type=all
```

As you may have noticed, the first `ipfs pin rm` command didn't work, it should have warned you that the given hash was "pinned recursively". There are three types of pins in the ipfs world; direct pins, which pin just a single block, and no others in relation to it. recursive pins, which pin a given block and all of its children, and indirect pins, which are the result of a given blocks parent being pinned recursively.

A pinned object cannot be garbage collected, if you don't believe me try this:
```
ipfs add foo
ipfs repo gc
ipfs cat <foo hash>
```

But if foo were to somehow become unpinned...
```
ipfs pin rm -r <foo hash>
ipfs repo gc
ipfs cat <foo hash>
```

By [whyrusleeping](https://github.com/whyrusleeping), apostrophes fixed by [Noisytoot](https://noisytoot.org)
