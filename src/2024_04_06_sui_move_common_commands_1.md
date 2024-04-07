# SUI Move常用命令——网络交互命令（client）

*rzexin 2024.04.06*

## 1 前言

- **本文使用的`sui`版本**

```bash
$ sui --version
sui 1.22.0-0362997459
```

- **`client`的子命令很丰富，以下将介绍其中部分常用命令**

可通过`help/-h`获取完整的子命令：

```bash
$ sui client -h
Client for interacting with the Sui network

Usage: sui client [OPTIONS] [COMMAND]
```

## 2 链网络

### （1）envs：查看所有链网络环境

> List all Sui environments
>
> 注：本地文件路径：`.sui/sui_config/client.yaml`

```bash
$ sui client envs 
╭──────────┬───────────────────────────────────────┬────────╮
│ alias    │ url                                   │ active │
├──────────┼───────────────────────────────────────┼────────┤
│ devnet   │ https://fullnode.devnet.sui.io:443    │        │
│ mainnet  │ https://sui-mainnet.nodeinfra.com:443 │ *      │
│ testnet  │ https://fullnode.testnet.sui.io:443   │        │
╰──────────┴───────────────────────────────────────┴────────╯
```

### （2）active-env：获取当前活跃环境

> Default environment used for commands when none specified

```bash
$ sui client active-env
devnet
```

### （3）new-env：添加链网络环境

> Add new Sui environment

```bash
$ sui client new-env --alias js --rpc http://127.0.0.1:9000
Added new Sui env [js] to config.
```

### （4）switch：切换网络

> Switch active address and network(e.g., devnet, local rpc server)
>
> 注：通过`switch`命令可以同时切换地址和网络。

```bash
$ sui client switch --env js
Active environment switched to [js]
```

## 3 地址

### （1）addresses：获取当前客户端管理的所有地址

> Obtain the Addresses managed by the client
>
> 注：本地文件`.sui/sui_config/sui.aliases`中存储的是地址`base64`编码的公钥，在文件`.sui/sui_config/sui.keystore`存储的是地址私钥

```bash
$ sui client addresses --json
{
  "activeAddress": "0x5c5882d73a6e5b6ea1743fb028eff5e0d7cc8b7ae123d27856c5fe666d91569a",
  "addresses": [
    [
      "jason",
      "0x5c5882d73a6e5b6ea1743fb028eff5e0d7cc8b7ae123d27856c5fe666d91569a"
    ],
    [
      "alice",
      "0x2d178b9704706393d2630fe6cf9415c2c50b181e9e3c7a977237bb2929f82d19"
    ],
    [
      "bob",
      "0xf2e6ffef7d0543e258d4c47a53d6fa9872de4630cc186950accbd83415b009f0"
    ]
  ]
}
```

### （2）active-address：获取当前活跃地址

> Default address used for commands when none specified

```bash
$ sui client active-address 
0x5c5882d73a6e5b6ea1743fb028eff5e0d7cc8b7ae123d27856c5fe666d91569a
```

### （3）new-address：创建新地址

> Generate new address and keypair with keypair scheme flag {`ed25519 | secp256k1 | secp256r1`} with optional derivation path, default to `m/44'/784'/0'/0'/0' for ed25519` or `m/54'/784'/0'/0/0 for secp256k1` or `m/74'/784'/0'/0/0 for secp256r1`. Word length can be { word12 | word15 | word18 | word21 | word24} default to word12 if not specified

```bash
# Usage: sui client new-address [OPTIONS] <KEY_SCHEME> [ALIAS] [WORD_LENGTH] [DERIVATION_PATH]

$ sui client new-address ed25519 js --json
Keys saved as Base64 with 33 bytes `flag || privkey` ($BASE64_STR). 
        To see Bech32 format encoding, use `sui keytool export $SUI_ADDRESS` where 
        $SUI_ADDRESS can be found with `sui keytool list`. Or use `sui keytool convert $BASE64_STR`.
{
  "alias": "js",
  "address": "0xe760ab0e273b4ec996abe76ce67e895e13064611cb4215d7358f2a63188e1a98",
  "keyScheme": "ED25519",
  "recoveryPhrase": "dismiss split donor onion hour heavy stand work detail issue room pink lion enroll"
}
```

### （4）switch：切换地址

```bash
$ sui client switch --address js
Active address switched to 0xe760ab0e273b4ec996abe76ce67e895e13064611cb4215d7358f2a63188e1a98
```

## 4 对象

### （1）objects：获取地址拥有的所有对象

