# Currency Contract

## Creating a Wallet

Any transaction sent to the blockchain will need to be signed by the respective authority's private key. Before you will be able to sign the transaction, you will need a wallet to store and manage the private key.

With EOS, `eosc' creates wallets:
```bash
./eosc wallet create
# Creating wallet: default
# Save password to use in the future to unlock this wallet.
# Without password imported keys will not be retrievable.
# "PW5JD9cw9YY288AXPvnbwUk5JK4Cy6YyZ83wzHcshu8F2akU9rRWE"
```
However, for development and test use, this way is annoying and troublesome. Especially, manipulation with numerous password keys is error-prone, to us. To ease our study, we develop wraps on EOS processes. For example, `eoscWalletCreate` runs the above process after deleting a previous instance of the default wallet, if such one exists, and, subsequently defines an public variable set to the password of the wallet.

Note that `eoscWalletCreate` has to be used as the argument of the `source` command (see --help):

```bash
cd $EOS_PROGRAMS/eosc
. ./eoscWalletCreate
# $defaultWalletPswd=PW5KfPzdpyKXEJ7Yhw2BFzjypZE6FoVcmS4Fww93KNT78vr5hVgt5
```
You can create multiple wallets by specifying unique name:
```bash
. ./eoscWalletCreate second-wallet
# $second_walletWalletPswd=PW5J6MsiNbgNu5ThgsZXk6uiVBC6o8SD1AW1jLo8XFgFFLWFbgqTe
```
And you will be see it in the list of your wallets:
```bash
./eosc wallet list
# Wallets:
# [
#   "default *",
#   "second_wallet *"
# ]
```
## Importing Key to Wallet

Import the key that you want to use to sign the transaction to your wallet. The following will import key for genesis accounts.
```bash
export initaPrivKey=5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3 && /
export initaPublKey=EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV && /
./eosc wallet import $initaPrivKey
# imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```
You will then be able to see the list of imported private keys and their respective public key:
```bash
./eosc wallet keys
# [[
#     "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
#     "5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"
#   ]
# ]
```
## Locking and Unlocking Wallet

To keep your private key safe, lock your wallet
```bash
./eosc wallet lock -n second_wallet
# Locked: 'second_wallet'
```
Notice that the locked wallet doesn't have * symbol in the list
```bash
./eosc wallet list
# Wallets:
# [
#  "default *",
#  "second_wallet"
#]
```
To unlock it specify the password you get when creating the wallet
```bash
./eosc wallet unlock -n second_wallet --password $second_walletWalletPswd
# Unlocked: 'second-wallet'
```

## Creating an Account

```bash
. ./eoscCreateKey owner
# $ownerPubl=5K4hEQuPvt46zY8KCb9VgD7au9ShvKsC8T4YgNQ5EWhtthoXHKK
# $ownerPriv=EOS6Wagm98tHbogzcRPukEJhx7gVM579zyN85prxF46FnxAZFZcoQ

. ./eoscCreateKey active
# $activePubl=5KCiDbGgdMgwXj6TY9CD1u657bpmyKugDXRDqX1ezNS59APV4hE
# $activePriv=EOS5V72zR8MX6AowFv95CPSCc2g6Q4TF12wQ5hmtpSjmHJWdT5y9i
```

Then create an account called tester:
```bash
./eosc wallet unlock --password $defaultWalletPswd && \
./eosc create account inita tester $ownerPubl $activePubl
```

## All together

```bash
export initaPrivKey=5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3 && \
export initaPublKey=EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV && \
cd $EOS_PROGRAMS/eosc && \

. ./eoscWalletCreate && \ # -> $defaultWalletPswd
./eosc wallet import $initaPrivKey && \

. ./eoscCreateKey owner && \ # -> $ownerPublKey $ownerPrivKey
. ./eoscCreateKey active && \ # -> $activePublKey $activePrivKey

./eosc wallet unlock --password $defaultWalletPswd && \
./eosc create account inita tester  $activePubl
```














EOS comes with example contracts that can be uploaded and run for testing purposes. We demonstrate how to upload and interact with the sample contract "currency". 

* Let the EOS node be running:
```bash
cd ${EOS_PROGRAMS}/eosd/ && ./eosd
```

