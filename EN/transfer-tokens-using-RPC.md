# Using EOS RPC API to Transfer EOS

This tutorial mainly uses the RPC API of EOS to use the RPC interface to operate the issued tokens. The following experiments are based on the EOS of `eosio/eos:20180521`.

## Deploy Token Contract

#### Create an account

```
cleos create account eosio eosio.token EOS5ySgzeHp9G7TqNDGpyzaCtahAeRcTvPRPJbFey5CmySL3vKYgE EOS5ySgzeHp9G7TqNDGpyzaCtahAeRcTvPRPJbFey5CmySL3vKYgE
```

Operation result:

```
executed transaction: 4a8b53ae6fa5e22ded33b50079e45550e39f3cb72ffa628e771ea21758844039  200 bytes  339 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS5ySgzeHp9G7TqNDGpy...
```

#### Deployment Contract

```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.token -p eosio.token
```

result:

```
Reading WAST/WASM from /opt/eosio/bin/data-dir/contracts/eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 41677b5fd5c701ca67a153abb09f79c04085cc51a9d021436e7ee5afda1781bd  8048 bytes  1212 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1560037f7e7f0060057f7e...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d65...
```

#### Creating EOS tokens

```
cleos push action eosio.token create '["eosio", "10000000000.0000 EOS",0,0,0]' -p eosio.token
```

result:

```
executed transaction: 566693cba0b0d5d11d85e40cdfb095d525612c5915e17ce75d309054e1912235  120 bytes  552 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 EOS"}
```

Send coins to eosio

```
cleos push action eosio.token issue '["eosio","1000000000.0000 EOS", "issue"]' -p eosio
```

result:

```
executed transaction: 73f72879d220c720fcefb16b6aaf3db0ba492bd62020853b2cd5051557d5fa87  128 bytes  677 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 EOS","memo":"issue"}
```

#### Check balance

If the above operation were correct, you should be able to query the balance:

```
cleos get currency balance eosio.token eosio
```

result:

```
1000000000.0000 EOS
```

## Deploying System Contracts

Deploy the `eosio.system` contract

```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.system -p eosio.token
```

result:

```
Reading WAST/WASM from /opt/eosio/bin/data-dir/contracts/eosio.system/eosio.system.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: f6fab6802bf8089b3ba48705f899e36fd681e58c622661ba2032eb1a85ee2d64  40440 bytes  8081 us
#         eosio <= eosio::setcode               "00a6823403ea30550000809d070061736d0100000001ba022f60027f7e0060067f7e7e7f7f7f0060057f7e7e7f7f0060047...
#         eosio <= eosio::setabi                "00a6823403ea3055a8250e656f73696f3a3a6162692f312e30050c6163636f756e745f6e616d65046e616d650f7065726d6...
```

## TODO: New Account

Here we use the *newaccount* action of the system contract *eosio.system* to create a new account.

## RPC usage of transfer

### abi_json_to_bin

First we need to serialize the json of the transfer:

```
POST http://127.0.0.1:8888/v1/chain/abi_json_to_bin
```

Data:

```
{
  "code": "eosio.token",
  "action": "transfer",
  "args": {
    "from": "eosio",
    "to": "noprom",
    "quantity": "1.0000 EOS",
    "memo": "created by noprom"
  }
}
```

The meaning of the parameters are:

| Parameter Name | Parameter Type |               Description                |
| :------------: | :------------: | :--------------------------------------: |
|      code      |     string     | The name of the smart contract, used here is **eosio.token** |
|     action     |     string     | action in a smart contract, with a transfer here: **transfer** |
|      from      |     string     | **transfer** method parameters, from which account transfer |
|       to       |     string     | **transfer** method parameters, transfer to which account |
|    quantity    |     string     | **transfer** method parameters, number of transfers, token name here is EOS, there may be other tokens |
|      memo      |     string     | **transfer** method parameters, transfer notes |

This will get a binary string that will be used as the `data` parameter in the following steps:

```
{
    "binargs": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
}
```
| Parameter Name | Parameter Type | Description                              |
| :------------: | :------------: | ---------------------------------------- |
|    binargs     |     string     | Serialized results as **data** request parameters in sign_transaction and push_transaction |

