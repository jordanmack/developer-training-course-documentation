# Lab: Updating Data in a Cell

Complete the transaction in `index.js` found in the folder `Lab-Updating-Data-in-a-Cell-Exercise` by adding code and values as necessary.

Your code should locate the two existing Cells which have data matching the contents of `HelloNervos.txt`. Update one with the contents of `HelloWorld.txt`. Update the other with the contents of `LoremIpsum.txt`.

Your resulting transaction should contain:

* Two inputs from `address1` that contain the data from the file `HelloNervos.txt`.
* One or more extra inputs from `address1` if more capacity is needed.
* One output to `address1` that contains the data from the file `HelloWorld.txt`.
* One output to `address1` that contains the data from the file `LoremIpsum.txt`.
* One output to `address1` with the change from the transaction, if necessary.
* A transaction fee.

![](../.gitbook/assets/transaction-structure%20%281%29.png)

In this lab exercise, all of the core Cell management logic has been removed. You must construct it yourself. Feel free to copy and paste some of your code from previous exercises to complete this lab exercise, but it's recommended that you try to write as much of the code as possible. 

1. Provide code to locate the two input cells that contain data matching the contents of `files/HelloNervos.txt`.
2. Update the data in those two cells with the contents of `files/HelloWorld.txt` and `files/LoremIpsum.txt` by creating two output cells.
3. If more input capacity is needed, add more input cells as necessary.
4. If a change cell is needed, add an output as needed.

Run your code by opening a terminal to the `Lab-Updating-Data-in-a-Cell-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Updating-Data-in-a-Cell-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

