# Lab: Calculating Capacity Requirements

Complete the transaction in `index.js` found in the folder `Lab-Calculating-Capacity-Requirements-Exercise` by adding code and values as necessary.&#x20;

1. Perform a manual cell collection and locate a usable live cell owned by the account `ckt1...gwga` and use it to populate the `PREVIOUS_OUTPUT` variable.
   * Hint: The last successful transaction we worked on earlier in this lesson will give you a usable out point matching this account. You should already have the TX hash.
2. &#x20;Populate the `TX_FEE` variable with a 0.0001 CKByte fee.
   * Hint: The fee value must be given as a BigInt value expressed in Shannons. There are 100,000,000 Shannons in a CKByte.
3. Populate the `output2` variable with a cell output structure that properly creates a change cell for any remaining CKBytes from the input cell.
   * Hint: You can copy the value of `output1` to get the required structure, then just change what is necessary.
4. The transaction you create should have one input, two outputs, and a TX fee.

![](../.gitbook/assets/lab-exercise-transaction.png)

Run your code by opening a terminal to the `Lab-Calculating-Capacity-Requirements-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Calculating-Capacity-Requirements-Solution` folder, but don't use it unless you absolutely need it!

Once your code successfully executes, the resulting transaction ID will be printed on the screen without any errors.