### get_info

We use **get_info** to get the latest block number:

```
GET http://127.0.0.1:8888/v1/chain/get_info
```

| Parameter Name | Parameter Type |     Description     |
| :------------: | :------------: | :-----------------: |
| head_block_num |     number     | latest block number |

Got the result:

```
{
    "server_version": "4e99cf47",
    "head_block_num": 246190,
    "last_irreversible_block_num": 246189,
    "last_irreversible_block_id": "0003c1ade32d4660e7872abaedb9413d7de2eefea63dd17eea6bd36dc3c11ac4",
    "head_block_id": "0003c1ae3018e5cb2bda7c6897b418e60eaf8d1a00622d731d7337c264dca453",
    "head_block_time": "2018-05-24T14:39:32",
    "head_block_producer": "eosio",
    "virtual_block_cpu_limit": 100000000,
    "virtual_block_net_limit": 1048576000,
    "block_cpu_limit": 99900,
    "block_net_limit": 1048576
}
```

### get_block

Use **get_block** to get the latest block specific information:

```
POST http://127.0.0.1:8888/v1/chain/get_block
```

DATA:

```
{"block_num_or_id":246190}
```

Request parameters:

| Parameter Name  | Parameter Type | Description  |
| :-------------: | :------------: | :----------: |
| Block_num_or_id |     number     | block number |

Got the result:

```
{
    "timestamp": "2018-05-24T14:39:32.000",
    "producer": "eosio",
    "confirmed": 0,
    "previous": "0003c1ade32d4660e7872abaedb9413d7de2eefea63dd17eea6bd36dc3c11ac4",
    "transaction_mroot": "0000000000000000000000000000000000000000000000000000000000000000",
    "action_mroot": "953a01dbf975cc120285e0c4e9fbe4be181189a821de7f2da467bcb373dce8f2",
    "schedule_version": 0,
    "new_producers": null,
    "header_extensions": [],
    "producer_signature": "SIG_K1_JvQ5qzgSJ4L1jYYZH2L7oM7D7hp37ejENN8UQL9AvnKpNvbCeF8gAzAKXUYvocH9fEUg2bxKGbU66QGRCjVGzxEXN7HTL6",
    "transactions": [],
    "block_extensions": [],
    "id": "0003c1ae3018e5cb2bda7c6897b418e60eaf8d1a00622d731d7337c264dca453",
    "block_num": 246190,
    "ref_block_prefix": 1753012779
}
```

The meaning of the parameters:

|  Parameter Name  | Parameter Type | Description                              |
| :--------------: | :------------: | ---------------------------------------- |
|    Timestamp     |     string     | Timestamp of the block                   |
|    Block_num     |     number     | block number as the **ref_block_num** request parameter in sign_transaction and push_transaction |
| ref_block_prefix |     number     | as **ref_block_prefix** request parameters in sign_transaction and push_transaction |

### unlock

To unlock the wallet before signing the transaction:

```
POST http://127.0.0.1:8888/v1/wallet/unlock
```

Data:

```
["noprom", "PW5KExxxxxS1HQzF1qJtWbxxxEFqSxdWBpgYUsxxxxxoxVRmjVw7Y"]
```

Get the result, if it is empty return the following result:

```
{}
```

### get_required_keys

We need to use this interface to get the required key

```
POST http://127.0.0.1:8888/v1/chain/get_required_keys
```

DATA:

```
{
    "available_keys": [
        "EOS5ySgzeHp9G7TqNDGpyzaCtahAeRcTvPRPJbFey5CmySL3vKYgE",
        "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
        "EOS6gXwNz2SKUNAZcyjzVvg6KdNgA1bSuVzCr8c5yWkGij52JKx8V"
    ],
    "transaction": {
        "actions": [
            {
                "account": "eosio.token",
                "authorization": [
                    {
                        "actor": "eosio",
                        "permission": "active"
                    }
                ],
                "data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d",
                "name": "transfer"
            }
        ],
        "context_free_actions": [
        ],
        "context_free_data": [
        ],
        "delay_sec": 0,
        "expiration": "2018-05-24T15:20:30.500",
        "max_kcpu_usage": 0,
        "max_net_usage_words": 0,
        "ref_block_num": 245107,
        "ref_block_prefix": 801303063,
        "signatures": [
        ]
    }
}
```

