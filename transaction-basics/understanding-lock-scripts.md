# Understanding Lock Scripts

Lock Scripts are one of the many powerful features that differentiate Nervos from most other blockchain platforms. A Lock Script is a small program \(called a script\), that is used to define ownership of a Cell. This program has the ability to fully examine the transaction it is included within. This gives the developer a tremendous amount of flexibility on how to manage access.

The default Lock Script is based on Secp256k1 cryptography, making it nearly identical to Bitcoin and Ethereum. This allows a Cell to be owned and unlocked by any user who possesses the private key. However, a Lock Script can do much more. A Cell can be owned by multiple people, or by a smart contract. A Cell can also be owned by no one, but unlock when specific conditions are met in a transaction.

### The Structure of a Lock Script

When we talk about a Lock Script, it's important to pay attention to the context. Lock Script can refer to the data structure that is defined within a Cell, or the underlying code which defines how the script operates. Right now, we're discussing the data structure.

Here is an image of the outputs from a `rpc get_transaction` request in `ckb-cli`.

![](../.gitbook/assets/get-transaction-outputs.png)

In the above image, the `lock` defines the Lock Script and it has three structural elements:

* `args` are short for arguments. This data provided to the Lock Script similarly to how arguments can be passed to a normal command-line program.
* `code_hash` is a Blake2b hash of the code that defines the Lock Script. The code itself is a RISC-V binary executable that is stored on-chain. 
* `hash_type` is a value that is either `data` or `type`. This controls how the `code_hash` is used in a transaction and can be used in the process of upgrading smart contracts. We will cover more about the usage of `code_hash` at a later time.

The `code_hash` value of `0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8` is the default Lock Script which uses Secp256k1 cryptography. When a Cell is included in a transaction, the binary code matching the `code_hash` will execute and verify that the proper Secp256k1 signature was provided.

### Lock Args

The Lock Script `args` can include any data needed to prove ownership. In the case of the Secp256k1 Lock Script, the `args` represent the associated Secp256k1 public key that owns the Cell. Specifically, it is formatted as a Blake2b hash of the user's public key, truncated to 160 bits \(20 bytes\). This is commonly referred to as the `lock_arg` and it is used often as a means of identifying an account.

The output from `account list` in `ckb-cli` shows the `lock_arg`.

![](../.gitbook/assets/account-list.png)

If you pay close attention, the `lock_arg` value of `0xc8328aabcd9b9e8e64fbc566c4385c3bdeb219d7` from the first account is a match for the `lock_arg` values in the previous screenshot of transaction outputs. Since the `code_hash` and `lock_arg` match, we know that those Cells are owned by the first account listed in the above screenshot.

### Lock Hash

A `lock_hash` is another common value that is used for identifying ownership of a Cell. A `lock_hash` is a Blake2b hash of the Lock Script data structure. A Lock Script contains `args`, `code_hash`, and `hash_type`, and all three of these values must be known to determine ownership. However, this is less convinient to reference. Since `lock_hash` is a hash of all three of these values, it serves the purpose of a single value identifier of Cell ownership.



