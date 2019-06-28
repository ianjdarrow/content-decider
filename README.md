# content-decider

The IPFS protocol is strictly content-neutral and does not make judgments about what types of data people should be able to store or share. However, IPFS should also support users' ability to make informed decisions about which types content they choose to store or serve. These decisions might be based on personal preferences or legal requirements – whatever the reasons, this proposal aims to make it easy to:

- subscribe to lists of content (`denylist`s) you prefer not to store or serve
- check files against a `content-decider`, which is essentially a denylist aggregator, before deciding whether to add the files to your repo

A `denylist` could be a list of anything, and denylists are generated and maintained by users, not by the IPFS project. Opting into any `denylist` is purely voluntary.

## The content-decider specification

Denylists are, unsurprisingly, lists of denied content. It's a Bad Idea to maintain a central list of CIDs of bad content, so denylist entries are instead the sha256 multihash\* _(ID: implementation detail, you tell me)_ of the denied content's CID.

A node operator may add one or more `content-decider`s to a list stored in `~/.ipfs/config.Experimental.ContentDeciders`. Each `content-decider` is an object:

```
{
  MultiAddr: <address to send messages>,
  Timeout: <int, optional, ms to wait for a response, default 5000>,
  DefaultAllowed: <bool, optional, how to handle a service failure, default false>
}
```

`content-deciders` make decisions on content by receiving messages in the format:

```
{
  HashedCID: <sha256 multihash of CID>
}
```

and returning messages in the format

```
{
  Allowed: <bool>,
  StatusCode: <int, optional, HTTP status code mapping to decision>,
  Reason: <string, optional, narrative description of decision>
}
```

> TODO: Additional detail on message formats.

> TODO: Example of a simple `content-decider` service.

When one or more `content-deciders` is specified, IPFS automatically verifies each CID with each `content-decider` before adding the file to the repo.

## Updating denylists

As additional relevant content is discovered, a `denylist` may be updated. `content-deciders` should implement a strategy for periodically updating

> TODO: Figure out efficient strategy for denylist updates.

> TODO: Figure out efficient strategy for checking all files in a repo against updated denylists.

## Further standardization

In the future, it may be desirable to further standardize the interface for denylist operations so they can be added directly to IPFS without the user running a separate service. For example, a user might run `ipfs denylist add pasta_sauce_recipes_that_use_sugar` and the IPFS daemon would transparently handle fetching and updating the relevant `denylist` and checking content against it.
