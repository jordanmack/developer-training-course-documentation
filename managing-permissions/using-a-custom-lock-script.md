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

The code hash is describing the code that should execute, but it does not indicate where the code is located on the blockchain. To do this, we need to use a cell dep.

### Cell Deps

We already learned about input cells and output cells, but there is a third type called cell deps. Short for cell dependencies, cell deps are similar to input cells, but they are not consumed. They can be used repeatedly by many scripts as a read-only component of the transaction.

Some of the common uses of cells deps are:

* Script Code - A lock script references the `code_hash` of the code it needs to execute, but it still needs to know where the code resides. This is done by specifying a live cell that contains this code as a cell dep.
* Script Libraries - Just like a library for a normal application, a script library contains commonly used code in scripts.
* State Data - A cell can contain any data, including state data for a smart contract. Data from an oracle is a good example. The data published by the oracle is read-only and can be utilized by many smart contracts that rely on it.

With the addition of cell deps we now have a complete path from the transaction to the code.

![](../.gitbook/assets/transaction-connections-2.png)

Live Cell \#1 has a lock script with a `code_hash` that matches the data in Live Cell \#2. The data in Live Cell \#2 is a RISC-V binary that contains the logic needed to determine if a cell should unlock in a transaction.

### Using the Always Success Lock in Lumos

Open the `index.js` file from the `Using-a-Custom-Lock-Script-Example` folder. If you scroll down to the `main()` function, you will see that there four main sections.

![](../.gitbook/assets/example-flow.png)

1. Initialize - There are the first three lines of code in `main()`. We initialize the Lumos configuration, start the Lumos Indexer, and initialize the lab environment.
2. Deploy Code - The `deployAlwaysSuccessBinary()` function creates a cell with the contents of the RISC-V binary located in the file `./files/always_success`. This is the always success lock, an on-chain script that always grants permission to the cell in any transaction.
3. Create Cell - The `createCellWithAlwaysSuccessLock()` function creates a cell that uses the always success lock.
4. Consume Cell - The `consumeCellWithAlwaysSuccessLock()` function consumes the cell with the always success lock that we just created.

Let's go through the `deployAlwaysSuccessBinary()` function. Some of the code at the beginning and end is redundant from previous topics, so we will only cover the relevant code.

```javascript
// Create a cell with data from the specified file.
const {hexString: hexString1, dataSize: dataSize1} = await readFileToHexString(dataFile1);
const outputCapacity1 = ckbytesToShannons(61n) + ckbytesToShannons(dataSize1);
const output1 = {cell_output: {capacity: intToHex(outputCapacity1), lock: addressToScript(address1), type: null}, data: hexString1};
transaction = transaction.update("outputs", (i)=>i.push(output1));
```

This code should look familiar since we've used it several times before. We're creating a cell that contains the contents of the always success binary.



![](../.gitbook/assets/transaction-connections-2.png)

```javascript
// Return the out point for the always success binary so it can be used in the next transaction.
const outPoint =
{
	tx_hash: txid,
	index: "0x0"
};
return outPoint;
```





