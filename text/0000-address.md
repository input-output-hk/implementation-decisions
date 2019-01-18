- Feature Name: address
- Start Date: 2019-01-17
- RFC PR: (leave this empty)
- Authors:
  - Nicolas Di Prima (nicolas.diprima@iohk.io)
  - Vincent Hanquez (vincent.hanquez@iohk.io)

# Summary
[summary]: #summary

A new address format for the cardano blockchain that exposes extendable
properties as well as being as compact as possible.

The new proposed address format exposes the following properties:

1. compact on the blockchain;
2. extendable within reason;
3. human readable encoding
  * that includes visual discriminant;
  * with checksum to detect errors;

# Motivation
[motivation]: #motivation

The current address model is wasting quite a lot of space. It is a large
CBOR structure and does not distinguish what is necessary for the blockchain
and what is necessary for the human readability. This RFC fixes this by
proposing a much simpler address format in the sense that it is extendable
within reason.

1. the _compact-ability_ of the address on the blockchain is a prime motivation
   of this proposal. Because of the model of the blockchain, every byte ever
   included in the ledger will be stored there for-ever. We need to ensure that
   we store only what is necessary. On the previous address encoding, the cbor
   model allowed for including unbounded byte array in the address which has been
   leverage to include random data;
2. Another motivation to keep the address as short as possible is that it will
   save on transaction fees too (the current fee algorithm being linear in number
   of bytes);
3. The new address model allows to up to 123 different types of addresses. The
   current RFC only proposes 2 models: simple and grouped.
4. we also propose a new human readable encoding. This will allow visual
   discrimination for the user between the old address format and the new one.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

An `Address` is a mean to associate a token to a cryptographic key pair.
In order for Bob to send a token to Alice, Alice needs to generate a key
pair, a `PublicKey` and a `PrivateKey`. Alice generates an `Address` from
this `PublicKey` and sends the `Address` to Bob and Bob adds on the Ledger
that he has sent a token to the public key.

To create a new address, we only need the following information:

* `PublicKey`: this will be necessary to be able to utilise the token;
* `Discriminant`: this will prevent misuse of addresses between testing
  environment and production environment;

This is the most simple address. It is called a **Single Address**. This is
because it is not associated with any staking key.

It is possible to create a **Grouped Address**, it groups addresses to a
staking key. To create a **Grouped Address** you need the `PublicKey`,
the `Discriminant` (just like for a **Single Address**) and another
`PublicKey` that is associating this address to another hierarchy.

The human readable encoding of the `Address` is [bech32]. It includes
a checksum for correctness (preventing mistyping the address). There
is also a visual discriminant for human readable discrimination between
cardano (production) addresses and testing version of the blockchain:

* `ta` is the visual discriminant for **Testing** address;
* `ca` is the visual discriminant for **Cardano** address (production address);

So to create a **Single Address** we need a public key.

```
PublicKey = 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20
```

A production **Single Address** would look like:
`ca1qvqsyqcyq5rqwzqfpg9scrgwpugpzysnzs23v9ccrydpk8qarc0jqxuzx4s`. Note the `ca` at
the beginning of the `Address`. This is our visual discriminant.

To create a **Grouped Address** we need an additional grouping key:

```rust
GroupKey = 292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f404142434445464748
```

A production **Grouped Address** would look like:
`ca1qsqsyqcyq5rqwzqfpg9scrgwpugpzysnzs23v9ccrydpk8qarc0jq2f29vkz6t30xqcnyve5x5mrwwpe8ganc0f78aqyzsjrg3z5v36gguhxny`.
Note the `ca` prefix again.

A test **Grouped Address** would look like:
`ta1ss5j52ev95hz7vp3xgengdfkxuurjw3m8s7nu06qg9pyx3z9ger5sqgzqvzq2ps8pqys5zcvp58q7yq3zgf3g9gkzuvpjxsmrsw3u8eqx5x7xh`.
Note the `ta` prefix here.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Binary serialization

