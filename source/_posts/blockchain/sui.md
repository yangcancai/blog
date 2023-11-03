---
title: sui
date: 2023-03-25 16:19:10
tags: sui
categories:
- sui
- blockchain
---

## Sui
0.000000001SUI=1MIST
1SUI=1000000000MIST

### 创建localnet
```shell
$ sui genesis --force
Config file ["/Users/cam/.sui/sui_config/client.yaml"] doesn't exist, do you want to connect to a Sui full node server [yN]?y
Sui full node server url (Default to Sui DevNet if not specified) :
Select key scheme to generate keypair (0 for ed25519, 1 for secp256k1, 2: for secp256r1:
1
Generated new keypair for address with scheme "secp256k1" [0x0243a9623b99552b35e2a6aea3ec0a40f607a04f]
Secret Recovery Phrase : [another truly verb share love sheriff portion west jar achieve ice accuse mimic pumpkin window key talk pet quiz kid swamp stay duty arrest]
Created new keypair for address with scheme ED25519: [0x2617955e0a5c55514f03290f3f89a3202f0808c6]
```
### 启动本地网络
```shell
$ sui start
```
### 切换网络到localnet
```shell
$ sui client envs
devnet => https://fullnode.devnet.sui.io:443 (active)
localnet => http://0.0.0.0:9000
$ sui client switch --env localnet
Active environment switched to [localnet]
$ sui client envs
devnet => https://fullnode.devnet.sui.io:443
localnet => http://0.0.0.0:9000 (active)
```
### 查看当前active-address下的资源
```shell
$ sui console
   _____       _    ______                       __
  / ___/__  __(_)  / ____/___  ____  _________  / /__
  \__ \/ / / / /  / /   / __ \/ __ \/ ___/ __ \/ / _ \
 ___/ / /_/ / /  / /___/ /_/ / / / (__  ) /_/ / /  __/
/____/\__,_/_/   \____/\____/_/ /_/____/\____/_/\___/
--- Sui Console 0.27.1 ---

Managed addresses : 3
Active address: 0x0243a9623b99552b35e2a6aea3ec0a40f607a04f
Keystore Type : File
Keystore Path : Some("/Users/cam/.sui/sui_config/sui.keystore")
Active environment : localnet
RPC URL: http://0.0.0.0:9000
Connecting to Sui full node. API version 0.27.1

Available RPC methods: ["sui_batchTransaction", "sui_devInspectTransaction", "sui_dryRunTransaction", "sui_executeTransaction", "sui_executeTransactionSerializedSig", "sui_getAllBalances", "sui_getAllCoins", "sui_getBalance", "sui_getCheckpointContents", "sui_getCheckpointContentsByDigest", "sui_getCheckpointSummary", "sui_getCheckpointSummaryByDigest", "sui_getCoinMetadata", "sui_getCoins", "sui_getCommitteeInfo", "sui_getDelegatedStakes", "sui_getDynamicFieldObject", "sui_getDynamicFields", "sui_getEvents", "sui_getLatestCheckpointSequenceNumber", "sui_getMoveFunctionArgTypes", "sui_getNormalizedMoveFunction", "sui_getNormalizedMoveModule", "sui_getNormalizedMoveModulesByPackage", "sui_getNormalizedMoveStruct", "sui_getObject", "sui_getObjectsOwnedByAddress", "sui_getRawObject", "sui_getReferenceGasPrice", "sui_getSuiSystemState", "sui_getTotalSupply", "sui_getTotalTransactionNumber", "sui_getTransaction", "sui_getTransactionAuthSigners", "sui_getTransactions", "sui_getTransactionsInRange", "sui_getValidators", "sui_mergeCoins", "sui_moveCall", "sui_pay", "sui_payAllSui", "sui_paySui", "sui_publish", "sui_requestAddDelegation", "sui_requestSwitchDelegation", "sui_requestWithdrawDelegation", "sui_splitCoin", "sui_splitCoinEqual", "sui_submitTransaction", "sui_subscribeEvent", "sui_tblsSignRandomnessObject", "sui_transferObject", "sui_transferSui", "sui_tryGetPastObject"]

Welcome to the Sui interactive console.
sui>-$ addresses
Showing 3 results.
0x0243a9623b99552b35e2a6aea3ec0a40f607a04f <=
0x2617955e0a5c55514f03290f3f89a3202f0808c6
0xe05563e3a9a5b39e8a54ea52d418cfd66ef8a192
sui>-$ active-address
0x0243a9623b99552b35e2a6aea3ec0a40f607a04f
sui>-$ objects
                 Object ID                  |  Version   |                    Digest                    |   Owner Type    |               Object Type
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 0x93d1dbb27ab540c598c876b34664bbbb78fbb786 |     12     | qmNQNSTFigKCBVFu6ODtBLPy+SE7VbxI2Dpw1fdKOK4= |  AddressOwner   |      0x2::coin::Coin<0x2::sui::SUI>
 0xa592e00ea4d5f75c365f72943f94cf3cc2abc76c |     1      | dBjv77qYrMEN4QQNSuW1zoCfGKxvWoYfKbvCKGZoVz8= |  AddressOwner   |      0x2::coin::Coin<0x2::sui::SUI>
 0xf059ffde87d73e27ce986ec9e198d42d4a94af31 |     1      | IAw5n3NbZmZEs9zCgN96idP6q4KHwXLIRO9v1yQtnsw= |  AddressOwner   |      0x2::coin::Coin<0x2::sui::SUI>
 0xf29fb02b03db7080b156168cdb947716e9dfc27d |     1      | 20uik0+hVu6p5Y2zQjA//2jE7HKkZFXPVW6/AWAtfgQ= |  AddressOwner   |      0x2::coin::Coin<0x2::sui::SUI>
Showing 4 results.
sui>-$ gas
                 Object ID                  |  Gas Value
----------------------------------------------------------------------
 0x93d1dbb27ab540c598c876b34664bbbb78fbb786 | 99999999999843
 0xa592e00ea4d5f75c365f72943f94cf3cc2abc76c | 100000000000000
 0xf059ffde87d73e27ce986ec9e198d42d4a94af31 | 100000000000000
 0xf29fb02b03db7080b156168cdb947716e9dfc27d | 100000000000000
```
### gas 费用计算
![gas费用](gas_fee.png)

### 转移对象
```shell
sui client transfer [OPTIONS] --to <TO> --object-id <OBJECT_ID> --gas-budget <GAS_BUDGET>

OPTIONS:
        --object-id <OBJECT_ID> 
            Object to transfer, in 20 bytes Hex string(当前address下的object-id)

        --gas <GAS>
            ID of the gas object for gas payment, in 20 bytes Hex string If not provided, a gas
            object with at least gas_budget value will be selected(支付gas费用的object id)

        --gas-budget <GAS_BUDGET>
            Gas budget for this transfer(gas预算不能太小也不能太高,太低或者太高事务失败也需要支付费用)

    -h, --help
            Print help information

        --json
            Return command outputs in json format

        --to <TO>
            Recipient address(接收对象的目标地址)
```

```shell
sui client transfer --to 0x2617955e0a5c55514f03290f3f89a3202f0808c6 --object-id 0x01a9f566b52365bd10e13232479ccdc495096e3a --gas-budget 173 --gas 0x97f10908ccca4bc20
```