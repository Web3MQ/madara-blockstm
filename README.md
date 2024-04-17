# Introduction
We have developed a parallel transaction executor based on substrate. It is designed to replace the native substrate serial executor.
Below, We will provide the benchmarking process and offer some analysis. Finally, We will provide precompiled binary executable files to facilitate benchmarking locally.

We have employed BlockSTM in two scenarios: one with [Madara](https://github.com/keep-starknet-strange/madara), a blockchain built on substrate utilizing the Cairo VM. And another with an AppChain built on Substrate, which does not integrate any VM.

# Madara BlockSTM Benchmark 
The benchmarking strategy for Madara in this session is as follows:
1. Deploy an ERC20 token.
2. Deploy N accounts.
3. Recharge transaction fees and ERC20 balances to the deployed N accounts.
4. Build N transactions using N accounts, where each account transfers funds to an address calculated using N+1 salt. This ensures that each transaction is unrelated.
5. Send the built transactions and record the time taken for all transactions to be included in a block.

## Madara BlockSTM Benchmark Machine
```
Machine Name：MacBook Pro
Chip: Apple M2 Pro
Total Cores: 12 
Memory: 16 GB
```
## Madara BlockSTM Benchmark Report
### Native Serial Executor
The native transaction executor of substrate is serial. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 12 | 1400 | 116.66 |
| 2 | 12 | 1400 | 116.66 |
| 3 | 13 | 1400 | 107.69 |
| Avg | 12.3 | 1400 | 113.82 |

We can observe in the node logs during benchmark:
```
🎁 Prepared block for proposing at 9466(2448 ms) [hash: 0x0a45b586f69c6f5be8c59496e6f00fbc463c9b9e169900c1bdecddf590008603; parent_hash: 0xcf2a…c29f; extrinsics (700):
```

This indicates that, with the native serial processor, it took 2448ms for 700 transactions.
Therefore, we can conclude that the processing speed of the native serial processor is 3.49ms/Tx.

### 2-Threaded BlockSTM
When using the BlockSTM parallel executor with only 2 threads. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 12 | 2400 | 200 |
| 2 | 13 | 2400 | 184.61 |
| 3 | 13 | 2400 | 184.61 |
| Avg | 12.3 | 2400 | 195.3 |

We can observe in the node logs during benchmark:
```
🎁 Prepared block for proposing at 9730 (1970 ms) [hash: 0x07a3cf9b154d9ba978fa26843f132bb4fc4a32ca7e14a9561cdd148df6b27b46; parent_hash: 0xc352…414b; ⚡️⚡️⚡️extrinsics (952)    
```
This indicates that, with the 2 threaded blockSTM executor, it took 1970ms for 952 transactions.
Therefore, we can conclude that the processing speed of the 2-threaded blockSTM executor is 2.06ms/Tx.

### 4-Threaded BlockSTM
When using the BlockSTM parallel executor with only 4 threads. Below are the results of our benchmark:
| | Duration(s) | Transaction Count| TPS|
|---|---|---|---|
| 1 | 11 | 4000 | 363.63 |
| 2 | 11 | 4000 | 363.63 |
| 3 | 11 | 4000 | 363.63 |
| Avg | 11 | 4000 | 363.63 |

We can observe in the node logs:
```
🎁 Prepared block for proposing at 9850 (999 ms) [hash: 0xbb3a836dc1bcc5902239d664a725a0ca0ec50e87940a86e2df4fa4cdd0306c6d; parent_hash: 0x1ded…a7d6; ⚡️⚡️⚡️extrinsics (994)    
```
This indicates that, with the 4-threaded blockSTM executor, it took 999ms for 994 transactions.
Therefore, we can conclude that the processing speed of the 4-threaded blockSTM executor is 1ms/Tx.

### Comparing TPS
The table below compares the TPS of various executors.
| Executor | TPS | Factor|
|---|---|---|
| Native Serial | 113.82 | 1.0 |
| 2ThreadBlockSTM | 195.3 | 1.71 | 
| 4ThreadBlockSTM | 363.63 | 3.19  |

Compared to Substrate transactions, individual Cairo transactions take longer to process. This results in fewer transactions that can be batched in a single block. Consequently, optimizations to the Runtime Interface within BlockSTM will not show significant improvements. Therefore, the performance enhancement of BlockSTM will be proportional to the number of CPU cores used. However, due to the reduced number of transactions, the proportion of time spent processing global keys relative to the entire transaction execution time decreases after parallel processing is completed. By increasing the number of threads further, higher performance can be achieved.

# Madara BlockSTM Benchmark Quick Start
We provide some precompiled programs and testing tools for those interested in testing the performance of Madara BlockSTM.
You can find these binary files in the [release section](https://github.com/Web3MQ/madara-blockstm/releases/tag/v0.0.1) of the current repository.

## Run BlockChain Node
The command to run Madara BlockSTM executor node:
```
./madara setup --from-remote
BLOCK_SIZE=30000 CONCURRENCY_LEVEL=2 ./madara --dev 
```
The `BLOCK_SIZE` and `CONCURRENCY_LEVEL` are controlled respectively by environment variables, specifying the transaction capacity per block and the number of threads used for concurrent processing. In the command mentioned above, each block accommodates 30,000 transactions, and concurrent processing is done using 2 threads.

The command to run  serial executor node
```
./madara setup --from-remote
./madara --dev 
```

## Run Benchmark Tool
Our benchmark script is written in Python. So, first ensure that Python 3 is installed.
- step1
```
pip install starknet-py==0.18.3 ujson aiofiles
```
- step2
```
python3 madara-benchmark.py
```

# Substrate BlockSTM Benchmark  
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
| 1 | 15 | 80000 | 5333.33 |
| 2 | 15 | 80000 | 5333.33 |
| 3 | 15 | 80000 | 5333.33 |
| Avg | 15 | 80000 | 5333.33 |

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

# Substrate BlockSTM Benchmark Quick Start
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