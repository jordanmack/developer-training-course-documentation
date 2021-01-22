# Using a Custom Lock Script

Lock Scripts are one of the many powerful features that differentiate Nervos from most other blockchain platforms. A Lock Script is a small on-chain program \(called a script\), that is used to define ownership of a cell. This program has the ability to fully examine the transaction it is included within. This gives the developer a tremendous amount of flexibility on how to manage access to cells.

The default Lock Script is based on Secp256k1 cryptography, making it nearly identical to Bitcoin and Ethereum. This allows a cell to be owned and unlocked by any user who possesses the private key. However, a Lock Script can do much more. A cell can be owned by a single person, by multiple people using a multi-sig lock script, by another script similar to a smart contract, or by no one.

When testing dapps and smart contracts, it is often convenient to test transactions using special Lock Scripts which always succeed \(unlock\) or always fail \(never unlock\) in any transaction. We will demonstrate how to create cells with these special locks, but first, we need to understand the structure of a Lock Script to do so.

### The Structure of a Lock Script

When we talk about a Lock Script, it's important to pay attention to the context. Lock Script can refer to the data structure that is defined within a cell, or the underlying code which defines how the script operates. Right now, we're discussing the data structure.

Here is an image of the output from a `rpc get_live_cell` request in `ckb-cli`.

![](../.gitbook/assets/get-live-cell.png)

In the above image, the `lock` defines the Lock Script and it has three structural elements:

* `args` are short for arguments. This is data provided to the Lock Script similarly to how arguments can be passed to a normal command-line program.
* `code_hash` is a Blake2b hash of the code that defines the Lock Script. The code itself is a RISC-V binary executable that is stored on-chain in another cell. 
* `hash_type` controls how the `code_hash` is used in a transaction. It is used in the process of upgrading smart contracts. We will cover more about the usage of `hash_type` in the next lesson.

The `code_hash` value of `0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8` is the default Lock Script which uses Secp256k1 cryptography. When a cell is included in a transaction, the binary code matching the `code_hash` will execute and verify that the proper Secp256k1 signature was provided.

To explain it another way, the `code_hash` value is a way of indicating what code should execute to determine if the cell should grant access and unlock. The `code_hash` value itself does not contain any logic to make this determination. It simply indicates what code should make that determination.

When the default Secp256k1 lock script \(`0x9bd7...cce8`\) executes, it makes that determination by checking if the proper credentials were provided using the Secp256k1 algorithm. To do this, it matches values from the Lock Args with values from the Witnesses. We will show exactly how this works later, but we will not need this for the always success and always fail lock scripts since they do not rely on those.

So far we know that a transaction has input cells, and each one of these inputs is referenced by an out point to a live cell. Each one of these cells has a lock script that indicates what code should execute using the code hash, but there is still something missing.

![](../.gitbook/assets/transaction-connections-1.png)

The code hash is describing the code that should execute, but it does not indicate where the code is located on the blockchain. To indicate   

### Cell Deps

We already learned about Input Cells and Output Cells, but there is a third type called Cell Deps. Short for Cell Dependencies, Cell Deps are similar to Input Cells, but they are not consumed. They can be used repeatedly by many scripts as a read-only component of the transaction.

Some of the common uses of Cells Deps are:

* Script Code - A Lock Script references the `code_hash` of the code it needs to execute, but it still needs to know where the code resides. This is done by specifying a Live Cell that contains this code as a Cell Dep.
* Script Libraries - Just like a library for a normal application, a script library contains commonly used code in scripts.
* State Data - A Cell can contain any data, including state data for a smart contract. Data from an oracle is a good example. The data published by the oracle is read-only and can be utilized by many smart contracts that rely on it.

### Default Lock Script Cell Deps

The default Lock Script has a `code_hash` of `0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8`. This tells CKB-VM the script that it needs to match to execute, but it doesn't tell it where that Cell is. To do that, we must add a Cell Dep for a Live Cell that has the matching script code that we need.

```javascript
// Add the cell dep for the lock script.
skeleton = skeleton.update("cellDeps", (cellDeps)=>cellDeps.push(locateCellDep({code_hash: "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8", hash_type: "type"})));
```

The code above is how we can quickly add a Cell Dep for the default Lock Script. You don't need to dig into this now. Just know that it is using Lumos to locate an out point for a Live Cell that contains the script code needed to execute the default Lock Script.

