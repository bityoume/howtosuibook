# SUI CLI最全命令详解4——Keytool（下）@SUI Move开发必知必会

*rzexin 2024.05.26*

## 1 前言

`SUI`的命令行工具（`CLI`）的`keytool`命令提供了一些列的子命令，用于生成私钥、管理地址、签名验签、多签及`zkLogin`相关功能。

在[《SUI CLI最全命令详解3——Keytool（上）》](https://learnblockchain.cn/article/8203)已介绍了**密钥对类**和**单签类**，本篇将介绍**多签类**和**zkLogin类**。

## 2 命令分类

以下是本文使用的`SUI`版本及`keytool`的命令清单。

### 2.1 命令清单

```bash
$ sui --version
sui 1.27.0-b6887716b4da

$ sui keytool -h
Sui keystore tool

Usage: sui keytool [OPTIONS] <COMMAND>

Commands:
  update-alias                             Update an old alias to a new one. If a new alias is not provided, a random one will be generated
  convert                                  Convert private key in Hex or Base64 to new format (Bech32 encoded 33 byte flag || private key starting with "suiprivkey"). Hex private key
                                               format import and export are both deprecated in Sui Wallet and Sui CLI Keystore. Use `sui keytool import` if you wish to import a key to Sui
                                               Keystore
  decode-or-verify-tx                      Given a Base64 encoded transaction bytes, decode its components. If a signature is provided, verify the signature against the transaction and
                                               output the result
  decode-multi-sig                         Given a Base64 encoded MultiSig signature, decode its components. If tx_bytes is passed in, verify the multisig
  generate                                 Generate a new keypair with key scheme flag {ed25519 | secp256k1 | secp256r1} with optional derivation path, default to m/44'/784'/0'/0'/0' for
                                               ed25519 or m/54'/784'/0'/0/0 for secp256k1 or m/74'/784'/0'/0/0 for secp256r1. Word length can be { word12 | word15 | word18 | word21 | word24}
                                               default to word12 if not specified
  import                                   Add a new key to Sui CLI Keystore using either the input mnemonic phrase or a Bech32 encoded 33-byte `flag || privkey` starting with
                                               "suiprivkey", the key scheme flag {ed25519 | secp256k1 | secp256r1} and an optional derivation path, default to m/44'/784'/0'/0'/0' for ed25519
                                               or m/54'/784'/0'/0/0 for secp256k1 or m/74'/784'/0'/0/0 for secp256r1. Supports mnemonic phrase of word length 12, 15, 18, 21, 24. Set an alias
                                               for the key with the --alias flag. If no alias is provided, the tool will automatically generate one
  export                                   Output the private key of the given key identity in Sui CLI Keystore as Bech32 encoded string starting with `suiprivkey`
  list                                     List all keys by its Sui address, Base64 encoded public key, key scheme name in sui.keystore
  load-keypair                             This reads the content at the provided file path. The accepted format can be [enum SuiKeyPair] (Base64 encoded of 33-byte `flag || privkey`) or
                                               `type AuthorityKeyPair` (Base64 encoded `privkey`). This prints out the account keypair as Base64 encoded `flag || privkey`, the network keypair,
                                               worker keypair, protocol keypair as Base64 encoded `privkey`
  multi-sig-address                        To MultiSig Sui Address. Pass in a list of all public keys `flag || pk` in Base64. See `keytool list` for example public keys
  multi-sig-combine-partial-sig            Provides a list of participating signatures (`flag || sig || pk` encoded in Base64), threshold, a list of all public keys and a list of their
                                               weights that define the MultiSig address. Returns a valid MultiSig signature and its sender address. The result can be used as signature field
                                               for `sui client execute-signed-tx`. The sum of weights of all signatures must be >= the threshold
  multi-sig-combine-partial-sig-legacy     
  show                                     Read the content at the provided file path. The accepted format can be [enum SuiKeyPair] (Base64 encoded of 33-byte `flag || privkey`) or `type
                                               AuthorityKeyPair` (Base64 encoded `privkey`). It prints its Base64 encoded public key and the key scheme flag
  sign                                     Create signature using the private key for for the given address (or its alias) in sui keystore. Any signature commits to a [struct
                                               IntentMessage] consisting of the Base64 encoded of the BCS serialized transaction bytes itself and its intent. If intent is absent, default will
                                               be used
  sign-kms                                 Creates a signature by leveraging AWS KMS. Pass in a key-id to leverage Amazon KMS to sign a message and the base64 pubkey. Generate PubKey from
                                               pem using MystenLabs/base64pemkey Any signature commits to a [struct IntentMessage] consisting of the Base64 encoded of the BCS serialized
                                               transaction bytes itself and its intent. If intent is absent, default will be used
  unpack                                   This takes [enum SuiKeyPair] of Base64 encoded of 33-byte `flag || privkey`). It outputs the keypair into a file at the current directory where
                                               the address is the filename, and prints out its Sui address, Base64 encoded public key, the key scheme, and the key scheme flag
  zk-login-sign-and-execute-tx             Given the max_epoch, generate an OAuth url, ask user to paste the redirect with id_token, call salt server, then call the prover server, create a
                                               test transaction, use the ephemeral key to sign and execute it by assembling to a serialized zkLogin signature
  zk-login-enter-token                     A workaround to the above command because sometimes token pasting does not work (for Facebook). All the inputs required here are printed from the
                                               command above
  zk-login-sig-verify                      Given a zkLogin signature, parse it if valid. If `bytes` provided, parse it as either as TransactionData or PersonalMessage based on
                                               `intent_scope`. It verifies the zkLogin signature based its latest JWK fetched. Example request: sui keytool zk-login-sig-verify --sig
                                               $SERIALIZED_ZKLOGIN_SIG --bytes $BYTES --intent-scope 0 --network devnet --curr-epoch 10
  zk-login-insecure-sign-personal-message  TESTING ONLY: Generate a fixed ephemeral key and its JWT token with test issuer. Produce a zklogin signature for the given data and max epoch.
                                               e.g. sui keytool zk-login-insecure-sign-personal-message --data "hello" --max-epoch 5
  help                                     Print this message or the help of the given subcommand(s)
```

### 2.2 命令分类

以上命令，初步分为以下4类，[上篇](https://learnblockchain.cn/article/8203)已介绍了前两类，本篇将介绍后两类。

-   **密钥对类**
    -   `generate`
    -   `show`
    -   `unpack`
    -   `convert`
    -   `import`
    -   `export`
    -   `list`
    -   `update-alias`
    -   `load-keypair`
-   **单签类**
    -   `sign`
    -   `decode-or-verify-tx`
-   **多签类**
    -   `multi-sig-address`
    -   `multi-sig-combine-partial-sig`
-   **zkLogin类**
    -   `zk-login-enter-token`
    -   `zk-login-sign-and-execute-tx`
    -   `zk-login-sig-verify`

## 3 多签类

>   `SUI` 支持多重签名（`multisig`）交易，这需要多个密钥进行授权，而不是单密钥签名。
>
>   `SUI` 支持 `k` 对 `n` 多重签名交易，其中 `k` 是阈值，`n` 是所有参与方的总权重。最大参与方数量为 `10`。
>
>   多重签名的有效参与密钥是纯 `Ed25519`、`ECDSA Secp256k1` 和 `ECDSA Secp256r1`。
>
>   如果序列化的多重签名包含足够数量的有效签名，其权重之和超过阈值，`SUI` 将视为多重签名有效，并执行交易。

### 3.1 `multi-sig-address`：创建多重签名地址

#### （1）说明

```bash
To MultiSig Sui Address. Pass in a list of all public keys `flag || pk` in Base64. See `keytool list` for example public keys
```

#### （2）用法

```bash
Usage: sui keytool multi-sig-address [OPTIONS] --threshold <THRESHOLD>

Options:
      --threshold <THRESHOLD>  
      --json                   Return command outputs in json format
      --pks <PKS>...           
      --weights <WEIGHTS>... 
```

#### （3）使用

-   **创建3个不同密钥方案的地址**

```bash
$ sui client new-address ed25519 js1
╭──────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Created new keypair and saved it to keystore.                                                        │
├────────────────┬─────────────────────────────────────────────────────────────────────────────────────┤
│ alias          │ js1                                                                                 │
│ address        │ 0x1a142b16f802832d5ab2247ab89fffae686168088b16d93aabb2e53f1cdc99f0                  │
│ keyScheme      │ ed25519                                                                             │
│ recoveryPhrase │ wait virtual cushion tornado cover grain excuse warfare dwarf runway satisfy crater │
╰────────────────┴─────────────────────────────────────────────────────────────────────────────────────╯

$ sui client new-address secp256k1 js2
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Created new keypair and saved it to keystore.                                                     │
├────────────────┬──────────────────────────────────────────────────────────────────────────────────┤
│ alias          │ js2                                                                              │
│ address        │ 0x895d63a8db8bdb07bf450adcaa47fea03cfb1082f4c379b74493a1386b730d6f               │
│ keyScheme      │ secp256k1                                                                        │
│ recoveryPhrase │ disorder fiber mention december scrap wreck curtain option emotion any keen away │
╰────────────────┴──────────────────────────────────────────────────────────────────────────────────╯

$ sui client new-address secp256r1 js3
╭─────────────────────────────────────────────────────────────────────────────────────────╮
│ Created new keypair and saved it to keystore.                                           │
├────────────────┬────────────────────────────────────────────────────────────────────────┤
│ alias          │ js3                                                                    │
│ address        │ 0x60b8928150fe7a43bbb83dc156b315d3624dd3d2a4178026892e64fe1ab0fd43     │
│ keyScheme      │ secp256r1                                                              │
│ recoveryPhrase │ steak lake dune ski banner eternal debate pond figure tell leave faith │
╰────────────────┴────────────────────────────────────────────────────────────────────────╯
```

-   **查看本地keystore**

```bash
sui keytool list
╭────────────────────────────────────────────────────────────────────────────────────────────╮
│ ╭─────────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ alias           │  js1                                                                 │ │
│ │ suiAddress      │  0x1a142b16f802832d5ab2247ab89fffae686168088b16d93aabb2e53f1cdc99f0  │ │
│ │ publicBase64Key │  AOMfccBag19KFImrHhmJK+8e4W96vTHIqeuvojzQbU3h                        │ │
│ │ keyScheme       │  ed25519                                                             │ │
│ │ flag            │  0                                                                   │ │
│ │ peerId          │  e31f71c05a835f4a1489ab1e19892bef1ee16f7abd31c8a9ebafa23cd06d4de1    │ │
│ ╰─────────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ alias           │  js3                                                                 │ │
│ │ suiAddress      │  0x60b8928150fe7a43bbb83dc156b315d3624dd3d2a4178026892e64fe1ab0fd43  │ │
│ │ publicBase64Key │  AgI7OJmvbQSHY1GKBIEoA/kiZt/PtPJEBM+hBGVCOw0Vzw==                    │ │
│ │ keyScheme       │  secp256r1                                                           │ │
│ │ flag            │  2                                                                   │ │
│ │ peerId          │                                                                      │ │
│ ╰─────────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ alias           │  js2                                                                 │ │
│ │ suiAddress      │  0x895d63a8db8bdb07bf450adcaa47fea03cfb1082f4c379b74493a1386b730d6f  │ │
│ │ publicBase64Key │  AQLX+tlpa2ylw057I7nBjyxza9QZQIi0n83KMBSrP7qL2A==                    │ │
│ │ keyScheme       │  secp256k1                                                           │ │
│ │ flag            │  1                                                                   │ │
│ │ peerId          │                                                                      │ │
│ ╰─────────────────┴──────────────────────────────────────────────────────────────────────╯ │
╰────────────────────────────────────────────────────────────────────────────────────────────╯
```

-   **创建多重签名**

```
```











## 4 zkLogin类



## 5 参考资料

https://docs-zh.sui-book.com/concepts/cryptography/transaction-auth/multisig

https://docs-zh.sui-book.com/concepts/cryptography/zklogin

## 6 更多

欢迎关注微信公众号：**Move中文**，开启你的 **Sui Move** 之旅！

![image-20240303160834039](assets/image-20240303160834039.png)
