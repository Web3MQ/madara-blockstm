# Introduction
We have developed a parallel transaction executor based on substrate. It is designed to replace the native substrate serial executor.
Below, We will provide the benchmarking process and offer some analysis. Finally, We will provide precompiled binary executable files to facilitate benchmarking locally.

**We've already integrated this solution into Madara. It will be released soon.**

# Benchmark 
The benchmark strategy for this session involves testing the `transfer_allow_death` transactions within the Balance module of Substrate.

We will generate N sender accounts and N receiver accounts. For each pair of sender and receiver, we will create one `transfer_allow_death` transaction. This process will result in N transactions in total.

## Benchmark Machine
```
Machine Name：MacBook Pro
Chip: Apple M2 Pro
Total Cores: 12 
Memory: 16 GB
```

## Benchmark Report
### Native Serial Executor
The native transaction executor of Substrate is serial. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 105 | 100000 | 952.38 |
| 2 | 105 | 100000 | 952.38 |
| 3 | 105 | 100000 | 952.38 |
| Avg | 105 | 100000 | 952.38 |

### 2-Threaded BlockSTM
When using the BlockSTM parallel executor with only 2 cores. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 14 | 66000 | 4714.28 |
| 2 | 14 | 66000 | 4714.28 |
| 3 | 14 | 66000 | 4714.28 |
| Avg | 105 | 100000 | 4714.28 |

| Executor | TPS | Factor|
|---|---|---|
| Substrate | 952.38 | 1.0 |
| 2ThreadBlockSTM | 4714.28 | 4.94 | 
| 4ThreadBlockSTM | 5333.33 | 5.6  |


| Executor | Speed (tx/s) |
|---|---|
| Substrate | 3703 |
| 2ThreadBlockSTM | 14705 |
| 4ThreadBlockSTM | 19607 |

# Quick Start
We provide some precompiled programs and testing tools for those interested in testing the performance of Substrate BlockSTM.
You can find these binary files in the [release section](https://github.com/Web3MQ/madara-blockstm/releases/tag/v0.0.1) of the current repository.

## Run BlockChain Node
The command to run BlockSTM executor node:
```
BLOCK_SIZE=30000 CONCURRENCY_LEVEL=2 ./node-template --dev --pool-limit 2000000
```
The `BLOCK_SIZE` and `CONCURRENCY_LEVEL` are controlled respectively by environment variables, specifying the transaction capacity per block and the number of threads used for concurrent processing. In the command mentioned above, each block accommodates 30,000 transactions, and concurrent processing is done using 2 threads.

The command to run  serial executor node
```
./node-template --dev --pool-limit 2000000
```

## Run Benchmark Tool
First, create a TOML configuration file.
```
# client url.
client_url = "ws://127.0.0.1:9944"

# account number
account_number =66000

# every account will send tx number
every_account_tx = 1
```
The above configuration file will connect to the node `ws://127.0.0.1:9944`, then create 66,000 accounts, with each account sending one transaction.

### Run Command
Begin the benchmark with the following command:
```
CONFIG_PATH=./config.toml ./benchmark 
```