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

## 新建账号

这里我们使用系统合约*eosio.system*的*newaccount*这个action来进行新建账号。