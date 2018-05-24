# 使用RPC接口来转账

本教程主要使用EOS的RPC接口来对发行的代币使用RPC接口来进行操作，下列实验均基于`eosio/eos:20180521`这个版本的EOS完成。

## 发布代币合约

#### 创建账号

```
cleos create account eosio eosio.token EOS5ySgzeHp9G7TqNDGpyzaCtahAeRcTvPRPJbFey5CmySL3vKYgE EOS5ySgzeHp9G7TqNDGpyzaCtahAeRcTvPRPJbFey5CmySL3vKYgE
```

操作结果:

```
executed transaction: 4a8b53ae6fa5e22ded33b50079e45550e39f3cb72ffa628e771ea21758844039  200 bytes  339 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.token","owner":{"threshold":1,"keys":[{"key":"EOS5ySgzeHp9G7TqNDGpy...
```

#### 部署合约

```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.token -p eosio.token
```

结果:

```
Reading WAST/WASM from /opt/eosio/bin/data-dir/contracts/eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 41677b5fd5c701ca67a153abb09f79c04085cc51a9d021436e7ee5afda1781bd  8048 bytes  1212 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1560037f7e7f0060057f7e...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d65...
```

#### 创建EOS代币

```
cleos push action eosio.token create '["eosio", "10000000000.0000 EOS",0,0,0]' -p eosio.token
```

结果:

```
executed transaction: 566693cba0b0d5d11d85e40cdfb095d525612c5915e17ce75d309054e1912235  120 bytes  552 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 EOS"}
```

给eosio发币

```
cleos push action eosio.token issue '["eosio","1000000000.0000 EOS", "issue"]' -p eosio
```

结果:

```
executed transaction: 73f72879d220c720fcefb16b6aaf3db0ba492bd62020853b2cd5051557d5fa87  128 bytes  677 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 EOS","memo":"issue"}
```

#### 查看余额

上面操作无误的话应该可以查询到余额:

```
cleos get currency balance eosio.token eosio
```

结果为:

```
1000000000.0000 EOS
```

## 部署系统合约

部署`eosio.system`合约

```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.system -p eosio.token
```

执行结果:

```
Reading WAST/WASM from /opt/eosio/bin/data-dir/contracts/eosio.system/eosio.system.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: f6fab6802bf8089b3ba48705f899e36fd681e58c622661ba2032eb1a85ee2d64  40440 bytes  8081 us
#         eosio <= eosio::setcode               "00a6823403ea30550000809d070061736d0100000001ba022f60027f7e0060067f7e7e7f7f7f0060057f7e7e7f7f0060047...
#         eosio <= eosio::setabi                "00a6823403ea3055a8250e656f73696f3a3a6162692f312e30050c6163636f756e745f6e616d65046e616d650f7065726d6...
```

## TODO: 新建账号

这里我们使用系统合约*eosio.system*的*newaccount*这个action来进行新建账号。

## 转账RPC

###abi_json_to_bin

首先我们需要序列化转账的json:

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

参数的意思分别为:

|   参数名称   |  参数类型  |                    描述                    |
| :------: | :----: | :--------------------------------------: |
|   code   | string |       智能合约的名字，这里用的是**eosio.token**       |
|  action  | string |    智能合约中的action, 这里用转账: **transfer**     |
|   from   | string |       **transfer**方法里面的参数，从哪个账户转账        |
|    to    | string |       **transfer**方法里面的参数，转账到哪个账户        |
| quantity | string | **transfer**方法里面的参数，转账数量，这里代币名称为EOS，还可以有其他代币 |
|   memo   | string |         **transfer**方法里面的参数，转账备注         |

这样会得到一个二进制的字符串，会用作后面的步骤中的`data`参数：

```
{
    "binargs": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d"
}
```

|  参数名称   |  参数类型  |                    描述                    |
| :-----: | :----: | :--------------------------------------: |
| binargs | string | 序列化的结果，在sign_transaction 和 push_transaction 中作为 **data** 请求参数 |

### get_info

我们使用get_info来获取最新的区块号:

```
GET http://127.0.0.1:8888/v1/chain/get_info
```

|      参数名称      |  参数类型  |  描述   |
| :------------: | :----: | :---: |
| head_block_num | number | 最新区块号 |

得到结果:

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

使用 **get_block**来获取最新的区块具体信息:

```
POST http://127.0.0.1:8888/v1/chain/get_block
```

DATA:

```
{"block_num_or_id":246190}
```

请求参数:

|      参数名称       |  参数类型  |  描述  |
| :-------------: | :----: | :--: |
| block_num_or_id | number | 区块号  |

得到结果:

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

参数含义:

|       参数名称       |  参数类型  |                    描述                    |
| :--------------: | :----: | :--------------------------------------: |
|    timestamp     | string |                 区块对应的时间戳                 |
|    block_num     | number | 区块号，作为sign_transaction 和 push_transaction中的 **ref_block_num**请求参数 |
| ref_block_prefix | number | 作为sign_transaction 和 push_transaction中的**ref_block_prefix** 请求参数 |

### unlock

在签名交易之前需要解锁钱包:

```
POST http://127.0.0.1:8888/v1/wallet/unlock
```

Data:

```
["noprom", "PW5KExxxxxS1HQzF1qJtWbxxxEFqSxdWBpgYUsxxxxxoxVRmjVw7Y"]
```

得到结果, 如果为空就返回下面的结果：

```
{}
```

### get_required_keys

我们需要用这个接口来获取所需要的key

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

得到结果:

```
{
    "required_keys": [
        "EOS6gXwNz2SKUNAZcyjzVvg6KdNgA1bSuVzCr8c5yWkGij52JKx8V"
    ]
}
```

结果中的key将作为**sign_transaction**的key

### sign_transaction

接着我们需要对转账交易进行签名：

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

参数含义:

|            参数名称            |  参数类型  |                 描述                  |
| :------------------------: | :----: | :---------------------------------: |
|       ref_block_num        | number |        **get_block**获得的最新区块号        |
|      ref_block_prefix      | number |      **get_block**获得的最新区块号相关信息      |
|         expiration         | string |   过期时间 = timestamp 加上 一段时间 ，例如1分钟   |
|          account           | string |   调用系统智能合约账号名，这里为**eosio.token**    |
|            name            | string |  **eosio.token**合约的**transfer**方法   |
|    authorization. actor    | string |              执行操作的用户名               |
| authorization.  permission | string |               执行操作的权限               |
|            data            | string | **abi_json_to_bin** 序列化后的 值 binargs |
|                            | string |               创建者的公钥                |

得到结果:

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

最后就是发送交易:

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

参数含义:

|            参数名称            |  参数类型  |                描述                |
| :------------------------: | :----: | :------------------------------: |
|       ref_block_num        | number |      **get_block**获得的最新区块号       |
|      ref_block_prefix      | number |    **get_block**获得的最新区块号相关信息     |
|         expiration         | string | 过期时间 = timestamp 加上 一段时间 ，例如1分钟  |
|          account           | string |  调用系统智能合约账号名，这里为**eosio.token**  |
|            name            | string | **eosio.token**合约的**transfer**方法 |
|    authorization. actor    | string |             执行操作的用户名             |
| authorization.  permission | string |             执行操作的权限              |
|         signatures         | string |    **sign_transaction** 得到的签名    |

得到结果

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

这样就完成了用RPC API来转账的功能。