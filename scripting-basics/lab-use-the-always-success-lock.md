# Lab: Use the Always Success Lock

Complete the exercise in `index.js` found in the folder `Lab-Using-the-Always-Success-Lock-Exercise` by adding code and values as necessary.

Your code should create a transaction with three cells using the always success lock script, then create a second transaction that consumes those cells.

Your resulting create transaction should contain:

* One or more input cells from `address1` that are used for capacity.
* Three output cells that use the always success lock and contain 41 CKBytes.
* One output to `address1` with the change from the transaction.
* A transaction fee of 0.001 CKBytes.

![](../.gitbook/assets/create-transaction-structure%20%288%29.png)

Your resulting consume transaction should contain:

* Three input cells that use the always success lock and contain 41 CKBytes.
* One output to `address1` with the change from the transaction.
* A transaction fee of 0.001 CKBytes.

![](../.gitbook/assets/consume-transaction-structure%20%288%29.png)

In this lab exercise, all of the core cell management logic has been removed. You must construct it yourself. Feel free to copy and paste some of your code from previous exercises to complete this lab exercise, but it's recommended that you try to write as much of the code as possible. 

Run your code by opening a terminal to the `Lab-Using-the-Always-Success-Lock-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Using-the-Always-Success-Lock-Solution` folder.

Once your code successfully completes all transactions a success message will be printed for the exercise.

