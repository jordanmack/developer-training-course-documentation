# Lab: Implement Automated Cell Collection

Complete the transaction in `index.js` found in the folder `Lab-Implement-Automated-Cell-Collection-Exercise` by adding code and values as necessary. 

1. 2. Perform a manual Cell collection and locate a usable Live Cell owned by the account `ckt1qyqvsv5240xeh85wvnau2eky8pwrhh4jr8ts8vyj37` and use it to populate the `previousOutput` variable.
3.  Populate the `txFee` variable with a 0.00123 CKByte fee.
   * Hint: The fee value must be given as a BigInt value expressed in Shannons. There are 100,000,000 Shannons in a CKByte.
4. Populate the `output2` variable with a Cell output structure that properly creates a change Cell for any remaining CKBytes from the Input Cell.
   * Hint: Pay attention to the value of `output1` for the syntax of the value.
5. The transaction you create should have one or more inputs, one output to address2 for 100 CKBytes, and one change Cell if necessary.

![](../.gitbook/assets/transaction-structure.png)

Run your code by opening a terminal to the `Lab-Implement-Automated-Cell-Collection-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Implement-Automated-Cell-Collection-Solution` folder, but don't use it unless you absolutely need it!

Once your code successfully executes, the resulting transaction id will be printed on the screen.

