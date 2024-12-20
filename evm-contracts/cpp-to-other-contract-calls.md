---
description: How contract calls happen from the C++ side in AppLayer.
---

# C++ to other contract calls

The `ContractHost` class employs templated functions to support flexible and efficient interaction with contracts. These templates enable passing any combination of arguments and return types (including `void`) to and from other types of contracts. This use of templates helps to leverage the fast ABI encoding/decoding processes, ensuring optimal performance and flexibility during contract execution:

```cpp
template <typename R, typename C, typename... Args>
R callContractViewFunction(
  const BaseContract* caller, const Address& targetAddr,
  R(C::*func)(const Args&...) const, const Args&... args
) const;

template <typename R, typename C, typename... Args>
R callContractFunction(
  BaseContract* caller, const Address& targetAddr,
  const uint256_t& value,
  R(C::*func)(const Args&...), const Args&... args
);
```

This approach allows for dynamic interaction with contracts without pre-defining all possible function signatures, accommodating various contract behaviors and states dynamically.
