# content-decider

The IPFS protocol is strictly content-neutral and does not make judgments about what types of data people should be able to store or share. However, IPFS should also support users' ability to make informed decisions about which types content they choose to store or serve. These decisions might be based on personal preferences or legal requirements – whatever the reasons, this proposal aims to make it easy to:

- create a standardized format for arbitrary lists of content some set of node operators might prefer not to store or serve (`denylists`)
- check files against a `content-decider`, which is essentially service operating as a a denylist aggregator, before deciding whether to add the files to your repo

A `denylist` could be a list of anything, and `denylists` are generated and maintained by users, not by the IPFS project. Opting into any `denylist` is purely voluntary.

## The content-decider specification

Denylists are simple lists of denied content. It's a Bad Idea to maintain a central list of CIDs of bad content, so denylist entries are instead the sha256 multihash\* _(ID: implementation detail, you tell me)_ of the denied content's CID.

> TODO: Discuss more optimized data structures for denylists

A node operator may add one or more `content-decider`s to a list stored in `~/.ipfs/config.Experimental.ContentDeciders`. `content-deciders` are added as objects:

```
{
  MultiAddr: <address to send messages>,
  Timeout: <int, optional, ms to wait for a response, default 5000>,
  DefaultAllowed: <bool, optional, how to handle a service failure, default false>
}
```

At a practical level, a `content-decider` is a persistent process that tells you whether you should add a file to your repo. A `content-decider` makes decisions about content by receiving messages in the format:

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

> TODO: Example of a simple `content-decider` process.

When one or more `content-deciders` is specified, IPFS automatically adds a pre-execution hook to verify any proposed CID with each `content-decider` before adding the file to the repo.

> TODO: Figure out where in IPFS this should live.

## Updating denylists

As additional relevant content is discovered, a `denylist` may be updated. `content-deciders` should implement a strategy for periodic updates.

> TODO: Figure out efficient strategy for denylist updates.

> TODO: Figure out efficient strategy for checking all files in a repo against updated denylists.

## Further standardization

In the future, it may be desirable to further standardize the interface for denylist operations so they can be added directly to IPFS without the user running a separate process. For example, a user might run `ipfs denylist add pasta_sauce_recipes_that_use_sugar` and the IPFS daemon would transparently handle fetching and updating the relevant `denylist` and checking content against it.
