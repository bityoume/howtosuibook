# 玩转Sui多签钱包1：普通公钥多签钱包 @SUI Move开发必知必会

## 1 前言

本系列文章将会分为3部分内容，包括：

-   **普通公钥多签钱包**
-   **`zkLogin`多签钱包**
-   **合约中的多签钱包**

在正式介绍之前，我们先了解一些基本概念。

### 1.1 什么是多签钱包

多签钱包是一种需要**多把**密钥才能完成交易授权的加密货币交易数字钱包。这意味着，跟普通钱包不同，多签钱包可以增加安全性。访问资金或发起链上交易需要多把私钥进行签名，而不是由单一的一把私钥。

### 1.2 多签钱包的机制

多签钱包的工作机制涉及**多个密钥**和 `M-of-N` 签名的概念。在 `M-of-N` 设置中，只有 `N` 个密钥中的 `M` 个对交易进行了签名，才能授权该交易。例如，在 `2-of-3` 多签钱包中，存在三个私钥，至少需要两个私钥才能授权交易。

### 1.3 多签钱包的优势

-   **提升安全性**：交易需要多个签名
-   **避免错误交易发生**：多个私钥持有人可以进行多重验证避免错误交易发生
-   **提高账户可恢复性**：通过设置合理阈值，若单一持有人私钥丢失，仍然可以使用账户资金

### 1.4 多签钱包的不足

- **复杂性**：多重签名钱包的设置、管理和使用要比传统的单签名钱包复杂得多
- **交易速度**：交易完成需要多方协调进行签名，带来一定的交易时延
- **协同风险**：多个参与者共同管理，可能存在信任风险

## 2 基于普通公钥的多签钱包

### 2.1 创建基于普通公钥的多签钱包

以下代码将创建三个公钥的多签钱包，其中公钥1和公钥2的权重为1，公钥3的权重为2，阈值为2。即：发起一笔交易，要么私钥1和私钥2进行多签、要么私钥3进行单签才能成功。

```typescript
import { Ed25519Keypair } from "@mysten/sui/keypairs/ed25519";
import { MultiSigPublicKey } from "@mysten/sui/multisig";

const kp1 = new Ed25519Keypair();
const kp2 = new Ed25519Keypair();
const kp3 = new Ed25519Keypair();

function createMultisigAddressWithPublicKey(): MultiSigPublicKey {
  const multiSigPublicKey = MultiSigPublicKey.fromPublicKeys({
    threshold: 2,
    publicKeys: [
      {
        publicKey: kp1.getPublicKey(),
        weight: 1,
      },
      {
        publicKey: kp2.getPublicKey(),
        weight: 1,
      },
      {
        publicKey: kp3.getPublicKey(),
        weight: 2,
      },
    ],
  });

  return multiSigPublicKey;
}

const multiSigPublicKey = createMultisigAddressWithPublicKey();
console.log(multiSigPublicKey.toSuiAddress());
```

### 2.2 多签地址构造原理

通过调用`.toSuiAddress()`方法，可以得到多签钱包地址。

