---
description: A primer on natively-coded smart contracts in the AppLayer ecosystem.
---

# Precompiled contracts

Precompiled contracts (also known as "native contracts" or "stateful pre-compiles" - "stateful" since they can maintain a state, and "pre-compile" because they're compiled machine code not interpreted by a virtual machine) are contracts coded with the blockchain's native language, with things like transaction parsing methods and arguments, as well as management and storage of the contract's variables in a database, being manually coded in the blockchain's native language (e.g. bdk-cpp in C++) to be tightly integrated with the blockchain itself.

Similar to Solidity contracts, they can be used to employ any type of logic within the network, but unlike Solidity, they aren’t subject to EVM constraints. This means we can take advantage of that fact and have full control of the contract's logic, unleashing blazing fast performance, flexibility and power.

The contract templates provided by AppLayer's BDK (in the `src/contract/templates` folder) are based on OpenZeppelin contracts, maintaining the same operational standards known in the Solidity ecosystem, but on native C++ (in the case of bdk-cpp).