> Obtain all objects owned by the address. It also accepts an address by its alias

```bash
# Usage: sui client objects [OPTIONS] [owner_address]

$ sui client objects --json
[
  {
    "data": {
      "objectId": "0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76",
      "version": "25131978",
      "digest": "33Ny3AZ3q169QD2uvrso6QtfigyFqN8DccZ1cUJR7v2y",
      "type": "0x99bb55e386947db154dcb23b8f39191536c6332b659f3c6aa7596a0cd1f5c4bc::LuckyCafe::Card",
      "owner": {
        "AddressOwner": "0x5c5882d73a6e5b6ea1743fb028eff5e0d7cc8b7ae123d27856c5fe666d91569a"
      },
      "previousTransaction": "4qYbEkvQWWshcgyg93NVtHt4hAMqnU9CMt86HJentZ4B",
      "storageRebate": "1535200",
      "content": {
        "dataType": "moveObject",
        "type": "0x99bb55e386947db154dcb23b8f39191536c6332b659f3c6aa7596a0cd1f5c4bc::LuckyCafe::Card",
        "hasPublicTransfer": true,
        "fields": {
          "cafe_id": "0x0463f197aa3c90a6c668089267aef6cff0ae3a50ed8144a7da84a49ece45f2e1",
          "id": {
            "id": "0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76"
          }
        }
      }
    }
  },
......
```

### （2）object：获取对象信息

> Get object info

```json
$ sui client object 0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76 --json
{
  "objectId": "0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76",
  "version": "25131978",
  "digest": "33Ny3AZ3q169QD2uvrso6QtfigyFqN8DccZ1cUJR7v2y",
  "type": "0x99bb55e386947db154dcb23b8f39191536c6332b659f3c6aa7596a0cd1f5c4bc::LuckyCafe::Card",
  "owner": {
    "AddressOwner": "0x5c5882d73a6e5b6ea1743fb028eff5e0d7cc8b7ae123d27856c5fe666d91569a"
  },
  "previousTransaction": "4qYbEkvQWWshcgyg93NVtHt4hAMqnU9CMt86HJentZ4B",
  "storageRebate": "1535200",
  "content": {
    "dataType": "moveObject",
    "type": "0x99bb55e386947db154dcb23b8f39191536c6332b659f3c6aa7596a0cd1f5c4bc::LuckyCafe::Card",
    "hasPublicTransfer": true,
    "fields": {
      "cafe_id": "0x0463f197aa3c90a6c668089267aef6cff0ae3a50ed8144a7da84a49ece45f2e1",
      "id": {
        "id": "0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76"
      }
    }
  }
}
```

### （3）transfer：转移对象

> Transfer object

```bash
$ export CARD=0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76

$ sui client transfer --to alice --object-id $CARD --gas-budget 10000000

# 可见AddressOwner已经进行了转移
$ sui client object 0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76 --json
{
  "objectId": "0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76",
  "version": "28078186",
  "digest": "2y7xvkF9fKPM2jdmxrwShtFsJXQhH7C17KLXBu2zepPN",
  "type": "0x99bb55e386947db154dcb23b8f39191536c6332b659f3c6aa7596a0cd1f5c4bc::LuckyCafe::Card",
  "owner": {
    "AddressOwner": "0x2d178b9704706393d2630fe6cf9415c2c50b181e9e3c7a977237bb2929f82d19"
  },
  "previousTransaction": "CUoG2NZT5L94fHqWZNTrAoUs88HDAhyXiSLxL3m9Zn6U",
  "storageRebate": "1535200",
  "content": {
    "dataType": "moveObject",
    "type": "0x99bb55e386947db154dcb23b8f39191536c6332b659f3c6aa7596a0cd1f5c4bc::LuckyCafe::Card",
    "hasPublicTransfer": true,
    "fields": {
      "cafe_id": "0x0463f197aa3c90a6c668089267aef6cff0ae3a50ed8144a7da84a49ece45f2e1",
      "id": {
        "id": "0xcc0b1b68febda7888b7681a70199ab291b38941ee965ed27af1b0bdf8ae79f76"
      }
    }
  }
}
```

## 5 COIN

### （1）balance：获取地址代币余额

> List the coin balance of an address
>
> ```json
> Usage: sui client balance [OPTIONS] [ADDRESS]
> ```

- **获取地址下所有代币余额**