It uses a simple serialization format which is made to be concise:

* First byte contains the discrimination information (1 bit) and the kind of address (7 bits)
* Remaining bytes contains a kind specific encoding describe after.

2 kind of address are currently supported:

* Single: Just a (spending) public key using the ED25519 algorithm
* Group: Same as single, but with a added (staking/group) public key
  using the ED25519 algorithm.

```
Single key:
    DISCRIMINATION_BIT || SINGLE_KIND_TYPE (7 bits) || SPENDING_KEY

Group key:
    DISCRIMINATION_BIT || GROUP_KIND_TYPE (7 bits)|| SPENDING_KEY || STAKING_KEY
```

Bit value for the `DISCRIMINATION_BIT`:

| value | purpose |
|---:|:--- |
| 0 | Production Address |
| 1 | Test Address |

Address kind:

| value | purpose |
|---:|:--- |
| 0 | reserved |
| 1 | reserved |
| 2 | reserved |
| 3 | Single Kind | followed by 32 bytes of an Ed25519 `PublicKey` |
| 4 | Group Kind  | followed by 32 bytes of an Ed25519 `PublicKey` and 32 bytes of an Ed25519 `PublicKey` |

Using the example given in [guide-level-explanation], a **Single Address** for
the public key:

```
PublicKey = 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20
```

would be encoded as follow:

|discriminant | first byte | public key |
|---:|:---|:--|
| Production | `0x03` (`0b0000_0011`) | `0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20` |
| Testing    | `0x83` (`0b1000_0011`) | `0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20` |

and a **Grouped Address** with the group key:

```
GroupKey = 292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f404142434445464748
```

|discriminant | first byte | public key | Group Key |
|---:|:---|:--|:--|
| Production | `0x04` (`0b0000_0100`) | `0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20` | `292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f404142434445464748` |
| Testing    | `0x84` (`0b1000_0100`) | `0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20` | `292a2b2c2d2e2f303132333435363738393a3b3c3d3e3f404142434445464748` |

For the Human readable encoding, we will use [bech32] (see [BIP-173]
for the implementation details).

## Bech32 for human readable encoding

See [bech32].

# Drawbacks
[drawbacks]: #drawbacks

N/A

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Current addresses are in average about 74 bytes long (and this is without the
stake key).

* a hash with a given 28 bytes size (`Blake2b-224`),
* a HDPayload of variable size, containing a:
  * 16 bytes poly1305 authentication tag
  * an encrypted CBOR encoded array of 32 bits derivation indices
* stake distribution, either:
  * nothing when using bootstrap era
  * a single key distribution, a hash with a given 32bytes size (`Blake2b-256`)
* Address type Tag
* 4 bytes of crc32
* 4 bytes of address discrimination

On the blockchain there is no reason to store the crc32. This is only
useful for the human readable encoding part to prevent mistyping.
There is also no need to have more than 1 bit of address discrimination
on the blockchain: the signature protocol takes into account the
protocol magic (the blockchain discriminant).

The new format only contains:

* 1 bytes of metadata:
  * 1bit of address discrimination (production/testing);
  * 7bits of address type (allowing up to 123 different types);
* 32 bytes of public key;
* 32 bytes of stake distribution (for non bootstrap era)

Smallest is 33bytes. Longest is 65bytes.

# Reference implementation
[reference-implementation]: #reference-implementation

* **Rust**: [chain-addr](https://github.com/input-output-hk/rust-cardano/blob/master/chain-addr/src/lib.rs)

# Prior art
[prior-art]: #prior-art

* [bec32] comes from [BIP-173];

# Unresolved questions
[unresolved-questions]: #unresolved-questions

# Future improvements
[future-improvements]: #future-improvements

* extend the enum to allow for multisig (will need a new RFC);

<!-- ################################################################## -->
[BIP-173]: https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki
[bech32]: https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki
