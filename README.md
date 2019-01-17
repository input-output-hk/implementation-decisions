# Implementation Decisions

A repository for all concrete decisions made about how to implement the cardano
blockchain. This repository contains all the technical details and improvements
that are needed for the developers. The repository contains git submodules that
pin down specific versions of the formal specifications which these concrete
technical decisions refer to. Changes to this need to be reflected in the
different documents.

The "RFC" (request for comments) process is intended to provide a consistent and
controlled path for new features to enter the blockchain and standard APIs, so
that all stakeholders can be confident about the direction the blockchain is
evolving in. It's also intended as an explicit list of concrete decisions that
are outside the scope of the abstract specifications, and therefore the weight
of the decision making should lie with the party making the proposal (who will
more than likely be involved in a concrete implementation at the time of making
it).

# When to write a proposal

Currently we only accept proposals and comments from within the IOHK
organization.

Proposals are to be made if:

1. The formal specification is too vague and a concrete decision needs to be
   taken. Example: it is said the we take an index `idx` from the set of `Idx`.
   However it doesn't specify wether it is acceptable to have negative indices
   or if it is bounded (32bits? 64bits?), of if it is from 0 or from 1. A
   proposal can be made to pin this down and guarantee that we are all talking
   about the same thing.
2. The current blockchain needs to include new features in order to implement the
   new feature defined by the formal spec. This needs to be documented here too.
3. An improvement to the current implementation needs to be made. For example
   one could propose an improvement to the way we serialize blocks.

# The process

1. fork this repository;
2. Copy <XXX-template.md> to `text/0000-my-feature.md` (where "my-feature" is
   descriptive. don't assign an RFC number yet).
3. Fill in the RFC;
4. Submit a pull request and fill in the `RFC PR:` section in
   <./XXXX-template.md>. As a pull request the RFC will receive design feedback,
   and the author should be prepared to revise it in response
5. The owners of this repository will orchestrate the review process. They will
   follow the comments and will direct the conversation toward consensus. If no
   consensus can be reached in a timely manner they may: close the PR or apply
   any changes they believe to be necessary before merging the PR.

# Guidelines for formal-specifications writers

Writers of formal-specifications referred in this document should check that
when these specifications are updated, all changes affecting `RFC`'s in this
repository should be accounted for. In practical terms this means that if a
change in the specifications affects a previous `RFC` the writer should:
- submit a new `RFC` which refers to the `RFC` that the changes in the spec is
  affecting.
- specify whether the previous `RFC` is rendered obsolete by the new `RFC`, or
  whether the latter is just an addendum to the former.
- update the version of the specification pinned in the git submodules.

# License

This repository is currently in the process of being licensed under either of

Apache License, Version 2.0, (LICENSE-APACHE or http://www.apache.org/licenses/LICENSE-2.0)
MIT license (LICENSE-MIT or http://opensource.org/licenses/MIT)
at your option.

# Contributions

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.