```bash
$ sui client balance 
╭───────────────────────────────────────────────╮
│ Balance of coins owned by this address        │
├───────────────────────────────────────────────┤
│ ╭───────────────────────────────────────────╮ │
│ │ coin      balance (raw)  balance          │ │
│ ├───────────────────────────────────────────┤ │
│ │ Sui       3123666992     3.12 SUI         │ │
│ │ RZX-name  10000          100.00 RZX-sym   │ │
│ ╰───────────────────────────────────────────╯ │
╰───────────────────────────────────────────────╯
```

- **获取指定代币余额**

```bash
# --coin-type <COIN_TYPE>  Show balance for the specified coin (e.g., 0x2::sui::SUI). All coins will be shown if none is passed

$ sui client balance --coin-type 0x2::sui::SUI
╭────────────────────────────────────────╮
│ Balance of coins owned by this address │
├────────────────────────────────────────┤
│ ╭─────────────────────────────────╮    │
│ │ coin  balance (raw)  balance    │    │
│ ├─────────────────────────────────┤    │
│ │ Sui   3123666992     3.12 SUI   │    │
│ ╰─────────────────────────────────╯    │
╰────────────────────────────────────────╯
```

- **获取每个代币对象ID和余额**

```bash
# --with-coins     Show a list with each coin's object ID and balance

$ sui client balance --with-coins  
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance of coins owned by this address                                                                  │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ╭───────────────────────────────────────────────────────────────────────────────────────────────╮       │
│ │ Sui: 3 coins, Balance: 3123666992 (3.12 SUI SUI)                                              │       │
│ ├───────────────────────────────────────────────────────────────────────────────────────────────┤       │
│ │ coinId                                                              balance (raw)  balance    │       │
│ ├───────────────────────────────────────────────────────────────────────────────────────────────┤       │
│ │ 0x043286ba5f16a930c04014e6a8790329a36f01103768055d9daace7cb2466cba  1123666992     1.12 SUI   │       │
│ │ 0x515a581d8f448102d45f304bbac4c7eea09d043ffa7bdf4089e9bf277b02a7bd  1000000000     1.00 SUI   │       │
│ │ 0x83d01a9c0b08abea7de5028a2bdbf55255ee6892b9d408b71209d16f1553e5e6  1000000000     1.00 SUI   │       │
│ ╰───────────────────────────────────────────────────────────────────────────────────────────────╯       │
│ ╭─────────────────────────────────────────────────────────────────────────────────────────────────────╮ │
│ │ RZX-name: 1 coin, Balance: 10000 (100.00 RZX-sym RZX-sym)                                           │ │
│ ├─────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ coinId                                                              balance (raw)  balance          │ │
│ ├─────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ 0x79985fae7d22dffac0a1f31e8bc90d52440af2a06242de55f99fe646df808ff7  10000          100.00 RZX-sym   │ │
│ ╰─────────────────────────────────────────────────────────────────────────────────────────────────────╯ │
│                                                                                                         │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### （2）faucet：领水

> Request gas coin from faucet. By default, it will use the active address and the active network
>
> 注：执行该命令可以直接领水，非常方便。

```bash
$ sui client faucet 
Request successful. It can take up to 1 minute to get the coin. Run sui client gas to check your gas coins.
```

### （3）merge-coin：代币合并

> Merge two coin objects into one coin

```bash
$ export COIN1=0xff1e2cee6496506ddd70ac3d4815e664ee104aba415f49d6b1deb05d1a1f3ce0
$ export COIN2=0xf50c3a6b073e15fb36c32166922c21d74bd4c7b1461561cd062e1802d9e965a3

$ sui client merge-coin --primary-coin $COIN1 --coin-to-merge $COIN2 --gas-budget 100000000

$ sui client gas
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x043286ba5f16a930c04014e6a8790329a36f01103768055d9daace7cb2466cba │ 1123625352         │ 1.12             │
......
│ 0xff1e2cee6496506ddd70ac3d4815e664ee104aba415f49d6b1deb05d1a1f3ce0 │ 2000000000         │ 2.00             │
╰────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────╯
```

### （4）split-coin：代币拆分

> Split a coin object into multiple coins
>
> 注：以下两种拆分方式不能混用！`error: the argument '--amounts <AMOUNTS>...' cannot be used with '--count <COUNT>'`

- **按指定数量拆分**

```bash
$ export COIN=0xe35bc45117fc7abddf4114c8e052544c3b1fc2d44103dd91ec32f1928788bce6
$ sui client split-coin --coin-id $COIN --amounts 1000 2000 --gas-budget 100000000

