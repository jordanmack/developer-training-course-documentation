# Lab: Convert Always Success to Data10

Complete the exercise in `index.js` in the folder `Lab-Data10-Exercise` by adding code and values as necessary.

The `index.js` file contains Lumos code to deploy, create, and consume cells using the **Always Success lock script**. Your task is to modify the functionality of this code to use the **Data10 type script**.

1. Change the binary that is used from `always_success` to `data10`.
   * Note: The data10 binary has already been compiled and provided. Do not change this binary or it may cause the lab to fail.
2. Update the `createCells()` function.
   1.  Create three cells that use the default lock with `address1`, and the `data10` type script.
   2. Each one of these cells should have a capacity of 104 CKBytes.
   3. Each of these three cells should contain one of the following three strings, in this specific order: "HelloWorld", "Foo Bar", and "LoremIpsum".

   * Hint: Make sure that your `createCells()` function has the necessary cell deps. Remember: A lock script executes on inputs only. A type script executes on both inputs and outputs.
3. Update the `consumeCells()` function to consume the three cells that were created.
   * Hint: Modify the `CellCollector()` part of the code to locate the proper cells to add.

Run your code by opening a terminal to the `Lab-Data10-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-Data10-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

