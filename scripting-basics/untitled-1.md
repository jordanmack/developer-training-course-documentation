# Untitled

For example, if we had a token type script that was trying to enforce the rule `input_tokens >= output_tokens`, it would need to locate all of the token cells in the transaction. There could be other cells in the transaction, but our token type script isn't concerned with those. A "token cell" is defined by a cell using the token type script, so we only need to locate those cells. The `GroupInput` and `GroupOutput` sources will only return those matching cells. Below is an image to help illustrate.

![](../.gitbook/assets/group-source-transaction.png)

In this image, Alice is sending two types of tokens, TokenA and TokenB. This image does not include CKBytes or TX fees to help make things easier to understand.

The cells representing TokenA have a red outline. The cells representing TokenB have a green outline. In this single transaction, Alice is sending 200 of TokenA to Bob, 100 of tokenA to Charlie, and 100 of TokenB to Daniel.

TokenA and TokenB will have different type scripts, and they will both execute in this transaction. Both scripts will enforce the rule `input_tokens >= output_tokens`. When the TokenA type script executes, it is only concerned with TokenA cells. When the TokenB type script executes, it is only concerned with the TokenB cells.

Both the TokenA and TokenB type scripts could use the `Input` and `Output` sources to do this. When they execute, they will see all of the cells in the transaction. You can see this in the image as `Input 0` and `Input 1` on the left, and `Output 0`, `Output 1`, and `Output 2` on the right. Both the TokenA and TokenB type scripts will see these same cell indexes when they execute. The next step would be for each script to examine each cell, look at the type script that is used, and filter the results to only the cells of concern.

There is a better way. When the scripts use the `GroupInput` and `GroupOutput` sources, this filtration process is done for them.

![](../.gitbook/assets/group-source-breakdown.png)

When TokenA uses `GroupInput` and `GroupOutput`, it will only see the cells of TokenA type. This is the single input cell with a red border and the two output cells with a red border. This is shown in the two middle columns of the above image.

When TokenB uses `GroupInput` and `GroupOutput`, it will only see the cells of TokenB type. These are the single input cell with a green border, and the single output cell with a green border. This is shown in the two right columns of the above image.

The use of `GroupInput` and `GroupOutput` works for type scripts as described, but it is slightly different for lock scripts. When a lock script uses `GroupInput`, the input cells with the same lock script will be returned. **When a lock script uses the `GroupOutput`, no cells will be returned.** The reason for this is that these groups are related to how scripts are being executed in CKB-VM, and lock scripts do not execute on outputs. We will describe exactly why this is later on. 