$ sui client gas
╭────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Showing 12 gas coins and their balances.                                                                   │
├────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────┤
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x043286ba5f16a930c04014e6a8790329a36f01103768055d9daace7cb2466cba │ 1120629592         │ 1.12             │
.....
│ 0xe35bc45117fc7abddf4114c8e052544c3b1fc2d44103dd91ec32f1928788bce6 │ 999997000          │ 0.99             │
│ 0xcb0e6ee241024eaefb8a9363e02cf98aa2a7a56738e6da648c66d0c699a1d3a6 │ 2000               │ 0.00             │
│ 0xe8d7bf9b843f4f4adc5962976f3e9afa677f431f9752ac4c3849b86a7798b20c │ 1000               │ 0.00             │
├────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────┤
│ Showing 12 gas coins and their balances.                                                                   │
╰────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

- **按份数拆分**

```bash
$ export COIN=0xe8d7bf9b843f4f4adc5962976f3e9afa677f431f9752ac4c3849b86a7798b20c
$ sui client split-coin --coin-id $COIN --count 2 --gas-budget 100000000

$ sui client gas
╭────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Showing 14 gas coins and their balances.                                                                   │
├────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────┤
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x043286ba5f16a930c04014e6a8790329a36f01103768055d9daace7cb2466cba │ 1116614072         │ 1.11             │
......
│ 0xe8d7bf9b843f4f4adc5962976f3e9afa677f431f9752ac4c3849b86a7798b20c │ 500                │ 0.00             │
│ 0x3ff13ef4795f4b7fd11205a1e80565b6e2ebd504c945a210fb789bb11dd53528 │ 500                │ 0.00             │
├────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────┤
│ Showing 14 gas coins and their balances.                                                                   │
╰────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### （5）transfer-sui：转移SUI

> Transfer SUI, and pay gas with the same SUI coin object. If amount is specified, only the amount is transferred; otherwise the entire object is transferred
>
> ```bash
> Usage: sui client transfer-sui [OPTIONS] --to <TO> --sui-coin-object-id <SUI_COIN_OBJECT_ID> --gas-budget <GAS_BUDGET>
> 
> Options:
>       --to <TO>                                  Recipient address (or its alias if it's an address in the keystore)
>       --sui-coin-object-id <SUI_COIN_OBJECT_ID>  Sui coin object to transfer, ID in 20 bytes Hex string. This is also the gas object
>       --gas-budget <GAS_BUDGET>                  Gas budget for this transfer
>       --amount <AMOUNT>                          The amount to transfer, if not specified, the entire coin object will be transferred
> ```

- **转移指定数量**

```bash
$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x05b1b5c60d1b777a5dfc7d5a32874a563f303b1902c0fcd7ea7ab2bd6c0dc29e │ 996016240          │ 0.99             │
│ 0x23accfba5fd1743b042c7c0a1596cf346b986c3f1844585ab1c8058baeca00a8 │ 250000000          │ 0.25             │
│ 0x2fcab6efdd3c96746d3e0bc2dc27995b0c68797bf439fc797b2c96181ab426e3 │ 250000000          │ 0.25             │
│ 0x7d233de155d5a58855447b3ae165e9ec605109861ce14f07f63bd83eb01a4856 │ 250000000          │ 0.25             │
│ 0xd2e2c66fa2d0b0ee4316987a7b1d9e004ce56ef21ee92ba28f8c094097d1bbbf │ 250000000          │ 0.25             │
╰────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────╯

$ sui client gas bob
No gas coins are owned by this address

$ export COIN=0xd2e2c66fa2d0b0ee4316987a7b1d9e004ce56ef21ee92ba28f8c094097d1bbbf

$ sui client  transfer-sui --to bob --sui-coin-object-id $COIN --amount 50000000 --gas-budget 100000000

$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x05b1b5c60d1b777a5dfc7d5a32874a563f303b1902c0fcd7ea7ab2bd6c0dc29e │ 996016240          │ 0.99             │
│ 0x23accfba5fd1743b042c7c0a1596cf346b986c3f1844585ab1c8058baeca00a8 │ 250000000          │ 0.25             │
│ 0x2fcab6efdd3c96746d3e0bc2dc27995b0c68797bf439fc797b2c96181ab426e3 │ 250000000          │ 0.25             │
│ 0x7d233de155d5a58855447b3ae165e9ec605109861ce14f07f63bd83eb01a4856 │ 250000000          │ 0.25             │
│ 0xd2e2c66fa2d0b0ee4316987a7b1d9e004ce56ef21ee92ba28f8c094097d1bbbf │ 198002120          │ 0.19             │
╰────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────╯

$ sui client gas bob
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x724b02db0f8e50aced87f8871d808a4eca06e132345faa57658cd4d1fd12d655 │ 50000000           │ 0.05             │
╰────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────╯
```

- **全部转移**

```bash
$ export COIN=0x7d233de155d5a58855447b3ae165e9ec605109861ce14f07f63bd83eb01a4856

$ sui client  transfer-sui --to bob --sui-coin-object-id $COIN --gas-budget 10000000

$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x05b1b5c60d1b777a5dfc7d5a32874a563f303b1902c0fcd7ea7ab2bd6c0dc29e │ 996016240          │ 0.99             │
│ 0x23accfba5fd1743b042c7c0a1596cf346b986c3f1844585ab1c8058baeca00a8 │ 250000000          │ 0.25             │
│ 0x2fcab6efdd3c96746d3e0bc2dc27995b0c68797bf439fc797b2c96181ab426e3 │ 250000000          │ 0.25             │
│ 0xd2e2c66fa2d0b0ee4316987a7b1d9e004ce56ef21ee92ba28f8c094097d1bbbf │ 198002120          │ 0.19             │
╰────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────╯

$ sui client gas bob
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x724b02db0f8e50aced87f8871d808a4eca06e132345faa57658cd4d1fd12d655 │ 50000000           │ 0.05             │
│ 0x7d233de155d5a58855447b3ae165e9ec605109861ce14f07f63bd83eb01a4856 │ 248990120          │ 0.24             │
╰────────────────────────────────────────────────────────────────────┴────────────────────┴──────────────────╯
```

### （6）pay-sui：支付SUI

> Pay SUI coins to recipients following following specified amounts, with input coins. 
>
> Length of recipients must be the same as that of amounts. 
>
> The input coins also include the coin for gas payment, so no extra gas coin is required
>
> 注：`pay-sui`命令与`transfer-sui`命令相似，差异是支持了批量转账，且可以支持多个`coin`会进行合并后，再转币。
>
> ```bash
> Usage: sui client pay-sui [OPTIONS] --gas-budget <GAS_BUDGET>
> 
> Options:
>       --input-coins <INPUT_COINS>...    The input coins to be used for pay recipients, including the gas coin
>       --recipients <RECIPIENTS>...      The recipient addresses, must be of same length as amounts. Aliases of addresses are also accepted as input
>       --amounts <AMOUNTS>...            The amounts to be paid, following the order of recipients
> ```

- **单地址转**

```bash
$ export COIN_ID=0x241d78606cb3b7d2f880854c49781e7c537d318a52c499faef24e101455adf9a

$ sui client pay-sui --input-coins $COIN_ID --recipients alice --amounts 10000 --gas-budget 1000000000

$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0x73166aad134319e9c26b19ec6ed0ce419812a7e2612c9ea0e1b93767de423c4d │ 10000      │
╰────────────────────────────────────────────────────────────────────┴────────────╯

$ sui client gas jason
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0x241d78606cb3b7d2f880854c49781e7c537d318a52c499faef24e101455adf9a │ 3760487788 │
│ 0x70b96720fadb6aa45620ab84efd9139e4674057207c93e4375350ec695865fab │ 988221840  │
╰────────────────────────────────────────────────────────────────────┴────────────╯
```

- **多地址转**

```bash
$ sui client pay-sui --input-coins $COIN_ID  --recipients alice bob --amounts 10000 10000 --gas-budget 1000000000

$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0x73166aad134319e9c26b19ec6ed0ce419812a7e2612c9ea0e1b93767de423c4d │ 10000      │
│ 0xb6013d96900f73c43654a48d5c1aef39838513ec33a189593f962e31f5bc7898 │ 10000      │
╰────────────────────────────────────────────────────────────────────┴────────────╯

$ sui client gas bob
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0xedca429699b178481b88e39502ed39bd322e315c582a70520bd32930a25dd758 │ 10000      │
╰────────────────────────────────────────────────────────────────────┴────────────╯
```

- **合并后转**

> 若指定多个`coin`的话，会先将这些`coin`合并后，再进行转币。

```bash
$ export COIN_ID2=0x70b96720fadb6aa45620ab84efd9139e4674057207c93e4375350ec695865fab
$ sui client pay-sui --input-coins $COIN_ID $COIN_ID2 --recipients alice bob --amounts 20000 
80000 --gas-budget 1000000000

$ sui client gas jason
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0x241d78606cb3b7d2f880854c49781e7c537d318a52c499faef24e101455adf9a │ 4743595988 │
╰────────────────────────────────────────────────────────────────────┴────────────╯

$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0x73166aad134319e9c26b19ec6ed0ce419812a7e2612c9ea0e1b93767de423c4d │ 10000      │
│ 0xb6013d96900f73c43654a48d5c1aef39838513ec33a189593f962e31f5bc7898 │ 10000      │
│ 0xbfe36b88a30be6bf814c26f7aa4ebeedd4c4a3a189238ba8ce2e712fd5eecc31 │ 20000      │
╰────────────────────────────────────────────────────────────────────┴────────────╯

$ sui client gas bob
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0xac824c384f0bab18bfff13da09c8ba9e65607c6918076b0f90005b5034071f92 │ 80000      │
│ 0xedca429699b178481b88e39502ed39bd322e315c582a70520bd32930a25dd758 │ 10000      │
╰────────────────────────────────────────────────────────────────────┴────────────╯
```

### （7）pay-all-sui：转移所有代币

> Pay all residual SUI coins to the recipient with input coins, after deducting the gas cost. The input coins also include the coin for gas payment, so no extra gas coin is required

```bash
$ export COIN_ID=0x241d78606cb3b7d2f880854c49781e7c537d318a52c499faef24e101455adf9a

$ sui client pay-all-sui --input-coins $COIN_ID --recipient alice --gas-budget 10000000

$ sui client gas jason
No gas coins are owned by this address

$ sui client gas alice
╭────────────────────────────────────────────────────────────────────┬────────────╮
│ gasCoinId                                                          │ gasBalance │
├────────────────────────────────────────────────────────────────────┼────────────┤
│ 0x241d78606cb3b7d2f880854c49781e7c537d318a52c499faef24e101455adf9a │ 4742586108 │
│ 0x73166aad134319e9c26b19ec6ed0ce419812a7e2612c9ea0e1b93767de423c4d │ 10000      │
│ 0xb6013d96900f73c43654a48d5c1aef39838513ec33a189593f962e31f5bc7898 │ 10000      │
│ 0xbfe36b88a30be6bf814c26f7aa4ebeedd4c4a3a189238ba8ce2e712fd5eecc31 │ 20000      │
╰────────────────────────────────────────────────────────────────────┴────────────╯
```

## 6 合约

### （1）publish：部署合约

> Publish Move modules
>
> ```bash
> Usage: sui client publish [OPTIONS] --gas-budget <GAS_BUDGET> [package_path]
> ```

```bash
$ sui client publish --gas-budget 100000000
```

- 执行该命令会创建：`PackageID`，后续在调用合约方法或升级合约都会使用到。

- 如果在`init`合约构造方法中有创建`Object`，也会输出对应`ObjectId`

### （2）call：调用合约方法

> Call Move function
>
> 详细示例可以参看：[《SUI Move官方示例合约实践》](https://learnblockchain.cn/column/42)
>
> ```bash
> Usage: sui client call [OPTIONS] --package <PACKAGE> --module <MODULE> --function <FUNCTION> --gas-budget <GAS_BUDGET>
> 
> Options:
>       --package <PACKAGE>               Object ID of the package, which contains the module
>       --module <MODULE>                 The name of the module in the package
>       --function <FUNCTION>             Function name in module
>       --type-args <TYPE_ARGS>...        Type arguments to the generic function being called. All must be specified, or the call will fail
>       --args <ARGS>...                  Simplified ordered args like in the function syntax ObjectIDs, Addresses must be hex strings
> ```

```bash
$ client call --function claim_red_packet --package $PACKAGE_ID --module red_packet --type-args 0x2::sui::SUI --args $RED_POCKET $WEATHER_ORACLE 0x6 --gas-budget 10000000
```

### （3）upgrade：升级合约

> Upgrade Move modules
>
> 详细示例可参看：[《SUI Move开发必知必会——如何进行合约升级？》](https://bityoume.github.io/howtosuibook/2024_01_21_sui_move_how_to_upgrade_package.html)
>
> ```bash
> Usage: sui client upgrade [OPTIONS] --upgrade-capability <UPGRADE_CAPABILITY> --gas-budget <GAS_BUDGET> [package_path]
> ```

```bash
$ sui client upgrade --gas-budget 10000000 --upgrade-capability $UPGRADE_CAP_ID
```

## 7 更多

欢迎关注微信公众号：**Move中文**，开启你的 **Sui Move** 之旅！

![image-20240402223437320](assets/move_cn.png)