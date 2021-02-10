# Understanding Lock Value Relationships

When talking about the ownership of a cell, a lot of different terms may be used to describe an account. We've used the terms Private Key, Public Key, Lock Script, Lock Arg, Lock Hash,  and Address. All of these are used for different purposes in different places to describe ownership, and all of these terms are related to each other.

Nervos allows developers to use any cryptography desired to secure accounts, but the default lock uses the [Secp256k1](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) algorithm in tandem with the [Blake2b](https://en.wikipedia.org/wiki/BLAKE_%28hash_function%29#BLAKE2) hashing algorithm. For the terms listed on this page, we will describe them from the context of the default lock 

#### Private Key

A Secp256k1 private key is a randomly generated sequence that is used to secure your accounts. The private key length is 256 bits long \(32 bytes\). A private key must be kept secret at all times because this is what is used to unlock funds.

#### Public Key

A Secp256k1 public key 256 bits in length and is derived from the private key. The public key does not need to be kept a secret, and it is used as a form of identifier.

It's important to understand what a public key is and what it's used for, but most of the time you will be working with values that are derived from the public key, rather than the public key itself.

#### Lock Script

A lock script can have two different meanings depending on the context. A lock script can refer to the data structure within a cell that indicates the code which should be executed to determine ownership of the cell, or it can refer to the code which is actually executed. Right now, we are talking about the data structure.

The lock script contains the `code_hash`, `hash_type`, and `args` fields. The `code_hash` and `hash_type` indicate the lock code that will be executed to determine ownership. The `args` indicates who the owner is.

#### Lock Arg

A lock arg is a 256-bit Blake2b hash of the public key, truncated to 160 bits \(20 bytes\). It's called a "lock arg" because it is most commonly used as the value that is placed into the `args` field of the lock script to indicate the owner of the cell.

The lock arg is the most common ownership identifier that is used on-chain, because it is used with both the default lock and the multi-sig lock. It is also commonly used as an account identifier in tools such as `ckb-cli`.

#### Lock Hash

A lock hash is a lock script data structure which has been serialized into binary using the [Molecule](https://github.com/nervosnetwork/molecule) library, then hashed with Blake2b. A lock script has three fields, and the combination of those three fields are used to determine ownership. A lock hash is a single value that can be used to represent all three field values in a lock script.

A lock hash is another common ownership identifier that is used frequently in dapp development, and occasionally with tools like `ckb-cli`.

#### CKB Address

An address is a lock script data structure that has been [encoded](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0021-ckb-address-format/0021-ckb-address-format.md) in a special format that saves space and includes a checksum to prevent accidental typing mistakes. Similar to a lock hash, an address is a single value that represents all three fields of a lock script. Unlike a lock hash, and address is reversible back to lock script form.

Addresses are most commonly used to represent accounts using the default lock and multi-sig lock, but they are fully capable of being used with any lock. Since all three fields of the lock script are represented, the address format can be used with any form of cryptography available today, or in the future.

Addresses are the most common ownership identifier used by end-users in both tooling and within dapps.

#### Component Relationship

As a developer, you will be working with these various formats and structures regularly, so it's important to understand their relationships with each other.

![](.gitbook/assets/lock-value-relationships.png)

This image illustrates how each of these values are related to each other. The generation of each value is one way, meaning you cannot get the original value from the derived value. The exception is the address format, which is specifically designed to be reversible back to a lock script.

