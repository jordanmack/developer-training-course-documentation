# Lab: Store a File in a Cell

Complete the transaction in `index.js` found in the folder `Lab-Store-a-File-in-a-Cell-Exercise` by adding code and values as necessary.

The transaction you create should have one or more inputs from `address1`, one output to `address1` with the `data` set to the contents of the file `HelloNervos.txt` one change cell back to `address1`if necessary, and a TX fee.

![](../.gitbook/assets/transaction-structure%20%282%29.png)



1. Populate the `txFee` variable with a 0.001 CKByte fee.
   * Hint: The fee value must be given as a BigInt value expressed in Shannons. There are 100,000,000 Shannons in a CKByte.
2. Populate the `hexString` variable with the contents of the `files/HelloNervos.txt` encoded as a hex string.
   * Hint: Use the [Node.js native functions](https://nodejs.dev/learn/reading-files-with-nodejs) to read the file to a Buffer, then use `.toString("hex")` to convert it to a hex string.
3. Populate the `dataSize` variable with the size of the data.
   * Hint: The size of the data should be in binary format, not hex string format.
4. Populate the `outputCapacity1` variable with the minimum amount of CKBytes necessary to create a cell with the data being included.
   * Hint: Capacity values added to the cell output structure must be in Shannons, and expressed as a hex value. Don't forget to use `intToHex()` and `ckbytesToShannons()`.
5. Populate the `output1` variable with the JSON structure for an output cell that is owned by `address` and has the data from `hexString`.
   * Hint: You can copy the structure from `output2` to use as a reference. 

Run your code by opening a terminal to the `Lab-Store-a-File-in-a-Cell-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Store-a-File-in-a-Cell-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

