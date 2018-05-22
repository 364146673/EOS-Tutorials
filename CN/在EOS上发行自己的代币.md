# 在EOS上发行自己的代币

这篇文章主要介绍如何在EOS上面发行自己的代币，下列实验均基于`eosio/eos:20180521`这个版本的EOS完成。

## 创建 KEY
这里首先创建一对key:
```
cleos create key
```
然后会得到一对key, 这里略去 `Private key`
```
Private key: ************************************
Public key: EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc
```
## 创建钱包
然后我们创建一个钱包，这里暂且将这个钱包叫做`token`，其中`-n`用来指定钱包名字
```
cleos wallet create -n token
```
得到钱包密码:
```
***********************************
```
## 导入私钥
然后导入我们第一步创建的私钥:
```
cleos wallet import -n token *******(请将此处替换为第一步中得到的Private key)
```

## 创建账户
然后我们可以用第一步得到的`public key`来创建一个账户`noprom`:
```
cleos create account eosio noprom EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc
```
得到结果:
```
executed transaction: 12bb1933cf9363699abebe421bcb22e8b39c86065505f5528c171857a99d1613  200 bytes  364 us
eosio <= eosio::newaccount            {"creator":"eosio","name":"noprom","owner":{"threshold":1,"keys":[{"key":"EOS8NFJ49egRRjp4j2kySikZPZ...
```
这里的参数的意思是，由`eosio`这个账户来创建一个叫做`noprom`的账户，这个账户的公钥为第一步中创建的`EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc`, 这里为了简单我们暂时将2中类型的key都设置为同一个，这两个key分别为`owner`和`active`的key。
同理，我们再创建一个叫做小明的账户:
```
cleos create account eosio xiaoming EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc
```
得到结果:
```
executed transaction: 5fba9946ddb63e3a438875ada8be009771ff2b8b08140db36b4f1f2155fe46f2  200 bytes  350 us
eosio <= eosio::newaccount            {"creator":"eosio","name":"xiaoming","owner":{"threshold":1,"keys":[{"key":"EOS8NFJ49egRRjp4j2kySikZ...
```
## 部署代币合约
首先创建一个专门部署这个合约的账号:
```
cleos create account eosio eosio.token EOS69CVnKwKMbNfGpdjfYMdLrvUwUenHDbSw9pFQHT9iaFUTWSA8Q EOS69CVnKwKMbNfGpdjfYMdLrvUwUenHDbSw9pFQHT9iaFUTWSA8Q
```
然后我们部署`eosio.token`这个合约:
```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.token -p eosio.token
```
这样合约就部署完了。
## 发行代币
```
cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOSDEVS", 0, 0, 0]' -p eosio.token
```
得到结果:
```
executed transaction: 31508996812b5d6369c099e30047fab919b294baba12eae434f755a6aef2327a  120 bytes  423 us
eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 EOSDEVS"}
```

这里我们执行了`eosio.token`这个合约里面的`create`方法，这个方法的作用就是创建一个代币。
这里我们创建了一个叫做`EOSDEVS`的代币，发行者为`eosio`这个账户，发行总量为`1000000000.0000`，使用`eosio.token`这个用户来创建这个代币。
## 给用户发币
接着我们可以给刚才创建的用户发放我们刚才创建的`EOSDEVS`代币:
```
cleos push action eosio.token issue '[ "noprom", "10.0000 EOSDEVS", "created by eosdevs" ]' -p eosio
```
操作结果:
```
executed transaction: 82059a49ada36b1124359314b6a139eba2e4b917da8240aec1f8a0ac378249c9  136 bytes  1162 us
eosio.token <= eosio.token::issue           {"to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
eosio.token <= eosio.token::transfer        {"from":"eosio","to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
eosio <= eosio.token::transfer        {"from":"eosio","to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
noprom <= eosio.token::transfer        {"from":"eosio","to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
```
这样就给`noprom`这个账户发了10个`EOSDEVS`代币，并加了一个备注: `created by eosdevs`。
## 获取账户余额
我们可以通过下面的方法来获得用户的账户余额:
```
cleos get currency balance eosio.token noprom
```
可以得到:
```
10.0000 EOSDEVS
```
其中`eosio.token`为我们部署的合约，`noprom`为账户名。
## 转账
要转账，我们可以调用`eosio.token`这个合约里面的`transfer`方法:
```
cleos push action eosio.token transfer '[ "noprom", "xiaoming", "5.0000 EOSDEVS", "Transfer from noprom to xiaoming" ]' -p noprom
```
操作结果:
```
executed transaction: 46d84306ee989d3c75235b2271ab9baed336b3fe403473357cbad3cf4094c756  160 bytes  937 us
eosio.token <= eosio.token::transfer        {"from":"noprom","to":"xiaoming","quantity":"5.0000 EOSDEVS","memo":"Transfer from noprom to xiaomin...
noprom <= eosio.token::transfer        {"from":"noprom","to":"xiaoming","quantity":"5.0000 EOSDEVS","memo":"Transfer from noprom to xiaomin...
xiaoming <= eosio.token::transfer        {"from":"noprom","to":"xiaoming","quantity":"5.0000 EOSDEVS","memo":"Transfer from noprom to xiaomin...
```
这里我们从`noprom`这个账户向`xiaoming`这个账户转了5个`EOSDEVS` 代币，备注为`Transfer from noprom to xiaoming`。
然后我们可以查看`noprom`和`xiaoming`的账户余额:
```
cleos get currency balance eosio.token noprom
5.0000 EOSDEVS
cleos get currency balance eosio.token xiaoming
```
可以看到转账成功，`noprom`和`xiaoming` 都有5个`EOSDEVS` 代币。