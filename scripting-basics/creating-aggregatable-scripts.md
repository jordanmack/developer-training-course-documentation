# Creating Aggregatable Scripts

One of the unique features of the Cell Model is that multiple cells can be combined into a single large transaction instead of creating separate small transactions. This process is called "aggregation". However, aggregation is only possible when scripts are programmed in a way flexible enough to allow it. In this lesson, we will demonstrate some of the basic principles of creating aggregatable scripts.  

### Minimal Concern Pattern

One of the most common ways to create an aggregatable script is the follow the minimal concern pattern.  Following this creates scripts that have composable cell logic that allows them to be combined in a single transaction safely without affecting other cells.

To incorporate the minimal concern pattern a script should:

1. Process all cells in the script group.
2. Allow any number of cells in a script group.
3. Ignore all cells that are not in the same script group.

In essence, a script should only check the minimal amount of information needed to ensure the validity of the cells it is concerned with.

Let's take a look at how this applies to the update transaction for the Counter script.

![](https://gblobscdn.gitbook.com/assets%2F-MLuiCvogNfxQTk5TWAq%2F-MXU0T9yPe_sU8F7MFQ_%2F-MXUM8V9U-gbd67QG2im%2Fconsume-transaction-structure.png?alt=media&token=16397219-5e71-4284-a52f-94bf382511db)

In this transaction, the Counter is being incremented by 1. This would be successful, but what if we increase the number of Counter cells in the transaction?