* Set up a wallet and importing account key. As the `config.ini` has an entry reading `plugin = eos::wallet_api_plugin`, EOS wallet is running as a part of `eosd` process. Every contract requires an associated account, so create a wallet. </br> 
Start a new bash (Ctr + Shift + \`), and input there:

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc wallet create 
# response is
# Creating wallet: default
# Save password to use in the future to unlock this wallet.
# Without password imported keys will not be retrievable.
# "PW5Jt75WX3P82TjCLtgKaR1js9yxHxmAwXoc8c2jZBQKeMvaqNJX1"
```

* For the purpose of this walkthrough, import the private key of the `inita` account, a test account included within genesis.json, so that you're able to issue API commands under authority of an existing account. The private key referenced below is found within your `config.ini` and is provided to you for testing purposes. 

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc wallet import \
5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
# response is
# imported private key for: 
# EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

* Create accounts for sample "currency" contract. Generate some public/private key pairs that will be later assigned as `owner_key` and `active_key`.

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc create key # owner_key
# response is
# Private key: 5KKLojCoiy9TFDuM6oZ4Qeymkkwh8E2GV8rYMtg3qbkBLQKNQ6m
# Public key:  EOS6EPtUUWh8X9vaaVj8GrLPuN1G3NmyXbT9PzyaEqz2eWXRZKjDM

cd ${EOS_PROGRAMS}/eosc/ && ./eosc create key # active_key
# response is
# Private key: 5KXrX8VkVqjvN3zxMma1RqDuSk6ojZd9tpkjf6gLnvTWu11mWyE
# Public key:  EOS6MEWsw4HWK5B7ThU8k9S1yAeRG2rRxNMLZZDczyNAxEuiLX1Li
```

* Run the `create` command where `inita` is the account authorizing the creation of the `currency` account and arguments are the public owner_key and the public active_key. </br>
```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc create account inita currency \
EOS6JWgw2m6CYuqNH9dBjxknJr4j7A4qEyX42vwbAstMxCsrkjg91 \
EOS5s1s1cQhdv6BLhfdj4XVy96pgYyQCpNJZJkJitmpf4QPjKQaTP 
# response is
```  
```json
{
  "transaction_id": "3c5c7bb041024a12afe43a57aab30845f433818a09c7ad47aed4ccf9a5aa0097",
  "processed": {
    "refBlockNum": 161,
    "refBlockPrefix": 3590202471,
    "expiration": "2017-09-25T13:10:51",
    "scope": [
      "eos",
      "inita"
    ],
    "signatures": [
      "20271a8d67fd31427338121cda887b2e58ac1c45c2d8ae73a49664f23d76a8556214558e1b351004612edde1be4fd990f9ee147f00e0cb0498501fa76476ee630c"
    ],
    "messages": [{
        "code": "eos",
```


* Go ahead and check that the account was successfully created

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc get account currency
# response is
```
```json
{
  "name": "currency",
  "eos_balance": 0,
  "staked_balance": 1,
  "unstaking_balance": 0,
  "last_unstaking_time": "2106-02-07T06:28:15"
}
```

* Now import the private active_key generated previously in the wallet:

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc wallet import \
5KXrX8VkVqjvN3zxMma1RqDuSk6ojZd9tpkjf6gLnvTWu11mWyE
```

* Before uploading a contract to blockchain, verify that there is no current contract:

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc get code currency 
# response is
# code hash: 0000000000000000000000000000000000000000000000000000000000000000
```

* With an account for a contract created, upload a sample contract:

```bash
cd ${EOS_PROGRAMS}/eosc/ && ./eosc set contract currency \
../../contracts/currency/currency.wast \
../../contracts/currency/currency.abi
```

As a response you should get a json with a `transaction_id` field. Your contract was successfully uploaded!

You can also verify that the code has been set with the following command

```bash
./eosc get code currency
```

It will return something like
```bash
code hash: 9b9db1a7940503a88535517049e64467a6e8f4e9e03af15e9968ec89dd794975
```

Next verify the currency contract has the proper initial balance:

```bash
./eosc get table currency currency account
{
  "rows": [{
     "account": "account",
     "balance": 1000000000
     }
  ],
  "more": false
}
```

<a name="pushamessage"></a>
### Transfering funds with the sample "currency" contract 

Anyone can send any message to any contract at any time, but the contracts may reject messages which are not given necessary permission. Messages are not
sent "from" anyone, they are sent "with permission of" one or more accounts and permission levels. The following commands shows a "transfer" message being
sent to the "currency" contact.  

The content of the message is `'{"from":"currency","to":"inita","amount":50}'`. In this case we are asking the currency contract to transfer funds from itself to
someone else.  This requires the permission of the currency contract.


```bash
./eosc push message currency transfer '{"from":"currency","to":"inita","amount":50}' --scope currency,inita --permission currency@active
```

Below is a generalization that shows the `currency` account is only referenced once, to specify which contact to deliver the `transfer` message to.

```bash
./eosc push message currency transfer '{"from":"${usera}","to":"${userb}","amount":50}' --scope ${usera},${userb} --permission ${usera}@active
```

We specify the `--scope ...` argument to give the currency contract read/write permission to those users so it can modify their balances.  In a future release scope
will be determined automatically.

As a confirmation of a successfully submitted transaction you will receive json output that includes a `transaction_id` field.

<a name="readingcontract"></a>
### Reading sample "currency" contract balance

So now check the state of both of the accounts involved in the previous transaction. 

```bash
./eosc get table inita currency account
{
  "rows": [{
      "account": "account",
      "balance": 50 
       }
    ],
  "more": false
}
./eosc get table currency currency account
{
  "rows": [{
      "account": "account",
      "balance": 999999950
    }
  ],
  "more": false
}
```

As expected, the receiving account **inita** now has a balance of **50** tokens, and the sending account now has **50** less tokens than its initial supply. 
