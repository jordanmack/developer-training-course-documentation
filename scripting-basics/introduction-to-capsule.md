# Introduction to Capsule

Capsule is a development framework used to create on-chain scripts using the Rust and C programming languages. This includes both lock scripts and type scripts. Capsule provides the necessary tools to bootstrap, compile, test, debug, and deploy a new project. We will give a quickstart here on Rust development to cover the basics of how Capsule is installed, and how a project is structured.

![](../.gitbook/assets/capsule.jpg)

### Installation

Capsule can be installed on Linux, MacOS, and Windows via [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Docker and Rust will need to be installed before Capsule will function properly.

You should already have Rust installed since it was a prerequisite to this course. Installation instructions for Docker can be found on [their website](https://docs.docker.com/get-docker/). Docker must also be configured to [allow management by a non-root user](https://docs.docker.com/engine/install/linux-postinstall/).

After Rust and Docker are installed, you can type the following to install capsule:

```
cargo install ckb-capsule
```

If you experience any problems, the full instructions can always be viewed on the [Capsule GitHub page](https://github.com/nervosnetwork/capsule).

### Creating Your First Capsule Project

From a terminal, type the following command to create a new project called `myproject` in the current directory:

```bash
capsule new myproject
```

This should display the output similar to the following.

```shell
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

```
capsule build
```

You should see output similar to this.

```shell
Building contract myproject
    Updating crates.io index
 Downloading crates ...
  Downloaded cstr_core v0.2.6
  Downloaded ckb-std v0.10.0
   Compiling memchr v2.5.0
   Compiling cfg-if v1.0.0
   Compiling cc v1.0.73
   Compiling cty v0.2.2
   Compiling buddy-alloc v0.4.1
   Compiling molecule v0.7.3
   Compiling ckb-standalone-types v0.1.2
   Compiling cstr_core v0.2.6
   Compiling ckb-std v0.10.0
   Compiling myproject v0.1.0 (/code/contracts/myproject)
    Finished dev [unoptimized + debuginfo] target(s) in 57.43s
Done
```

This builds the current project binaries in debug mode. Now that our binary is built, we can test it using the following command.

```
capsule test
```

This will execute all tests for the project. The first time this runs, it may take a while to compile because it's building all the necessary testing tools including a light-weight simulator that will be used run the compiled scripts in a simulated blockchain environment. The output should be similar to this.

```shell
Finished test [unoptimized + debuginfo] target(s) in 1m 12s
     Running unittests src/lib.rs (target/debug/deps/tests-eb0bb2889f396ebc)

running 2 tests
[contract debug] script args is Bytes([])
test tests::test_empty_args ... ok
[contract debug] script args is Bytes([42])
[contract debug] tx hash is [190, 153, 144, 72, 54, 166, 192, 113, 110, 225, 123, 15, 89, 128, 80, 36, 182, 213, 40, 188, 238, 250, 137, 35, 188, 180, 199, 13, 87, 148, 207, 39]
consume cycles: 215704
test tests::test_success ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.32s

   Doc-tests tests

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The tests ran successfully! 🎉

### Project Structure

Next, we will look at how a project is structured. Open the `myproject/contracts/myproject/src` folder, which contains the source files for the `myproject` contract. You will see the following files.

* **entry.rs** - This file contains the logic of the smart contract.
* **error.rs** - This file contains error codes used in the project.
* **main.rs** - This file contains boilerplate code for the contract.

Every Rust program begins with `main.rs`. Feel free to open it and take a look at what it contains, but we're not going to go through it here. This file is all boilerplate code that is needed to build a script. We won't need to touch anything in this file.&#x20;

Next, let's look at the contents of `entry.rs`.

{% code lineNumbers="true" %}
```rust
// Import from `core` instead of from `std` since we are in no-std mode
use core::result::Result;

// Import heap related library from `alloc`
// https://doc.rust-lang.org/alloc/index.html
use alloc::{vec, vec::Vec};

// Import CKB syscalls and structures
// https://docs.rs/ckb-std/
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
{% endcode %}

This file contains the main logic of the script. This example script contains boilerplate code and example code. Functionally, all it does is check if `args` were supplied to the script, and it gives an error if there were none.

Lines 1-14 are the various includes. Rust scripts must be programmed in `no_std`, which is why lines 2 and 6 import `Result` and `Vec` from somewhere else other than the Rust standard library. Line 16 imports the local error crate, which we will cover momentarily.

Lines 18 to 36 are all boilerplate example code. Lines 21 to 23 print the `args` for the executing script to the terminal. This is visible in the test output from earlier.

Lines 26 to 28 show how a custom error would be used. This would immediately return an error, which would, in turn, cause the transaction to fail with the indicated error code.

Lines 30 and 31 print the transaction hash to the console.

Line 33 is a demo code that allocates an unused vector as a buffer.

Line 35 exits the script with success.

Next, let's look at `error.rs`.

{% code lineNumbers="true" %}
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
{% endcode %}

This file contains all the possible error codes for our script.

Lines 6 to 9 are the standard errors that could be returned from the CKB node. Lines 14 to 25 map the system error codes to those used in our script on lines 6 to 9.

Line 11 is our custom error. We can add as many as are needed here. Any time a script fails an error code will be returned. This is very useful for debugging and testing.

Now that you understand the basic structure of a project, you will be able to better understand the example scripts. We will cover more of Capsule's features in another lesson.

### Clone the Developer Training Course Script Examples

Our lessons going forward will make use of a number of example lock scripts and type scripts. These are contained in a separate GitHub repo. Use the command below to clone the Develop Training Course Script Examples into a directory of your choosing.

```bash
git clone https://github.com/jordanmack/developer-training-course-script-examples.git
```

Next, enter the directory and build all the script binaries.

```
cd developer-training-course-script-examples
capsule build
```

After the build is completed, run all the tests to verify that the scripts are built properly.

```
capsule test
```

These scripts are intended to be used with the lessons, but feel free to experiment with any of these examples.

### Clone the Developer Training Course Script Labs

Our lessons will also have labs that utilize Capsule. These labs can be found in a separate GitHub repo. Use the command below to clone the Develop Training Course Script Labs into a directory of your choosing.

```bash
git clone https://github.com/jordanmack/developer-training-course-script-labs.git
```

There is no need to build at this time. This step will be done during the lab exercises.&#x20;
