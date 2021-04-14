# Lab: Convert Always Success to JSONCell

Complete the exercise in `index.js` in the folder `Lab-JSONCell-Exercise` by adding code and values as necessary.

The `index.js` file contains Lumos code to deploy, create, and consume cells using the **Always Success lock script**. Your task is to modify the functionality of this code to use the **JSONCell type script**.

1. Change the binary that is used from `always_success` to `jsoncell`.
   * Note: The jsoncell binary has already been compiled and provided. Do not change this binary or it may cause the lab to fail.
2. Update the `createCells()` function.
   1.  Create three cells that use the default lock with `address1`, and the `jsoncell` type script.
   2. Each one of these cells should contain only the minimum capacity necessary to hold the cell data.
   3. Each of these three cells should contain one of the following JSON objects, in this specific order: `"Hello World!"`, `["Foo", "Bar"]`, and `{"Lorem": "Ipsum"}`.

   * Hint: Make sure that your `createCells()` function has the necessary cell deps. Remember: A lock script executes on inputs only. A type script executes on both inputs and outputs.
3. Update the `consumeCells()` function to consume the three cells that were created.
   * Hint: Modify the `CellCollector()` part of the code to locate the proper cells to add.

Run your code by opening a terminal to the `Lab-JSONCell-Exercise` folder and running `node index.js`. If you get stuck you can find the solution in the `Lab-JSONCell-Solution` folder.

Once your code successfully executes, the resulting transaction ID will be printed on the screen.