Got the result:

```
{
    "required_keys": [
        "EOS6gXwNz2SKUNAZcyjzVvg6KdNgA1bSuVzCr8c5yWkGij52JKx8V"
    ]
}
```

The key in the result will be the key to **sign_transaction**

### sign_transaction

Then we need to sign the transfer transaction:

```
POST http://127.0.0.1:8888/v1/wallet/sign_transaction
```

Data:

```
[
  {
    "ref_block_num": 246190,
    "ref_block_prefix": 1753012779,
    "expiration": "2018-05-24T15:30:32.000",
    "actions": [
      {
        "account": "eosio.token",
        "name": "transfer",
        "authorization": [
          {
            "actor": "eosio",
            "permission": "active"
          }
        ],
        "data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
      }
    ],
    "signatures": []
  },
  [
    "EOS6gXwNz2SKUNAZcyjzVvg6KdNgA1bSuVzCr8c5yWkGij52JKx8V"
  ],
  ""
]
```

The meaning of the parameters:

|      Parameter Name       | Parameter Type | Description                              |
| :-----------------------: | :------------: | ---------------------------------------- |
|       ref_block_num       |     number     | Latest block number obtained by **get_block** |
|     ref_block_prefix      |     number     | **get_block** Get the latest block number information |
|        expiration         |     string     | expiration time = timestamp plus time, eg 1 minute |
|          Account          |     string     | Call system smart contract account name, here is **eosio.token** |
|           name            |     string     | **transfer** method of **eosio.token** contract |
|   Authorization. actor    |     string     | user name to perform the operation       |
| authorization. permission |     string     | permission to perform operations         |
|           data            |     string     | **abi_json_to_bin** Serialized Values ​​binargs |
|                           |     string     | creator's public key                     |

Got the result:

```
{
    "expiration": "2018-05-24T15:30:32",
    "ref_block_num": 49582,
    "ref_block_prefix": 1753012779,
    "max_net_usage_words": 0,
    "max_cpu_usage_ms": 0,
    "delay_sec": 0,
    "context_free_actions": [],
    "actions": [
        {
            "account": "eosio.token",
            "name": "transfer",
            "authorization": [
                {
                    "actor": "eosio",
                    "permission": "active"
                }
            ],
            "data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
        }
    ],
    "transaction_extensions": [],
    "signatures": [
        "SIG_K1_Khn918pY1NHmnbF41bsqFE7sPYrniZPtTns68qUo3m92jp6gbegkpRHYSp9RH95T3u82XUvjZLM33AP83ZqiGApBo7JnBF"
    ],
    "context_free_data": []
}
```

### push_transaction

The last is to send the transaction:

```
POST http://127.0.0.1:8888/v1/chain/push_transaction
```

Data:

```
{
  "compression": "none",
  "transaction": {
    "expiration": "2018-05-24T15:30:32",
    "ref_block_num": 49582,
    "ref_block_prefix": 1753012779,
    "context_free_actions": [],
    "actions": [
        {
            "account": "eosio.token",
            "name": "transfer",
            "authorization": [
                {
                    "actor": "eosio",
                    "permission": "active"
                }
            ],
            "data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
        }
    ],
    "transaction_extensions": []
  },
  "signatures": [
        "SIG_K1_Khn918pY1NHmnbF41bsqFE7sPYrniZPtTns68qUo3m92jp6gbegkpRHYSp9RH95T3u82XUvjZLM33AP83ZqiGApBo7JnBF"
   ]
}
```

The meaning of the parameters:

|      Parameter Name       | Parameter Type | Description                              |
| :-----------------------: | :------------: | ---------------------------------------- |
|       ref_block_num       |     number     | Latest block number obtained by **get_block** |
|     ref_block_prefix      |     number     | **get_block** Get the latest block number information |
|        expiration         |     string     | expiration time = timestamp plus time, eg 1 minute |
|          account          |     string     | Call system smart contract account name, here is **eosio.token** |
|           name            |     string     | **transfer** method of **eosio.token** contract |
|   authorization. actor    |     string     | user name to perform the operation       |
| authorization. permission |     string     | permission to perform operations         |
|        signatures         |     string     | **sign_transaction** Signed              |

