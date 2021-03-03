# Introduction to Capsule

Capsule is a development framework for used to create on-chain scripts using the Rust and C programming languages. This includes both lock scripts and type scripts. Capsule provides the necessary tools to bootstrap, compile, test, debug, and deploy a new project. We will give a quickstart here to cover the basics of how a project is structured, and how testing is performed.

![](../.gitbook/assets/capsule.jpg)

### Installation

Capsule can be installed on Linux, MacOS, and Windows via [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Docker and Rust will need to be installed before Capsule will function properly.

You should already have Rust installed since it was a prerequisite to this course. Installation instructions for Docker can be found on [their website](https://docs.docker.com/get-docker/). Docker must also be configured to [allow management by a non-root user](https://docs.docker.com/engine/install/linux-postinstall/).

After Rust and Docker are installed, you can type the following to install capsule:

```text
cargo install ckb-capsule
```

If you experience any problems, the full instructions can always be viewed on the [Capsule GitHub page](https://github.com/nervosnetwork/capsule).

### Creating Your First Capsule Project

From a terminal, type the following command to create a new project called `myproject` in the current directory:

```bash
capsule new myproject
```

This should display the output similar to the following.

```bash
New project "myproject"
Created file "capsule.toml"
Created file "deployment.toml"
Created file "README.md"
Created file "Cargo.toml"
Created file ".gitignore"
Initialized empty Git repository in /home/username/myproject/.git/
Created "/home/username/myproject"
Created tests
     Created library `tests` package
New contract "myproject"
     Created binary (application) `myproject` package
Rewrite Cargo.toml
Rewrite ckb_capsule.toml
Done
```

This initializes a capsule project called `myproject` and creates a single contract in the project called `myproject` within it. You can create as many contracts as you would like within a project using the `capsule new-contract` command, but we will start with the included default script.

Enter the project directory using `cd myproject`, then use the following command to build the project.

```text
capsule build
```

You should see output similar to this.

```bash
Building contract myproject
 Downloading crates ...
  Downloaded ckb-std v0.7.3
   Compiling cfg-if v0.1.10
   Compiling cc v1.0.41
   Compiling buddy-alloc v0.4.1
   Compiling molecule v0.6.0
   Compiling ckb-standalone-types v0.0.1-pre.1
   Compiling ckb-std v0.7.3
   Compiling myproject v0.1.0 (/code/contracts/myproject)
    Finished dev [unoptimized + debuginfo] target(s) in 6.13s
Done
```

This builds the current project binaries in debug mode. Now that our binary is built, we can test it using the following command.

```text
capsule test
```

This will execute all tests for the project. The first time this runs, it may take a while to compile because it's building all the necessary testing tools including a light-weight simulator that will be used run the compiled scripts in a simulated blockchain environment. The output should be similar to this.

```bash
    Finished test [unoptimized + debuginfo] target(s) in 0.38s
     Running target/debug/deps/tests-b429868b4af9df7d

running 2 tests
[contract debug] script args is Bytes([42])
[contract debug] script args is Bytes([])
test tests::test_empty_args ... ok
[contract debug] tx hash is [112, 109, 48, 116, 133, 71, 95, 186, 208, 254, 244, 53, 91, 80, 108, 213, 206, 112, 25, 187, 136, 180, 123, 124, 240, 31, 48, 192, 193, 175, 179, 176]
consume cycles: 322770
test tests::test_success ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests tests

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

The tests ran successfully! 🎉

### Project Structure

Next, we will look at how a project is structured. Open the `myproject/contracts/myproject/src` folder, which contains the source files for the `myproject` contract. You will see the following files.

* **entry.rs** - This file contains the logic of the smart contract.
* **error.rs** - This file contains error codes used in the project.
* **main.rs** - This file contains boilerplate code for the contract.

Every Rust program begins with `main.rs`. Feel free to open it and take a look at what it contains, but we're not going to go through it here. This file is all boilerplate code that is needed to build a script. We won't need to touch anything in this file. 

Next, let's look at the contents of `entry.rs`.

```rust
// Import from `core` instead of from `std` since we are in no-std mode
use core::result::Result;

// Import heap related library from `alloc`
// https://doc.rust-lang.org/alloc/index.html
use alloc::{vec, vec::Vec};

// Import CKB syscalls and structures
// https://nervosnetwork.github.io/ckb-std/riscv64imac-unknown-none-elf/doc/ckb_std/index.html
use ckb_std::{
    debug,
    high_level::{load_script, load_tx_hash},
    ckb_types::{bytes::Bytes, prelude::*},
};

use crate::error::Error;

pub fn main() -> Result<(), Error> {
    // remove below examples and write your code here

    let script = load_script()?;
    let args: Bytes = script.args().unpack();
    debug!("script args is {:?}", args);

    // return an error if args is invalid
    if args.is_empty() {
        return Err(Error::MyError);
    }

    let tx_hash = load_tx_hash()?;
    debug!("tx hash is {:?}", tx_hash);

    let _buf: Vec<_> = vec![0u8; 32];

    Ok(())
}
```

This file contains the main logic of the script. This example script contains boilerplate code and example code. Functionally, all it does is check if `args` were supplied to the script, and it gives an error if there were none.

Lines 1-14 are the various includes. Rust scripts must be programmed in `no_std`, which is why lines 2 and 6 import `Result` and `Vec` from somewhere else other than the Rust standard library. Line 16 imports the local error crate, which we will cover momentarily.

Lines 18 to 36 are all boilerplate example code. Lines 21 to 23 print the `args` for the executing script to the terminal. This is visible in the test output from earlier.

Lines 26 to 28 show how a custom error would be used. This would immediately return an error, which would, in turn, cause the transaction to fail with the indicated error code.

Lines 30 and 31 print the transaction hash to the console.

Line 33 is a demo code that allocates an unused vector as a buffer.

Line 35 exits the script with success.

Next, let's look at `error.rs`.

```rust
use ckb_std::error::SysError;

/// Error
#[repr(i8)]
pub enum Error {
    IndexOutOfBound = 1,
    ItemMissing,
    LengthNotEnough,
    Encoding,
    // Add customized errors here...
    MyError,
}

impl From<SysError> for Error {
    fn from(err: SysError) -> Self {
        use SysError::*;
        match err {
            IndexOutOfBound => Self::IndexOutOfBound,
            ItemMissing => Self::ItemMissing,
            LengthNotEnough(_) => Self::LengthNotEnough,
            Encoding => Self::Encoding,
            Unknown(err_code) => panic!("unexpected sys error {}", err_code),
        }
    }
}
```

This file contains all the possible error codes for our script.

Lines 6 to 9 are the standard errors that could be returned from the CKB node. Lines 14 to 25 map the system error codes to those used in our script on lines 6 to 9.

Line 11 is our custom error. We can add as many as are needed here. Any time a script fails an error code will be returned. This is very useful for debugging and testing.

### Testing

Capsule comes with a robust testing framework that allows you to run automated tests against a CKB node simulator. This tool is so powerful that many developers use it as the primary way to develop applications, using test-driven development \(TDD\).

Let's look at how tests are structured. Open the `myproject/tests/src` folder. You will see the following files.

* **lib.rs** - This file contains boilerplate code used for the test suite.
* **tests.rs** - This file contains all test cases for the project.

Feel free to open `lib.rs` and take a look at what it contains, but we're not going to go through it here. This file is all boilerplate code that is needed for testing. We won't need to touch anything in this file. 

Let's take a look at `tests.rs`. Starting at the top you see this.

```rust
use super::*;
use ckb_testtool::context::Context;
use ckb_tool::ckb_types::{
    bytes::Bytes,
    core::TransactionBuilder,
    packed::*,
    prelude::*,
};
use ckb_tool::ckb_error::assert_error_eq;
use ckb_tool::ckb_script::ScriptError;
```

These are the basic imports for the test suite.

```rust
const MAX_CYCLES: u64 = 10_000_000;
```

This is the maximum amount of cycles \(CPU resources\) that a script can use. If you're coming from Ethereum, you can think of this as a gas limit. This is a fairly high number, so it doesn't generally need to be changed.

```rust
// error numbers
const ERROR_EMPTY_ARGS: i8 = 5;
```

Here are the error numbers that are being tested for. These should match the values from the `error.rs` file. Remember, the `test.rs` file is for the entire project. It might contain multiple contracts \(scripts\). These errors should be named appropriately so they don't get confused.

```rust
#[test]
fn test_success() {
    // deploy contract
    let mut context = Context::default();
    let contract_bin: Bytes = Loader::default().load_binary("myproject");
    let out_point = context.deploy_cell(contract_bin);

    // prepare scripts
    let lock_script = context
        .build_script(&out_point, Bytes::from(vec![42]))
        .expect("script");
    let lock_script_dep = CellDep::new_builder()
        .out_point(out_point)
        .build();

    // prepare cells
    let input_out_point = context.create_cell(
        CellOutput::new_builder()
            .capacity(1000u64.pack())
            .lock(lock_script.clone())
            .build(),
        Bytes::new(),
    );
    let input = CellInput::new_builder()
        .previous_output(input_out_point)
        .build();
    let outputs = vec![
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script.clone())
            .build(),
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script)
            .build(),
    ];

    let outputs_data = vec![Bytes::new(); 2];

    // build transaction
    let tx = TransactionBuilder::default()
        .input(input)
        .outputs(outputs)
        .outputs_data(outputs_data.pack())
        .cell_dep(lock_script_dep)
        .build();
    let tx = context.complete_tx(tx);

    // run
    let cycles = context
        .verify_tx(&tx, MAX_CYCLES)
        .expect("pass verification");
    println!("consume cycles: {}", cycles);
}
```

This is the first test that will execute. There is a lot of code here. This is deploying the contract, building cells with scripts, forming a transaction, then executing the transaction in a virtual environment that simulates a blockchain. As the `test_success()` function name indicates, this is testing for an expected success scenario.

Line 5 loads the `myproject` contract binary that was built in the `myproject/build/debug` directory. You must always compile your script using `capsule build` before it is available for testing. Line 6 deploys the code and gets the out point.

Line 9 to 11 build a lock script from the deployed `myproject` binary. On line 10, the `args` are being specified as the second parameter in the `build_script()` method. It's being set to a single byte with the value `42`. Lines 12 to 14 deploy the lock script.

Lines 17 to 26 create an input cell. The script has a capacity of 1,000 Shannons and uses the lock script we just deployed. This is far below the requirements of a real cell, but in this simulator, the rules are relaxed.

Lines 27 to 36 create two output cells. Both cells have 500 Shannons of capacity and use the same lock script. Line 38 defines the data that is in our cells. Both cells have no data.

Lines 41 to 47 populate the transaction and build it. Our lock script doesn't require a signature, so none are included here. If we did need one, we would populate it using `.witnesses()`.

Lines 50 to 53 execute the transaction, expecting that it will pass. The cycles are printed to the screen showing how many were consumed during execution.

```rust
#[test]
fn test_empty_args() {
    // deploy contract
    let mut context = Context::default();
    let contract_bin: Bytes = Loader::default().load_binary("myproject");
    let out_point = context.deploy_cell(contract_bin);

    // prepare scripts
    let lock_script = context
        .build_script(&out_point, Default::default())
        .expect("script");
    let lock_script_dep = CellDep::new_builder()
        .out_point(out_point)
        .build();

    // prepare cells
    let input_out_point = context.create_cell(
        CellOutput::new_builder()
            .capacity(1000u64.pack())
            .lock(lock_script.clone())
            .build(),
        Bytes::new(),
    );
    let input = CellInput::new_builder()
        .previous_output(input_out_point)
        .build();
    let outputs = vec![
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script.clone())
            .build(),
        CellOutput::new_builder()
            .capacity(500u64.pack())
            .lock(lock_script)
            .build(),
    ];

    let outputs_data = vec![Bytes::new(); 2];

    // build transaction
    let tx = TransactionBuilder::default()
        .input(input)
        .outputs(outputs)
        .outputs_data(outputs_data.pack())
        .cell_dep(lock_script_dep)
        .build();
    let tx = context.complete_tx(tx);

    // run
    let err = context
        .verify_tx(&tx, MAX_CYCLES)
        .unwrap_err();
    // we expect an error raised from 0-indexed cell's lock script
    let script_cell_index = 0;
    assert_error_eq!(
        err,
        ScriptError::ValidationFailure(ERROR_EMPTY_ARGS).input_lock_script(script_cell_index)
    );
}
```

This is the second test that will execute. As the `test_empty_args()` function name indicates, this is testing for an expected failure scenario due to there not being any args supplied to the script. A lot of this code is very similar to the previous code block, so we will only look at the most important lines.

Line 10 creates our lock script. The second parameter is the `args` to the script, and it is empty this time. 

Lines 50 to 58 execute the transaction. This is expected to fail, and line 57 specifies `ERROR_EMPTY_ARGS` as the expected error code. As long as the script returns this specific error code, this test will validate successfully.

