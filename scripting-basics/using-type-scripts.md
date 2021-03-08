# Using Type Scripts

Nervos has two types of scripts, lock scripts and type scripts. We've worked with many lock scripts in the previous examples, and now we will start working with type scripts.

There are a number of similarities between lock scripts and type scripts. Both scripts have the same `code_hash`, `hash_type`, and `args` fields. Both are executing RISC-V binaries in CKB-VM. Both have access to the same details of a transaction during execution. Both play a part in answering the overshadowing question: "is this transaction valid?"

However, there are strong semantical differences between lock scripts and type scripts. Lock scripts are concerned with ownership, and type scripts are concerned with state transitions. Lock scripts define how a cell is accessed, type scripts define how a cell behaves. When used as a form of identification, lock scripts define the owner or a cell, and type scripts define what the cell is.

### Script Execution

Many of the details surrounding lock scripts and type scripts are the same. Based on the similarities mentioned above, it might seem like the two are interchangeable. In some limited cases, this is true. Both have the same three fields, `code_hash`, `hash_type`, and `args`. If you were to swap the information of these three fields between the lock script and type script on a cell, the code binaries would still execute. However, there is a high likelihood that the scripts would not function as intended and the cell could become permanently locked.

There is one major distinction between lock scripts and type scripts in regards to execution. Lock scripts execute on input cells. Type scripts execute on both input cells and output cells. It may not seem like that big of a difference, but this is the single factor that leads to all the semantical differences mentioned above. If lock scripts and type scripts executed at the exact same times, then the two would be completely interchangeable.

 





