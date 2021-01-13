# Updating Data in a Cell

Updating the data in a Cell is another common process that must be performed, but it's not quite as intuitive as one might think. Cells are immutable structures. Once they are added to the blockchain they cannot be modified in any way. The process to update the data of a Cell is to consume an existing Cell and then create a new one in its place.

![](../.gitbook/assets/updating-cell-data-flow.png)

In the above example image, Charlie is updating the data in a Cell he owns. The input Cell is consumed and effectively destroyed. In its place, a new Cell is created with different data. Note: The transaction fee has been omitted from this example for simplicity.

In a sense, it isn't really an "update" at all because the Cell being consumed has no direct connection to the Cell being created. The process should be relatively simple at this point, but it is important to completely understand what is going on. Once we start working with smart contracts in later lessons it will become apparent why this conceptual difference is very important.

### Updating Data Using Lumos

Updating a Cell in Lumos consists of first locating the existing Cell to be updated, and then constructing the new Cell to replace it.

To locate the existing Cell, we will use the `CellCollector()` as we did previously, but we will need to update the query to only return Cells that have the specific data we're looking for.

Here is the query that is used with the `CellCollector()` in the shared library function `collectCapacity()`.

```javascript
{lock: lockScript, type: null}
```



