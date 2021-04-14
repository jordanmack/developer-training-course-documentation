# Lab: Use the DataRange Script in Lumos

Complete the exercise in `index.js` in the folder `Lab-DataRange-Exercise` by adding code and values as necessary.

The `index.js` file contains Lumos code to deploy, create, and consume cells using the **DataCap type script**. Your task is to modify the functionality of this code to use the **DataRange type script**.

1. Change the binary that is used from `datacap` to `datarange`.
   * Note: The datarange binary has already been compiled and provided. Do not change this binary or it may cause the lab to fail.
2. Update the `createCells()` function.
   1.  Create three cells that use the default lock with `address1`, and the `datarange` type script.
   2. Each one of these cells should contain only the minimum capacity necessary to hold the cell data.
   3. Each of these three cells should contain the following data size restrictions, in this specific order: \(0, 10\), \(3, 12\), \(128, 256\). These size pairs represent the minimum and maximum, respectively.
   4. Each of these three cells should contain any data of your choosing, that matches the given size ranges.
3. Update the `consumeCells()` function to consume the three cells that were created.
   * Hint: Modify the `CellCollector()` part of the code to locate the proper cells to add.

Run your code by opening a terminal to the `Lab-DataRange-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-DataRange-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

