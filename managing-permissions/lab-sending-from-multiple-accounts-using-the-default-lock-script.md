# Lab: Sending From Multiple Accounts Using the Default Lock Script

Complete the transaction in `index.js` found in the folder `Lab-Sending-From-Multiple-Accounts-Exercise` by adding code and values as necessary.

Your task is to create a single transaction where Alice, Bob, and Charlie each send 100 CKBytes to Dan.

Your resulting transaction should contain:

* A 100 CKByte input cell from Alice.
* A 100 CKByte input cell from Bob.
* A 100 CKByte input cell from Charlie.
* An output cell to Daniel for 299.999 CKBytes.
* A transaction fee of 0.001 CKBytes.

![](../.gitbook/assets/transaction-structure%20%285%29.png)

In this lab exercise, we will describe the syntax, but it is up to you to construct it. You may copy and paste some of your code from previous exercises to complete this lab exercise if needed, but it's recommended that you try to write as much of the code as possible without looking at previous examples.

1. 
Run your code by opening a terminal to the `Lab-Sending-From-Multiple-Accounts-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Sending-From-Multiple-Accounts-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

