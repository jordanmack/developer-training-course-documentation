# Unused Content

### 

A Cell is composed of four basic components: Capacity, Data, Lock Script, and Type Script.

* **Capacity** is the number of CKBytes held within the Cell. It's called capacity because it represents the amount of on-chain space that the Cell can occupy. If your Cell takes up 100 bytes of space, then it needs to have a capacity of at least 100, which means it must contain at least 100 CKBytes.
* **Data** is any kind of data you want to store in the on-chain state. This can be data in any form, but the two most common are smart contract binaries and smart contract data.
* **Lock Script** is a small program that determines who owns and has control over the Cell. The default Lock Script on Nervos uses the Secp256k1 algorithm, which is the same as Bitcoin and Ethereum. But unlike Bitcoin and Ethereum, a developer has complete control over the algorithms and can change them if needed.
* **Type Script** is a small program that validates state changes any time the Cell is included in a transaction. A Type Script is what enables small contract like functionality within the Cell Model.

### 

### 

### 

### Inputs and Outputs

Almost every transaction has at least one input and one output. A transaction is used to describe how state changes, and a cell is the most basic representation of a single piece of state on Nervos. An input cell is the current state, and an output cell is the resulting state after the transaction executes. Below is a basic example.

This transaction has a single input cell and a single output cell. The input cell has 100 CKBytes and is owned by Alice. The output cell has 100 CKBytes and is owned by Bob. Since an input cell describes the current state, and an output describes the resulting state, we can infer that this transaction is sending 100 CKBytes from Alice to Bob.

Now let's look at how this would appear in code.

```javascript
// Add the input cell.
const input = await getLiveCell(nodeUrl, alicesCell);
skeleton = addInput(skeleton, input);

// Add a change cell as our output.
let output = {cell_output: {capacity: ckBytesToShannons(100n), lock: addressToScript(bobsAddress), type: null}, data: "0x"};
skeleton = addOutput(skeleton, output);
```







### Advantages of Lock Scripts

There are several advantages to using Lock Scripts instead of the hardcoded authentication methods which are used in most other blockchains.

#### Flexibility

The developer is able to have complete control over how authentication is handled. When new cryptography standards emerge in the future, using them is as simple as including a new library in your codebase.

However, the flexibility of Lock Scripts goes far beyond new cryptography. We will cover more cases in future lessons that demonstrate how a Lock Script plays a critical role in more complicated dapps.

#### Scalability

The Cell Model allows for very high-performance script execution. This is possible because each Live Cell is a piece of immutable state that is effectively pre-sharded. Lock Scripts can be executed in parallel, leading to massive scalability on layer 1, and even higher on layer 2.

#### Recoverability

Cell Deps also provide a means of recovering a smart contract if a necessary dependency is consumed, removing it from the state. Cell Deps are never linked to a specific out point, so any resource can be securely redeployed as long as the original source code or binary is available.









### Lock Hash

A `lock_hash` is another common value that is used for identifying ownership of a Cell. A `lock_hash` is a Blake2b hash of the Lock Script data structure. A Lock Script contains `args`, `code_hash`, and `hash_type`. A Lock Script is less convenient to reference since all three of these values must be known. Since a `lock_hash` encompasses all three of these values, it serves the purpose of a single value identifier of Cell ownership.











### Lock Args

The Lock Script `args` can include any data needed to prove ownership. In the case of the Secp256k1 Lock Script, the `args` represent the associated Secp256k1 public key that owns the Cell. Specifically, it is formatted as a 256-bit \(32 byte\) Blake2b hash of the user's Secp256k1 public key, truncated to 160 bits \(20 bytes\). This is commonly referred to as the `lock_arg` and it is used often as a means of identifying an account.

The output from `account list` in `ckb-cli` shows the `lock_arg`.

![](.gitbook/assets/account-list.png)

Pay close attention to the `lock_arg` value of the first account. The `lock_arg` has a value of `0xc8328aabcd9b9e8e64fbc566c4385c3bdeb219d7` which is an exact match for the `lock_arg` values in the previous screenshot of the live cell. The `lock_arg` is the account that owns the Cell, and since it matches, we know that those Cells are owned by the first account listed in the above screenshot.







### Default Lock Script Cell Deps

The default Lock Script has a `code_hash` of `0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8`. This tells CKB-VM the script that it needs to match to execute, but it doesn't tell it where that Cell is. To do that, we must add a Cell Dep for a Live Cell that has the matching script code that we need.

```javascript
// Add the cell dep for the lock script.
skeleton = skeleton.update("cellDeps", (cellDeps)=>cellDeps.push(locateCellDep({code_hash: "0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8", hash_type: "type"})));
```

The code above is how we can quickly add a Cell Dep for the default Lock Script. You don't need to dig into this now. Just know that it is using Lumos to locate an out point for a Live Cell that contains the script code needed to execute the default Lock Script.

