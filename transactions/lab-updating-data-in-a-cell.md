# Lab: Updating Data in a Cell

Complete the transaction in `index.js` found in the folder `Lab-Updating-Data-in-a-Cell-Exercise` by adding code and values as necessary.

Locate the two existing Cells which have data matching the contents of `HelloNervos.txt`. Update one with the contents of `HelloWorld.txt`. Update the other with the contents of `LoremIpsum.txt`.

Your resulting transaction should contain:

* Two inputs from `address1` that contain the data from the file `HelloNervos.txt`.
* One or more extra inputs from `address1` if more capacity is needed.
* One output to `address1` that contains the data from the file `HelloWorld.txt`.
* One output to `address1` that contains the data from the file `LoremIpsum.txt`.
* One output to `address1` with the change from the transaction, if necessary.
* A transaction fee.

![](../.gitbook/assets/transaction-structure%20%281%29.png)

1. 2. Populate the `hexString` variable with the contents of the `files/HelloNervos.txt` encoded as a hex string.
   * Hint: Use the [Node.js native functions](https://nodejs.dev/learn/reading-files-with-nodejs) to read the file to a Buffer, then use `.toString("hex")` to convert it to a hex string.
3. Populate the `dataSize` variable with the size of the data.
   * Hint: The size of the data should be in binary format, not hex string format.
4. Populate the `outputCapacity1` variable with the minimum amount of CKBytes necessary to create a Cell with the data being included.
   * Hint: Capacity values added to the Cell output structure must be in Shannons, and expressed as a hex value. Don't forget to use `intToHex()` and `ckbytesToShannons()`.
5. Populate the `output1` variable with the JSON structure for an output Cell that is owned by `address` and has the data from `hexString`.
   * Hint: You can copy the structure from `output2` to use as a reference. 

Run your code by opening a terminal to the `Lab-Store-a-File-in-a-Cell-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Store-a-File-in-a-Cell-Solution` folder.

Once your code successfully executes, the resulting transaction id will be printed on the screen.

