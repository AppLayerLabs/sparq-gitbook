---
description: A walkthrough on the basics of a Dynamic Contract's structure.
---

# Creating a Dynamic Contract (Simple)

Let's create a test contract that allows three variables - `name`, `number` and `tuple` - to be changed by the owner of the contract, as well as perform basic operations with structs/tuples and event emissions. We will call this contract `SimpleContract` (which just so happens to be already implemented by the project due to internal testing purposes, but you can do it yourself by hand - check the `src/contract/templates/simplecontract.{h,cpp,sol}` files for reference).

### Creating the Files

First we need to create the contract's header (`.h`) and source (`.cpp`) files, as is customary in C++ development - the header will contain the definition of our contract class, and the source will contain the implementation details.

Go to your local testnet's root folder, into the `src/contract/templates` subfolder, and create two new files - `simplecontract.h` and `simplecontract.cpp`. Those files will contain the declaration and definition of your contract's logic, respectively.

Then, add both files to the `CMakeLists.txt` file in the same folder, so they can be compiled with the project:

```cmake
set(CONTRACT_HEADERS
  # ...
  ${CMAKE_SOURCE_DIR}/src/contract/templates/simplecontract.h
  # ...
)
set(CONTRACT_SOURCES
  # ...
  ${CMAKE_SOURCE_DIR}/src/contract/templates/simplecontract.cpp
  # ...
)
```

In the following subchapters, we'll implement the header file first, then the source file, write some tests and finally deploy the contract alongside the blockchain.
