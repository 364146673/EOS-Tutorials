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
    "head_block_num": 167172,
    "last_irreversible_block_num": 167171,
    "last_irreversible_block_id": "00028d0306c1be5743f920a5423b6101497c1b1cb7a92ee478bbe1cbaff1f487",
    "head_block_id": "00028d04b87fe52bee1171bddd2af9fc6f8e2f5cd3019eed27a8aba55ff610ea",
    "head_block_time": "2018-05-24T03:41:03",
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
{"block_num_or_id":167172}
```

请求参数:

|      参数名称       |  参数类型  |  描述  |
| :-------------: | :----: | :--: |
| block_num_or_id | number | 区块号  |

得到结果:

```
{
    "timestamp": "2018-05-24T03:41:03.000",
    "producer": "eosio",
    "confirmed": 0,
    "previous": "00028d0306c1be5743f920a5423b6101497c1b1cb7a92ee478bbe1cbaff1f487",
    "transaction_mroot": "0000000000000000000000000000000000000000000000000000000000000000",
    "action_mroot": "55fc5e788439f33c27f3e890f1f171307c93650d51a3709a1225ea136c87fdd5",
    "schedule_version": 0,
    "new_producers": null,
    "header_extensions": [],
    "producer_signature": "SIG_K1_KBUdejuynu9VWiJBHjocrAz3PthzyVxwYxFkkCEuit7J37iEpHK8i3hXj9jUGsmHY77C452qpQp7mdvaAivMc1CYVH4zBD",
    "transactions": [],
    "block_extensions": [],
    "id": "00028d04b87fe52bee1171bddd2af9fc6f8e2f5cd3019eed27a8aba55ff610ea",
    "block_num": 167172,
    "ref_block_prefix": 3178303982
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

### **sign_transaction**

接着我们需要对转账交易进行签名：

```
POST http://127.0.0.1:8888/v1/wallet/sign_transaction
```

Data:

```
[
  {
    "ref_block_num": 168302,
    "ref_block_prefix": 627421541,
    "expiration": "2018-05-24T05:50:28.000",
    "actions": [
      {
        "account": "eosio.token",  //有 transfer 的 action 的智能合约账号
        "name": "transfer",
        "authorization": [
          {
            "actor": "eosio",
            "permission": "active"
          }
        ],
        "data": "0000000000ea305500000000487a2b9d102700000000000004454f53000000001163726561746564206279206e6f70726f6d" // //abi_json_to_bin 的响应参数 binargs
      }
    ],
    "signatures": []
  },
  [
    "EOS5ySgzeHp9G7TqNDGpyzaCtahAeRcTvPRPJbFey5CmySL3vKYgE" //创建者的公钥（交易发起者的公钥），其实是用的公钥对应的私钥进行签名的，签名前需要先解锁包含此私钥的钱包
  ],
  ""
]
```

得到结果:

```
{
    "expiration": "2018-05-24T05:50:28",
    "ref_block_num": 37230,
    "ref_block_prefix": 627421541,
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
        "SIG_K1_KiqCGCz3NWykgiLorC1g3CjRp1weBJf7Kp5AMypzdyBGUkg7TB9gpX7BCt3hdgGtqYMmtBPt1ZFtyQdTzUVL1EiqvWFNmH"
    ],
    "context_free_data": []
}
```

