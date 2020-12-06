# Sending a Transaction with Code

### Lesson Introduction

In this lesson, we will introduce Nervos' Cell Model, and use it to generate a transaction using code. You will need the outpoints you verified from the last lab exercise, so make sure you have them handy.

### The Cell Model

In the last lab exercise, you may have noticed that the command you used to verify that your outputs is called `get_live_cell`. It's called this because, in Nervos' terminology, both inputs and outputs are canonically referred to as "Cells".

A Cell is the most basic structure needed to represent a single piece of state data. The design is inspired by Bitcoin's outputs, but Cells have more flexible functionality. Cells can be used to represent any kind of on-chain asset type on Nervos, such as tokens, NFTs, and wrapped assets.

![](../.gitbook/assets/cell-model.png)

Below are the terms used for the Nervos Cell Model. We will stick to the Nervos terminology going forward, but know that it is not uncommon for others to use Bitcoin terminology when speaking about Nervos.

| Bitcoin | Nervos Cell Model |
| :--- | :--- |
| Input | Input Cell |
| Output | Output Cell |
| Unspent Output | Live Cell |
| Spent Output | Dead Cell |
| Spend | Consume  |

A Cell can only be used as an input to a transaction a single time, just like we covered in the last lesson. A Live Cell is one that has not been used as an Input Cell and is available to be used. A Dead Cell is one that has already been consumed by using it as an Input Cell and is no longer available for use.

Every Cell has an owner, and an individual can own any number of Cells. In the illustration below, Alice, Bob, and Charlie each own several Cells with different balances.

* Alice has two Cells for a total of 300 CKBytes.
* Bob has two Cells for a total of 700 CKBytes.
* Charlie has four Cells for a total of 2,000 CKBytes.

![](../.gitbook/assets/cell-owners.png)

Let's say that Charlie wants to send 700 CKBytes to Alice. To create a transaction, the relevant Live Cells must be gathered for use as inputs in a process called Cell collection. Charlie has four Cells available that could be used, but none of them have enough to send Alice 700 CKBytes, so we will need to use multiple Cells.

![](../.gitbook/assets/charlie-transaction.png)

During Cell collection we needed at least 700 CKBytes to pay Alice, so we gathered two Cells to cover that amount. Our total input Cells contain 1,000 CKBytes, but Charlie only wants to send 700 CKBytes to Alice. This means Charlie needs to send 300 CKBytes back to himself as change.

After the transaction has confirmed, the Cells which were used as inputs will be consumed, and two new Cells will be created. The new Cells created in the transaction are outlined in red below. 

![](../.gitbook/assets/cell-owners-2.png)

### Thinking in Code

Next we will create a similar transaction in code. Open `index.js` from the `03-01` folder in the Developer Training Course repo you cloned from GitHub. If you don't have this, go back to the Lab Exercise Setup section for instructions on how to clone it from GitHub.

This code you're looking at will generate a transaction with one input and one output. We will be generating a real transaction on your CKB Dev Blockchain, but the code you see here is simplified for learning purposes. The input will be specified by one of two outpoints you verified in the last lab exercise. The output is a change cell that returns the CKBytes back to the same account, less the transaction fee. 

![](../.gitbook/assets/code-transaction.png)

You will see the code below near the top of the file.

```javascript
const nodeUrl = "http://127.0.0.1:8114/";
const privateKey = "0xd00c06bfd800d27397002dca6fb0993d5ba6399b4238b2f29ee9deb97593d2bc";
const address = "ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37";
const previousOutput =
{
	tx_hash: "0x0000000000000000000000000000000000000000000000000000000000000000",
	index: "0x0"
};
const txFee = 100_000n;
```

* The `nodeUrl` is the URL of the CKB Dev Blockchain which should be running locally.
* The `privateKey` is the key used to sign transactions, and is set to the first account which contains the genesis issued CKBytes.
* The `address` is the CKB address of the account being signed, and is set to the first account which contains the genesis issued CKBytes.
*  The `previousOutput` contains the outpoint of a live Cell to be used in this transaction. 
* The `txFee` is the amount of transaction fees to pay, in Shannons. There are 100,000,000 Shannons in a CKByte, just like there are 100,000,000 Satoshis in a Bitcoin.