通过查看[源码](https://github.com/MystenLabs/sui/blob/mainnet-v1.26.2/sdk/typescript/src/multisig/publickey.ts#L187)，我们知道`Sui`多签地址的生成原理如下：

| 所占字节数 | 1            | 2    | 64    | 1    | 64    | 1    | 64    | 1    |
| ---------- | ------------ | ---- | ----- | ---- | ----- | ---- | ----- | ---- |
| **取值**   | 0x03         | 2    | xxx   | 1    | xxx   | 1    | xxx   | 2    |
| **说明**   | 多签地址标识 | 阈值 | 公钥1 | 权重 | 公钥2 | 权重 | 公钥3 | 权重 |

将以上字节拼接后，进行`blake2b`哈希后，转成16进制字符串，再在开通拼接上`0x`后，即为`Sui`多签地址。

```tsx
	/**
	 * Return the Sui address associated with this MultiSig public key
	 */
	override toSuiAddress(): string {
		// max length = 1 flag byte + (max pk size + max weight size (u8)) * max signer size + 2 threshold bytes (u16)
		const maxLength = 1 + (64 + 1) * MAX_SIGNER_IN_MULTISIG + 2;
		const tmp = new Uint8Array(maxLength);
		tmp.set([SIGNATURE_SCHEME_TO_FLAG['MultiSig']]);

		tmp.set(bcs.u16().serialize(this.multisigPublicKey.threshold).toBytes(), 1);
		// The initial value 3 ensures that following data will be after the flag byte and threshold bytes
		let i = 3;
		for (const { publicKey, weight } of this.publicKeys) {
			const bytes = publicKey.toSuiBytes();
			tmp.set(bytes, i);
			i += bytes.length;
			tmp.set([weight], i++);
		}
		return normalizeSuiAddress(bytesToHex(blake2b(tmp.slice(0, i), { dkLen: 32 })));
	}
```

### 2.3 用户消息签名及验签

#### （1）需要联合签名

-   根据前面创建的多签，公钥1和公钥2的权重都是1，阈值为2，则需要私钥1和私钥2分别进行多签，再调用接口`combinePartialSignatures`合并成完整的签名。

-   用户可以调用`verifyPersonalMessage`接口，传入签名消息和合并后的完整签名进行验签，若验签成功将返回`true`，验签失败返回`false`

```tsx
async function signVerifyMsg() {
  const message = new TextEncoder().encode("hello sui");

  const signature1 = (await kp1.signPersonalMessage(message)).signature;
  const signature2 = (await kp2.signPersonalMessage(message)).signature;

  const combinedSignature = multiSigPublicKey.combinePartialSignatures([
    signature1,
    signature2,
  ]);

  const isValid = await multiSigPublicKey.verifyPersonalMessage(
    message,
    combinedSignature
  );

  console.log(isValid);
}
```

>   **签名合并接口代码实现为：**
>
>   https://github.com/MystenLabs/sui/blob/mainnet-v1.26.2/sdk/typescript/src/multisig/publickey.ts#L251
>

#### （2）MultiSigSigner签名

-   如果只是一个签名者就够了，例如前面创建的公钥3权重为2，阈值为2，估使用私钥3进行单签交易就好，我们可以使用`getSigner`获取到签名者，然后使用该签名者调用`signPersonalMessage`获取签名即可。

-   验签方法同上。

```tsx
async function signVerifyMsg() {
  const message = new TextEncoder().encode("hello sui");

  const signer = multiSigPublicKey.getSigner(kp3);
  const { signature } = await signer.signPersonalMessage(message);

  const isValid = await multiSigPublicKey.verifyPersonalMessage(
    message,
    signature
  );

  console.log(isValid);
}
```

### 2.4 交易签名及验签

#### （1）为多签地址领水

> 可以在水龙头领水，或者使用有资金的地址转账，可参考：[SUI CLI常用命令解析1——Client @SUI Move开发必知必会](https://learnblockchain.cn/article/7832)

地址下需要有2个`COIN`，一个用于转账操作，一个支付`GAS`：

```bash
# 将多签地址加入环境变量
$ export MULTISIG_ADDR=0xe2f8d8835b7e5e7c1cea8aeac814a03da0b9892fb876a4cf245d7cbe5d6b7808

# 将构成多签地址的三个地址加入环境变量
export ADDR1=0xec6f2a5687d53fe9b9b7348bbefd80784c5a1de7d99742f64c1ea1e04e18592d
export ADDR2=0xd1b3a97f5d9abe811f8ae97e9bf2d868b2f3ff5c66cc40d5940be432050310ff
export ADDR3=0xc877f47e5f05356f28a4f9ef01c09b93c8a7431c288480b9c7c7d3bf2b312f30

# 查看多签钱包当前余额
$ sui client gas $MULTISIG_ADDR --json
[
  {
    "gasCoinId": "0x3ee028858c3c90035265a412171adbe1ee5f6b0d96998f7719c80fa3b6c2cee3",
    "mistBalance": 1000000000,
    "suiBalance": "1.00"
  },
  {
    "gasCoinId": "0xff1e2cee6496506ddd70ac3d4815e664ee104aba415f49d6b1deb05d1a1f3ce0",
    "mistBalance": 1000000000,
    "suiBalance": "1.00"
  }
]
```

#### （2）构造转账交易

本示例将多签地址`MULTISIG_ADDR`的中的`...cee3`转账给`ADDR1`，`...3ce0`用于支付`GAS`费。

需要地址1和地址2对应的2把私钥分别进行签名，才能合并成有效的完成签名，并完成交易的成功执行。

代码如下：

```tsx
// 没有任何COIN的地址
const ADDR1 =
  "0xec6f2a5687d53fe9b9b7348bbefd80784c5a1de7d99742f64c1ea1e04e18592d";

// 拥有2个COIN的多签钱包地址
const MULTISIG_ADDR =
  "0xe2f8d8835b7e5e7c1cea8aeac814a03da0b9892fb876a4cf245d7cbe5d6b7808";

async function transfer() {
  // 使用测试网络
  const rpcUrl = getFullnodeUrl("testnet");
  const client = new SuiClient({ url: rpcUrl });

  // 从多签钱包中取出2个COIN
  const coins = await client.getAllCoins({ owner: MULTISIG_ADDR, limit: 2 });

  // 构造交易
  const tx = new Transaction();

  // 设置发送方为多签钱包地址
  tx.setSender(MULTISIG_ADDR);

  // 将COIN1装发给ADD1
  tx.transferObjects([coins.data[0].coinObjectId], ADDR1);

  // 设置COIN支付GAS
  tx.setGasPayment([
    {
      objectId: coins.data[1].coinObjectId,
      version: coins.data[1].version,
      digest: coins.data[1].digest,
    },
  ]);
  tx.setGasBudget(100000000);

  // 构造待签名串
  const txBytes = await tx.build({ client });

  // 用户1签名
  const signature1 = (await kp1.signTransaction(txBytes)).signature;

  // 用户2签名
  const signature2 = (await kp2.signTransaction(txBytes)).signature;

  // 合并签名
  const combinedSignature = multiSigPublicKey.combinePartialSignatures([
    signature1,
    signature2,
  ]);

  // 本地校验签名是否正确
  const isValid = await multiSigPublicKey.verifyTransaction(
    txBytes,
    combinedSignature
  );

  // 若校验通过，这里将返回true
  console.log(isValid);

  // 发送交易
  await client
    .executeTransactionBlock({
      transactionBlock: txBytes,
      signature: combinedSignature,
      requestType: "WaitForLocalExecution",
      options: {
        showEffects: true,
      },
    })
    .then((txRes) => {
      const status = txRes.effects?.status?.status;
      if (status !== "success") {
        throw new Error(
          `Could not transfer coin! ${txRes.effects?.status?.error}`
        );
      }
      return txRes;
    })
    .catch((err) => {
      throw new Error(`Error thrown: Could not transfer coin!: ${err}`);
    });
}
```

#### （3）交易执行结果

若交易执行成功，我们将在`ADDR1`中看到转入了多签地址的`COIN1`。

- **查看多签地址**

可见多签地址的中的`...cee3`已转出，`...3ce0`支付了`GAS`费，金额相应减少了：

```tsx
$ sui client gas $MULTISIG_ADDR --json
[
  {
    "gasCoinId": "0xff1e2cee6496506ddd70ac3d4815e664ee104aba415f49d6b1deb05d1a1f3ce0",
    "mistBalance": 997970360,
    "suiBalance": "0.99"
  }
]
```

- **查看`ADDR1`地址**

可见`ADDR1`地址已经接收到了，从多签钱包中，转账过来的代币。

```bash
$ sui client gas $ADDR1 --json
[
  {
    "gasCoinId": "0x3ee028858c3c90035265a412171adbe1ee5f6b0d96998f7719c80fa3b6c2cee3",
    "mistBalance": 1000000000,
    "suiBalance": "1.00"
  }
]
```

## 3 参考资料

https://sdk.mystenlabs.com/typescript/cryptography/multisig

## 4 更多

欢迎关注微信公众号：**Move中文**，开启你的 **Sui Move** 之旅！

![image-20240303160834039](assets/image-20240303160834039.png)