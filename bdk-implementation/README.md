---
description: How the functional elements of AppLayer interact with each other.
---

# Blockchain Development Kit (BDK) implementation

This chapter aims to explain in technical detail how the BDK is implemented, as well as its submodules and how everything comes together to deliver a blazing fast blockchain.

The first few subchapters paint a more holistic view of the BDK, as most components are pretty straight-forward to understand. Developers are expected to use the [Doxygen](https://doxygen.nl) documentation as a reference to further understand how the project works. The later subchapters show some components that are particularly denser and/or complex enough that they warrant their own separated explanations.

Looking at a higher level of abstraction, the original C++ implementation of the BDK is structured like this:

* The `src/bins` folder contains the source files for the project's main executables - the blockchain executable itself, contract ABI generator, network simulator, faucet API and other testing-related executables are all coded here in their respective subfolders
* The `src/bytes` folder contains code related to the `bytes` class, a container that deals with raw bytes - specifically the `bytes::join()` function and the `bytes::View` class, both used extensively across the project
* The `src/contract` folder contains everything related to the logic of smart contracts - from ABI parsing to custom variable types and template contracts
* The `src/core` folder contains the heart of the BDK - the main components of the blockchain and what makes it tick
* The `src/libs` folder contains third-party libraries not inherently tied to the project but used throughout development
* The `src/net` folder contains everything related to networking, communication between nodes and support for protocols such as gRPC, HTTP, P2P, JSON-RPC, etc.
* The `src/utils` folder contains several commonly-used functions, structures, classes and overall logic to support the functioning of the BDK as a whole

There is also a `tests` folder that contains several unit tests for each of the components described above.

## Source tree

For the more visually inclined, here is a source tree (headers only) containing all of the files inside the `src` folder (except `src/bins` as it only contains source files), their respective subfolders and which components are declared in them. Each component is further explained through the following subchapters of this documentation. For more technical details (e.g. API references for developers), please refer to the [Doxygen](https://www.doxygen.nl) documentation on the project's own repository.

```
src
├── bytes
│   ├── initializer.h (bytes::Initializer, bytes::SizedInitializer)
│   ├── join.h (bytes::join())
│   ├── range.h (bytes::Range, bytes::DataRange, bytes::BorrowedDataRange)
│   └── view.h (bytes::View, bytes::Span)
├── contract (Contracts)
│   ├── abi.h (ABI - encoders, decoders, helper structs, etc.)
│   ├── calltracer.h (trace namespace, Call struct, CallTracer class)
│   ├── contractfactory.h (ContractFactory)
│   ├── contract.h (ContractGlobals, ContractLocals, BaseContract)
│   ├── contracthost.h (ContractHost)
│   ├── contractmanager.h (ContractManager)
│   ├── contractstack.h (ContractStack)
│   ├── customcontracts.h (for declaring custom contracts)
│   ├── dynamiccontract.h (DynamicContract)
│   ├── event.h (Event)
│   ├── templates (folder for contract templates)
│   │   ├── dexv2 (subfolder for the DEXV2 contract components)
│   │   │   ├── dexv2factory.h (DEXV2Factory)
│   │   │   ├── dexv2library.h (DEXV2Library)
│   │   │   ├── dexv2pair.h (DEXV2Pair)
│   │   │   ├── dexv2router02.h (DEXV2Router02)
│   │   │   └── uq112x112.h (UQ112x112 - used in DEX contracts for fixed-point fractions)
│   │   ├── erc20.h (ERC20)
│   │   ├── erc20wrapper.h (ERC20Wrapper)
│   │   ├── erc721.h (ERC721)
│   │   ├── erc721test.h (ERC721Test, used solely for testing purposes)
│   │   ├── erc721uristorage.h (ERC721URIStorage, converted from OpenZeppelin)
│   │   ├── nativewrapper.h (NativeWrapper)
│   │   ├── ownable.h (Ownable, converted from OpenZeppelin)
│   │   ├── pebble.h (Pebble)
│   │   ├── randomnesstest.h (RandomnessTest)
│   │   ├── simplecontract.h (SimpleContract)
│   │   ├── snailtracer.h, snailtraceroptimized.h (SnailTracer and SnailTracerOptimized, converted from the original EVM impl)
│   │   ├── testThrowVars.h (TestThrowVars, used solely for testing purposes)
│   │   └── throwtestA.h, throwtestB.h, throwtestC.h (for testing nested contract calls)
│   └── variables (Safe Variables for use within Dynamic Contracts)
│       ├── reentrancyguard.h (ReentrancyGuard)
│       ├── safeaddress.h (SafeAddress)
│       ├── safearray.h (SafeArray)
│       ├── safebase.h (SafeBase - used as base for all other types)
│       ├── safebool.h (SafeBool)
│       ├── safebytes.h (SafeBytes)
│       ├── safeint.h (SafeInt and respective aliases)
│       ├── safestring.h (SafeString)
│       ├── safetuple.h (SafeTuple)
│       ├── safeuint.h (SafeUint and respective aliases)
│       ├── safeunorderedmap.h (SafeUnorderedMap)
│       └── safevector.h (SafeVector)
├── core (Core components)
│   ├── blockchain.h (Blockchain, Syncer)
│   ├── consensus.h (Consensus)
│   ├── dump.h (Dumpable, DumpManager, DumpWorker)
│   ├── rdpos.h (Validator, rdPoS)
│   ├── state.h (BlockValidationStatus, State)
│   └── storage.h (Storage)
├── libs (Third-party libraries)
│   ├── BS_thread_pool_light.hpp (https://github.com/bshoshany/thread-pool)
│   ├── catch2/catch_amalgamated.hpp (https://github.com/catchorg/Catch2)
│   ├── json.hpp (https://github.com/nlohmann/json)
│   ├── wyhash.h (https://github.com/wangyi-fudan/wyhash)
│   └── zpp_bits.h (https://github.com/eyalz800/zpp_bits)
├── net (Networking)
│   ├── http (HTTP part of networking)
│   │   ├── httpclient.h (HTTPClient)
│   │   ├── httplistener.h (HTTPListener)
│   │   ├── httpparser.h (parser functions for HTTP requests)
│   │   ├── httpserver.h (HTTPServer)
│   │   ├── httpsession.h (HTTPQueue, HTTPSession)
│   │   └── jsonrpc (Namespace for handling JSONRPC data)
│   │       ├── blocktag.h (BlockTag, BlockTagOrNumber)
│   │       ├── call.h (call() - a function that processes a JSON RPC call)
│   │       ├── error.h (Error - for abstracting JSON RPC errors)
│   │       ├── methods.h (contains all implemented JSON RPC methods)
│   │       ├── parser.h (Parser and helper tempate functions)
│   │       └── variadicparser.h (VariadicParser and helper template functions)
│   └── p2p (P2P part of networking)
│       ├── broadcaster.h (Broadcaster)
│       ├── discovery.h (DiscoveryWorker - worker thread for ManagerDiscovery)
│       ├── encoding.h (collection of enums, structs, classes, encoders and decoders used in P2P communications)
│       ├── managerbase.h (ManagerBase - used as base for ManagerDiscovery and ManagerNormal)
│       ├── managerdiscovery.h (ManagerDiscovery)
│       ├── managernormal.h (ManagerNormal)
│       ├── nodeconns.h (NodeConns)
│       └── session.h (Session)
└── utils (Utility components)
    ├── clargs.h (definitions for helper functions, enums and structs that deal with command-line argument parsing)
    ├── contractreflectioninterface.h (ContractReflectionInterface - interface for registering Dynamic Contracts)
    ├── db.h (DBPrefix, DBServer, DBEntry, DBBatch, DB)
    ├── dynamicexception.h (DynamicException - custom exception class)
    ├── ecdsa.h (PrivKey, Pubkey, UPubkey, Secp256k1)
    ├── evmcconv.h (EVMCConv - namespace for EVMC-related data conversion functions)
    ├── finalizedblock.h (FinalizedBlock)
    ├── hex.h (Hex)
    ├── intconv.h (IntConv - namespace for signed integer aliases and data conversion functions)
    ├── jsonabi.h (JsonAbi - namespace for writing contract ABIs to JSON format)
    ├── logger.h (LogType, Log, LogInfo, Logger, LogicalLocationProvider)
    ├── merkle.h (Merkle)
    ├── options.h (Options singleton - generated by CMake through a .in file)
    ├── randomgen.h (RandomGen)
    ├── safehash.h (SafeHash, FNVHash)
    ├── strconv.h (StrConv - namespace for string-related data conversion and manipulation functions)
    ├── strings.h (FixedBytes and its derivatives - Hash, Functor, Signature, Address, StorageKey)
    ├── tx.h (TxBlock, TxValidator)
    ├── uintconv.h (UintConv - namespace for unsigned integer aliases and data conversion functions)
    └── utils.h (Utils namespace and other misc struct and enum definitions)
```