We'll walk through each line of code to give a deeper explanation of what is happening. This first line initializes the lab environment. At the beginning of each lab we call this to simplify the setup of the environment. An empty transaction skeleton is returned for us to work with.

```javascript
// Initialize our lab and create a basic transaction skeleton to work with.
let {transaction} = await initializeLab(nodeUrl, privateKey);
```

This creates an input from a live Cell using the outpoint you specified, then adds it to the transaction.

```javascript
// Add the input cell to the transaction.
const input = await getLiveCell(nodeUrl, previousOutput);
transaction = addInput(transaction, input);
```

This creates a change Cell output with the same capacity as the input, less the TX fee. The `lock` defines who the owner of this newly created Cell will be. We will explain `type` and `data` in a later lesson.

```javascript
// Add an output cell.
let output = {cell_output: {capacity: input.cell_output.capacity - txFee, lock: addressToScript(address), type: null}, data: "0x"};
transaction = addOutput(transaction, output);
```

This signs the transaction using the private key. Signing the transaction authorizes the usage of any input Cells, which are owned by the private key.

```javascript
// Sign the transaction.
const signedTx = signTransaction(transaction, privateKey);
```

This prints the current transaction to the screen in an easy to read format.

```javascript
// Print the details of the transaction to the console.
describeTransaction(transaction.toJS());
```

This sends the signed transaction to the local CKB Dev Blockchain node and prints the resulting TX hash to the screen. If you watch your CKB node output in another terminal window you should see it confirm shortly after submission.

```javascript
// Send the transaction to the RPC node.
const result = await sendTransaction(nodeUrl, signedTx);
console.log("Transaction Sent:", result);
```

Next, scroll back up to the top. Change the `previousOutpoint` value to match the outpoint you verified at the end of the last lesson. You should have verified two outpoints. The outpoint you want is the one that is owned by the address `ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37` since that is the private key we are using. Hint: The `lock_arg` which you recorded can be used to match it with the address. Use the `ckb-cli` command `account list` to find out the `lock_arg` for the matching testnet address.

```javascript
const previousOutput =
{
	tx_hash: "0x0000000000000000000000000000000000000000000000000000000000000000",
	index: "0x0"
};
```

After you have updated the code with the proper outpoint, open up a terminal and execute the command `node index.js` from within the code directory. Your output should be similar. Record the TX hash since we will use it again later.

```javascript
$ node index.js

Inputs:
  - capacity: 1,713,808,107,881,380 Shannons
    lock: 0x32e555f3ff8e135cece1351a6a2971518392c1e30375c1e006ad0ce8eac07947
    type: null
    out_point: 0x0017950609aa557433a117eab807361fe3e21794f08fd29fa201fb005928bb3e-0x0
Outputs:
  - capacity: 1,713,808,107,781,380 Shannons
    lock: 0x32e555f3ff8e135cece1351a6a2971518392c1e30375c1e006ad0ce8eac07947
    type: null

Transaction Sent: 0xbdf6c1cbf69e97234aae29b6db4a1df107240cb478ff290c214d403b1dfbd94d
```

Within a few seconds your transaction should confirm. You can use the `ckb-cli` command below to check the status of the transaction. The transaction is confirmed once the status at the bottom of the output reads `status: committed`.

```text
rpc get_transaction --hash <TX_HASH>
```

Go back to the terminal where you ran the code, and try executing the code again. Can you guess what will happen before running it? Run the code again using the same command as before: `node index.js`.

You should get the following error:

```text
UnhandledPromiseRejectionWarning: Error: Live Cell not found at out point: 0x3a52afb04b91097c84ca287ce58f98c1a454a3aa53497fbdd0ad6cba4b66f43b-0x0
```

The reason we received this error is that the outpoint we specified in the code was used as an input the first time we ran it. Using a Live Cell as an input will consume it and transform it into a Dead Cell. This can only occur a single time, which is why we received that error when trying to use it again.

### Lab Exercise



