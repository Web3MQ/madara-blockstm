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
When using the BlockSTM parallel executor with only 2 threads. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 14 | 66000 | 4714.28 |
| 2 | 14 | 66000 | 4714.28 |
| 3 | 14 | 66000 | 4714.28 |
| Avg | 14 | 66000 | 4714.28 |

### 4-Threaded BlockSTM
When using the BlockSTM parallel executor with only 4 threads. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 14 | 66000 | 5333.33 |
| 2 | 14 | 66000 | 5333.33 |
| 3 | 14 | 66000 | 5333.33 |
| Avg | 14 | 100000 | 5333.33 |

### Comparing TPS
The table below compares the TPS of various executors.
| Executor | TPS | Factor|
|---|---|---|
| Substrate | 952.38 | 1.0 |
| 2ThreadBlockSTM | 4714.28 | 4.94 | 
| 4ThreadBlockSTM | 5333.33 | 5.6  |

### Performance Analysis 
Moving from a single thread to two threads, we achieved a significant improvement, even surpassing the expected multiplier from the increase in threads. This is because BlockSTM not only modifies the executor but also changes the Runtime interface. In the original Substrate architecture, each transaction undergoes a complex process of calls and a series of encoding and decoding steps from the BlockBuilder to the Runtime API and then to the Executor. BlockSTM reduces this process to one step.

**However, the performance improvement from two threads to four threads did not meet expectations. This is because, after parallelizing transaction execution, we need to handle global key in a serial manner. As the number of transactions increases, the time consumed by this process also increases. After optimizing this process, performance will be further improved.**

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