# Understanding Lock Value Relationships

When talking about the ownership of a cell, a lot of different terms may be used to describe an account. We've used the terms Private Key, Public Key, Lock Arg, Lock Hash, Lock Script, and Address. All of these are used for different purposes in different places to describe ownership, and all of these terms are related to each other.

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

A lock hash is a lock script which has been serialized into a binary serialization format using the Molecule library, then hashed using a 256-bit Blake2b hash.

#### CKB Address

An address is a special encoded value that specifies both an identity and how it should be accessed. It also includes a checksum value so it cannot be typed incorrectly. [CKB addresses](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0021-ckb-address-format/0021-ckb-address-format.md) have many possible uses which we will cover later. For now, think of it as an encoded form of the lock arg.

#### Component Relationship

Each one of these components is derived from the previous component in a way that is cryptographically provable. As a developer, you will be working with lock args and CKB addresses often. The important thing to remember is that both of these are identifying values that are derived from a public key, which means they can be used with a locking mechanism that can be unlocked using the private key. 

![](../.gitbook/assets/account-components-1.png)

### 

