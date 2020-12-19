# Understanding Lock Scripts

Lock Scripts are one of the many powerful features that differentiate Nervos from most other blockchain platforms. A Lock Script is a small program \(called a script\), that is used to define ownership of a Cell. This program has the ability to fully examine the transaction it is included within. This gives the developer a tremendous amount of flexibility on how to manage access.

The default Lock Script is based on Secp256k1 cryptography, making it nearly identical to Bitcoin and Ethereum. This allows a Cell to be owned and unlocked by any user who possesses the private key. However, a Lock Script can do much more. A Cell can be owned by multiple people, or by a smart contract. A Cell can also be owned by no one, but unlock when specific conditions are met in a transaction.

### The Structure of a Lock Script

When we talk about a Lock Script, it's important to pay attention to the context. Lock Script can refer to the data structure that is defined within a Cell, or the underlying code which defines how the script operates. Right now, we're discussing the data structure.

Here is an image of the outputs from a `rpc get_transaction` request in `ckb-cli`.

![](../.gitbook/assets/get-transaction-outputs.png)

The `lock` defines the Lock Script, and it has three structural elements:

* `args` are short for arguments, and is data provided to the Lock Script similarly to how arguments can be passed to a normal command-line program.
* `code_hash` is a Blake2b hash of the code that defines the Lock Script. The code itself is a RISC-V binary executable that is stored on-chain. 
* `hash_type` is a value that is either `data` or `type`. This controls how the `code_hash` is used in a transaction and can be used in the process of upgrading smart contracts. We will cover more about the usage of `code_hash` at a later time.



