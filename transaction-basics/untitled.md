# Introduction to Lumos

Lumos is a framework for building dapps. It is aimed at the backend side of dapp development and is very useful for creating transactions and interacting with the blockchain.

Open the `index.js` file from the `03-01` folder in the Developer Training Course repo you cloned from GitHub. If you don't have this, go back to the Lab Exercise Setup section for instructions on how to clone it from GitHub.

This code in `index.js` will generate a basic transaction with one input and one output. We will be generating a real transaction on your CKB Dev Blockchain, but the code you see here is simplified to make it easier to follow.

The Input Cell that the code uses will be specified by one of two out points you verified in the last lab exercise. The output that is created is a Change Cell that returns the CKBytes back to the same account, minus the transaction fee. 

![](../.gitbook/assets/code-transaction.png)

Starting at the top of the file, we have the includes.

```javascript
const {addressToScript} = require("@ckb-lumos/helpers");
const {addInput, addOutput, describeTransaction, getLiveCell, initializeLab, sendTransaction, signTransaction} = require("./lab.js");
```

We have an include from Lumos, but most are from our lab.js library. To keep things more easy to follow, lab.js abstracts out some of the more complex functionality of Lumos. As we get more familiar with Lumos, we will slowly introduce more functionality.

Next, you will see a group of variables, which we will explain.

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

* The `nodeUrl` variable is set to the URL of the CKB Dev Blockchain you set up in the Lab Exercise Setup section.
* The `privateKey` variable is set to the key used to sign transactions. This is the private key for the account `ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37` which contains some genesis issued CKBytes. You may recognize this address from when you executed `account list` in `ckb-cli`.
* The `address` variable is set to the CKB address of the account being signed, and is set to the same account as the `privateKey`.
*  The `previousOutput` variable will be set to the out point of a live Cell to be used in this transaction. 
* The `txFee` variable is the amount of transaction fee to pay, in Shannons. There are 100,000,000 Shannons in a CKByte, just like there are 100,000,000 Satoshis in a Bitcoin.

We'll walk through each line of code to give a deeper explanation of what is happening. This first line initializes the lab environment.

```javascript
// Initialize our lab and create a basic transaction skeleton to work with.
let {transaction} = await initializeLab(nodeUrl, privateKey);
```

The `initializeLab` function is something we use on lab exercises, but it would never be used in a production environment. It sets up each lab in a way that we can focus specifically on the relevant code. In this lab, it returns a transaction skeleton for us to work with.

This creates an input from a live Cell using the out point you specified in the `previousOutput` variable, then adds it to the transaction.

```javascript
// Add the input cell to the transaction.
const input = await getLiveCell(nodeUrl, previousOutput);
transaction = addInput(transaction, input);
```

This creates an output for a change Cell with the same capacity as the input, minus the TX fee. The `lock` defines who the owner of this newly created Cell will be, and that is defined with the `address` variable We will explain `type` and `data` in a later lesson.

```javascript
// Add an output cell.
let output = {cell_output: {capacity: input.cell_output.capacity - txFee, lock: addressToScript(address), type: null}, data: "0x"};
transaction = addOutput(transaction, output);
```

This prints the current transaction to the screen in an easy to read format.

```javascript
// Print the details of the transaction to the console.
describeTransaction(transaction.toJS());
```

This signs the transaction using the private key specified in the `privateKey` variable. Signing the transaction authorizes the usage of any Input Cells that are owned by that private key.

```javascript
// Sign the transaction.
const signedTx = signTransaction(transaction, privateKey);
```

This sends the signed transaction to the local CKB Dev Blockchain node and prints the resulting TX hash to the screen. If you watch your CKB node output in another terminal window you should see it confirm shortly after submission.

```javascript
// Send the transaction to the RPC node.
const result = await sendTransaction(nodeUrl, signedTx);
console.log("Transaction Sent:", result);
```

Now scroll back up to the top. We need to change the `previousOutpoint` value to match one of the out points you verified at the end of the last lesson. You should have verified two out points. The out point you want is the one that is owned by the address `ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37` since that is the private key we are using. Hint: The `lock_arg` which you recorded can be used to match it with the address. Use the `ckb-cli` command `account list` to find out the `lock_arg` for the matching testnet address. We will cover the purpose of what a `lock_arg` is in the next lesson.

```javascript
const previousOutput =
{
	tx_hash: "0x0000000000000000000000000000000000000000000000000000000000000000",
	index: "0x0"
};
```

After you have updated the code with the proper out point, open up a terminal and execute the command `node index.js` from within the code directory to run the code. Your output should be similar to that below. Record the TX hash since we will use it again later.

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

Within a few seconds, your transaction should confirm. You can use the `ckb-cli` command below to check the status of the transaction. The transaction is confirmed once the status at the bottom of the output reads `status: committed`.

```text
rpc get_transaction --hash <TX_HASH>
```

Go back to the terminal where you ran the code, and try executing the code again. Can you guess what will happen before running it? Run the code again using the same command as before: `node index.js`.

You should get the following error:

```text
UnhandledPromiseRejectionWarning: Error: Live Cell not found at out point: 0x3a52afb04b91097c84ca287ce58f98c1a454a3aa53497fbdd0ad6cba4b66f43b-0x0
```

The reason we received this error is that the out point we specified in the code has already been used. Using a Live Cell as an input will consume it and transform it into a Dead Cell. This can only occur a single time, which is why we received that error when trying to use it again.

