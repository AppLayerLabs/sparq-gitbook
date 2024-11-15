---
description: How contract calls happen from the EVM side in AppLayer.
---

# EVM to other contract calls

For calls from the EVM to another contract, the `ContractHost::call()` function plays a crucial role. It is tasked with creating and handling calls to other contracts, encapsulating the complexity of contract interaction within a simple interface:

```cpp
evmc::Result call(const evmc_message& msg) noexcept final;
```

This function is designed to handle both C++ and EVM contract calls, as shown below:

```cpp
evmc::Result ContractHost::call(const evmc_message& msg) noexcept {
  evmc::Result result;
  const bool isContractCall = isCall(msg);

  if (isContractCall) {
    this->traceCallStarted(msg);
  }

  switch (this->decodeContractCallType(msg))
  {
  case ContractType::CREATE: {
    result = this->callEVMCreate(msg);
    break;
  }
  case ContractType::CREATE2: {
    result = this->callEVMCreate2(msg);
    break;
  }
  case ContractType::PRECOMPILED: {
    result = this->processBDKPrecompile(msg);
    break;
  }
  case ContractType::CPP: {
    result = this->callCPPContract(msg);
    break;
  }
  default:
    result = this->callEVMContract(msg);
    break;
  }

  if (isContractCall) {
    this->traceCallFinished(result.raw());
  }

  return result;
}
```
