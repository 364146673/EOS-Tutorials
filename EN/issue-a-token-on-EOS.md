# Issue your own tokens on EOS

This article mainly introduces how to issue your own tokens on EOS. The following experiments are based on the version of `EOSO/EOS:20180521`.

## Create KEY

Here first create a pair of keys:

```
cleos create key
```

Then I get a pair of keys, here I omit the `Private key`

```
Private key: ************************************
Public key: EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc
```

## Create Wallet

Then we create a wallet, and here is the wallet called `token`, where `-n` is used to specify the wallet name.

```
cleos wallet create -n token
```

Get wallet password:

```
***********************************
```

## Import private key

Then import the private key we created in the first step:

```
cleos wallet import -n token *******(please replace this with the private key obtained in the first step)
```

## Create Account

Then we can create an account `noprom` with the `public key` from the first step:

```
cleos create account eosio noprom EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc
```

Got the result:

```
executed transaction: 12bb1933cf9363699abebe421bcb22e8b39c86065505f5528c171857a99d1613  200 bytes  364 us
eosio <= eosio::newaccount            {"creator":"eosio","name":"noprom","owner":{"threshold":1,"keys":[{"key":"EOS8NFJ49egRRjp4j2kySikZPZ...
```

The parameter here means that an account named `noprom` is created by the account of eosio. The public key of this account is `EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc` created in the first step. For the sake of simplicity, we temporarily set the keys of type 2 to the same one. The two keys are the `owner` and `active` keys.

In the same way, we create another account called `xiaoming`:

```
cleos create account eosio xiaoming EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc EOS8NFJ49egRRjp4j2kySikZPZ68rWQy5KAR12wygooLPw8ThUqAc
```

Got the result:

```
executed transaction: 5fba9946ddb63e3a438875ada8be009771ff2b8b08140db36b4f1f2155fe46f2  200 bytes  350 us
eosio <= eosio::newaccount            {"creator":"eosio","name":"xiaoming","owner":{"threshold":1,"keys":[{"key":"EOS8NFJ49egRRjp4j2kySikZ...
```

## Deploy a token contract

First create an account that specifically deploys this contract:

```
cleos create account eosio eosio.token EOS69CVnKwKMbNfGpdjfYMdLrvUwUenHDbSw9pFQHT9iaFUTWSA8Q EOS69CVnKwKMbNfGpdjfYMdLrvUwUenHDbSw9pFQHT9iaFUTWSA8Q
```

Then we deploy the contract `eosio.token`:

```
cleos set contract eosio.token /opt/eosio/bin/data-dir/contracts/eosio.token -p eosio.token
```

This will complete the deployment of the contract.

## Issue a token

```
cleos push action eosio.token create '[ "eosio", "1000000000.0000 EOSDEVS", 0, 0, 0]' -p eosio.token
```

Got the result:

```
executed transaction: 31508996812b5d6369c099e30047fab919b294baba12eae434f755a6aef2327a  120 bytes  423 us
eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 EOSDEVS"}
```

Here we implement the `create` method in the contract of `eosio.token`. The purpose of this method is to create a token.

Here we have created a token called `EOSDEVS`. The issuer is the `eosio` account. The total number of issues is `1000000000.0000`. Use the user `eosio.token` to create this token.

## Send token to user

Then we can issue the `EOSDEVS` token we just created to the user we just created:

```
cleos push action eosio.token issue '[ "noprom", "10.0000 EOSDEVS", "created by eosdevs" ]' -p eosio
```

Got the result:

```
executed transaction: 82059a49ada36b1124359314b6a139eba2e4b917da8240aec1f8a0ac378249c9  136 bytes  1162 us
eosio.token <= eosio.token::issue           {"to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
eosio.token <= eosio.token::transfer        {"from":"eosio","to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
eosio <= eosio.token::transfer        {"from":"eosio","to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
noprom <= eosio.token::transfer        {"from":"eosio","to":"noprom","quantity":"10.0000 EOSDEVS","memo":"created by eosdevs"}
```

This action sends 10 `EOSDEVS` tokens to the `noprom` account and adds a note: `created by eosdevs`.

## Get account balance

We can get the user's account balance by the following method:

```
cleos get currency balance eosio.token noprom
```

You can get:

```
10.0000 EOSDEVS
```

Where `eosio.token` is the contract we deploy and `noprom` is the account name.

## Transfer

To transfer, we can call the `transfer` method in the contract `eosio.token`:

```
cleos push action eosio.token transfer '[ "noprom", "xiaoming", "5.0000 EOSDEVS", "Transfer from noprom to xiaoming" ]' -p noprom
```

Got the result:

```
executed transaction: 46d84306ee989d3c75235b2271ab9baed336b3fe403473357cbad3cf4094c756  160 bytes  937 us
eosio.token <= eosio.token::transfer        {"from":"noprom","to":"xiaoming","quantity":"5.0000 EOSDEVS","memo":"Transfer from noprom to xiaomin...
noprom <= eosio.token::transfer        {"from":"noprom","to":"xiaoming","quantity":"5.0000 EOSDEVS","memo":"Transfer from noprom to xiaomin...
xiaoming <= eosio.token::transfer        {"from":"noprom","to":"xiaoming","quantity":"5.0000 EOSDEVS","memo":"Transfer from noprom to xiaomin...
```

Here we transferred 5 `EOSDEVS` tokens from the `noprom` account to `xiaoming`. The note is `Transfer from noprom to xiaoming`.

Then we can check the account balance of `noprom` and `xiaoming`:

```shell
cleos get currency balance eosio.token noprom
5.0000 EOSDEVS
cleos get currency balance eosio.token xiaoming
```

You can see that the transfer was successful. `noprom` and `xiaoming` have 5 `EOSDEVS` tokens.