Got the result:

```
{
    "transaction_id": "015ba92c7ad7294f0d70c772e7ba6ed678b11734418bf9ec48b001ce65c48e2f",
    "processed": {
        "id": "015ba92c7ad7294f0d70c772e7ba6ed678b11734418bf9ec48b001ce65c48e2f",
        "receipt": {
            "status": "executed",
            "cpu_usage_us": 644,
            "net_usage_words": 18
        },
        "elapsed": 644,
        "net_usage": 144,
        "scheduled": false,
        "action_traces": [
            {
                "receipt": {
                    "receiver": "eosio.token",
                    "act_digest": "77d11fbbb7e6a5d67c28c2578bc2042704ed76d494d7e426eaecb54dceb0dc0b",
                    "global_sequence": 246506,
                    "recv_sequence": 12,
                    "auth_sequence": [
                        [
                            "eosio",
                            246499
                        ]
                    ],
                    "code_sequence": 1,
                    "abi_sequence": 1
                },
                "act": {
                    "account": "eosio.token",
                    "name": "transfer",
                    "authorization": [
                        {
                            "actor": "eosio",
                            "permission": "active"
                        }
                    ],
                    "data": {
                        "from": "eosio",
                        "to": "noprom",
                        "quantity": "1.0000 EOS",
                        "memo": "created by noprom"
                    },
                    "hex_data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
                },
                "elapsed": 446,
                "cpu_usage": 0,
                "console": "",
                "total_cpu_usage": 0,
                "trx_id": "015ba92c7ad7294f0d70c772e7ba6ed678b11734418bf9ec48b001ce65c48e2f",
                "inline_traces": [
                    {
                        "receipt": {
                            "receiver": "eosio",
                            "act_digest": "77d11fbbb7e6a5d67c28c2578bc2042704ed76d494d7e426eaecb54dceb0dc0b",
                            "global_sequence": 246507,
                            "recv_sequence": 246488,
                            "auth_sequence": [
                                [
                                    "eosio",
                                    246500
                                ]
                            ],
                            "code_sequence": 1,
                            "abi_sequence": 1
                        },
                        "act": {
                            "account": "eosio.token",
                            "name": "transfer",
                            "authorization": [
                                {
                                    "actor": "eosio",
                                    "permission": "active"
                                }
                            ],
                            "data": {
                                "from": "eosio",
                                "to": "noprom",
                                "quantity": "1.0000 EOS",
                                "memo": "created by noprom"
                            },
                            "hex_data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
                        },
                        "elapsed": 3,
                        "cpu_usage": 0,
                        "console": "",
                        "total_cpu_usage": 0,
                        "trx_id": "015ba92c7ad7294f0d70c772e7ba6ed678b11734418bf9ec48b001ce65c48e2f",
                        "inline_traces": []
                    },
                    {
                        "receipt": {
                            "receiver": "noprom",
                            "act_digest": "77d11fbbb7e6a5d67c28c2578bc2042704ed76d494d7e426eaecb54dceb0dc0b",
                            "global_sequence": 246508,
                            "recv_sequence": 7,
                            "auth_sequence": [
                                [
                                    "eosio",
                                    246501
                                ]
                            ],
                            "code_sequence": 1,
                            "abi_sequence": 1
                        },
                        "act": {
                            "account": "eosio.token",
                            "name": "transfer",
                            "authorization": [
                                {
                                    "actor": "eosio",
                                    "permission": "active"
                                }
                            ],
                            "data": {
                                "from": "eosio",
                                "to": "noprom",
                                "quantity": "1.0000 EOS",
                                "memo": "created by noprom"
                            },
                            "hex_data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
                        },
                        "elapsed": 4,
                        "cpu_usage": 0,
                        "console": "",
                        "total_cpu_usage": 0,
                        "trx_id": "015ba92c7ad7294f0d70c772e7ba6ed678b11734418bf9ec48b001ce65c48e2f",
                        "inline_traces": []
                    }
                ]
            }
        ],
        "except": null
    }
}
```

This completes the function of transferring funds using the RPC API.