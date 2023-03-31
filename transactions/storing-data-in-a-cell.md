# Storing Data in a Cell

A cell can be used to store any kind of data that is desired by the developer. However, the cost of storing state data on a globally decentralized blockchain is high when compared to normal or cloud-based storage. This means that some types of data are more appropriate than others.

| Typically Appropriate         | Typically Inappropriate |
| ----------------------------- | ----------------------- |
| Script Code (Smart Contracts) | Images                  |
| Dapp State Data               | Movies                  |
| Token Balances                | Music                   |
| Oracle Data                   | PDFs                    |
| Data Hashes (Anchoring)       |                         |

### Understanding the Costs of State Storage

Bitcoin often uses the comparison of blockchain storage to prime real estate. This comparison is also used for Nervos, but in an even stronger sense. Possessing one CKByte gives the holder the right to store one byte of data in the blockchain state. This makes a CKByte similar to a real estate deed.&#x20;

In order to store one megabyte of data on Nervos, you would need to hold 1,048,576 CKBytes. This makes the storage of large data prohibitively expensive for large files. Notice that I said _hold_ CKBytes and not _pay_ CKBytes. This is because those CKBytes can be reclaimed once the data is removed from the state.

CKBytes are used to pay state rent while a cell occupies blockchain state. A cell must have at least enough capacity (CKBytes) for the space the cell occupies in the blockchain state, including all data within it. The CKBytes are effectively locked in the cell until the cell is consumed. During this time, the CKBytes locked in a cell are subject to targeted inflation. This inflation pays the state rent indirectly, requiring no action from the owner of the cell.

When CKBytes are locked they are ineligible to use the NervosDAO, which pays users interest on their CKBytes. This interest is equal to the inflation which pays the state rent, effectively negating it, and making the CKByte a deflationary currency similar to Bitcoin.

For more information on this topic, you can read the [Crypto-Economic White Paper](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0015-ckb-cryptoeconomics/0015-ckb-cryptoeconomics.md). This is optional but will give much deeper insight into how the economics of Nervos work.

### Methods of Storing Data

There are three main methods of storing data in a cell:

* Using `ckb-cli` is convenient for one-off files.
* Using Lumos Framework is the most common way for dapps to generate cells with data.
* Using Capsule Framework is a common way to deploy scripts created with Capsule.

Under the hood, all three methods ultimately rely on RPC calls to a ckb node, but it's much less common to interact directly with the RPC. We will demonstrate `ckb-cli` and Lumos now, and Capsule will be introduced later.

### Storing Data Using ckb-cli

Open a terminal and enter the top level of the `developer-training-course` folder. From there, execute the following command:

{% code overflow="wrap" lineNumbers="true" %}
```shell
ckb-cli wallet transfer --from-account ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqwgx292hnvmn68xf779vmzrshpmm6epn4c0cgwga --to-address ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqwgx292hnvmn68xf779vmzrshpmm6epn4c0cgwga --to-data-path "./files/HelloNervos.txt" --capacity 74
```
{% endcode %}

This will create a new cell that contains the contents of the file `HelloNervos.txt`. The capacity of the new cell is exactly 74. This is because the size of `HelloNervos.txt` is 13 bytes. If you remember from earlier, the minimum capacity for a standard cell is 61 bytes. The minimum capacity requirements are always for the total space a cell occupies, which includes both the data in the cell and the overhead of the structures that comprise the cell itself. The cell structures take 61 bytes, the data takes 13 bytes, and 61 + 13 = 74.

Now let's look at the cell that was just created. Execute the following command, replacing the TX Hash with the transaction from the `wallet transfer` command we just executed above.

```bash
ckb-cli rpc get_live_cell --tx-hash <tx_hash> --index 0 --with-data
```

Your output should be very similar to this:

```
cell:
  data:
    content: 0x48656c6c6f204e6572766f7321
    hash: 0xaa44a1b32b437a2a68537398f7730b4d3ef036cd1fdcf0e7b15a04633755ac31
  output:
    capacity: 0x1b9130a00
    lock:
      args: 0xc8328aabcd9b9e8e64fbc566c4385c3bdeb219d7
      code_hash: 0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8
      hash_type: type
    type: ~
status: live
```

The hex-encoded content of `0x48656c6c6f204e6572766f7321` decodes to `Hello Nervos!`, which is the content of the file `HelloNervos.txt`. The data hash value of `0xaa44a1b32b437a2a68537398f7730b4d3ef036cd1fdcf0e7b15a04633755ac31` is the hash of the content itself.

To verify, let's check the hash of the file itself using this command:

```bash
ckb-cli util blake2b --binary-path "./files/HelloNervos.txt"
```

The output should match the data hash value from the previous command.

Looking at the output capacity, we have a value of `0x1b9130a00`. When decoded back to decimal, it is a value of 7,400,000,000. The value is in Shannons, which means this is exactly 74 CKBytes.

### Storing Data Using Lumos

Storing data using Lumos is very similar to what you've already done in previous labs. Here is an example of the JSON structure used as an output in previous code examples:

```javascript
{
    cellOutput:
    {
        capacity: outputCapacity,
        lock: addressToScript(address),
        type: null
    },
    data: "0x"
}
```

The `data` field is a hex string of the data to create the cell with. The process to add data is simple. Replace this with the hex string for the data desired, and adjust the capacity if necessary to accommodate the extra storage required by the data.&#x20;

Looking at the code example in the folder `Storing-Data-in-a-Cell-Example`, we see the code equivalent of the `ckb-cli` command we used earlier.

```javascript
// Create a cell with a capacity large enough for the data being placed in it.
const hexString = "0x48656c6c6f204e6572766f7321"; // "Hello Nervos!" as a hex string.
const dataSize = ((hexString.length - 2) / 2); // Calculate the size of hexString as binary.
const outputCapacity1 = intToHex(ckbytesToShannons(61n) + ckbytesToShannons(dataSize)); // 61 CKBytes for the Cell minimum + the size of the data.
const output1 = {cellOutput: {capacity: outputCapacity1, lock: addressToScript(address), type: null}, data: hexString};
transaction = transaction.update("outputs", (i)=>i.push(output1));
```

On line 2 you see the text "Hello Nervos!" encoded as a hex string. On line 3 we calculate the size of the data if it was decoded back to binary. On line 4 we use that information to calculate the capacity needed for the cell, which is equal to the base requirement of 61 CKBytes plus CKBytes equal to the amount of data being stored. On line 5 we specify the `data` for the cell using the `hexString` provided from out function.&#x20;

In a terminal, open the `Storing-Data-in-a-Cell-Example` directory and then execute the example using `node index.js`. The example should execute successfully and print a transaction hash. Using this hash with the command below:

```
ckb-cli rpc get_live_cell --tx-hash <tx_hash> --index 0 --with-data
```

Your output should match that of this below.

```
cell:
  data:
    content: 0x48656c6c6f204e6572766f7321
    hash: 0xaa44a1b32b437a2a68537398f7730b4d3ef036cd1fdcf0e7b15a04633755ac31
  output:
    capacity: 0x1b9130a00
    lock:
      args: 0x988a9c3e74c09dab76c8e41d481a71f4d36d772f
      code_hash: 0x9bd7e06f3ecf4be0f2fcd2188b23f1b9fcc88e5d4b65a8637b17723bbda3cce8
      hash_type: type
    type: ~
status: live
```

If you compare this with the output from earlier, it should be identical. We have created two different cells, but the data contained within is identical.
