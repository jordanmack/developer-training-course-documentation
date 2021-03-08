# Using Type Scripts

Nervos has two types of scripts, lock scripts and type scripts. We've worked with many lock scripts in the previous examples, and now we will start working with type scripts.

There are a number of similarities between lock scripts and type scripts. Both scripts have the same `code_hash`, `hash_type`, and `args` fields. Both are executing RISC-V binaries in CKB-VM. Both have access to the same details of a transaction during execution. Both play a part in answering the overshadowing question: "is this transaction valid?"

However, there are strong semantical differences between lock scripts and type scripts. Lock scripts are concerned with ownership, and type scripts are concerned with state transitions. Lock scripts define how a cell is accessed, type scripts define how a cell behaves. When used as a form of identification, lock scripts define the owner or a cell, and type scripts define what the cell is.

### Script Execution





