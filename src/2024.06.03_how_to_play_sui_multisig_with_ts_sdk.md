# 使用TypeScript玩转Sui多签钱包 @SUI Move开发必知必会

## 前言

本文将通过`TypeScript`代码来介绍`Sui`多签钱包的三种实现形式，包括：

-   普通公钥多签钱包
-   `zkLogin`多签钱包
-   混合多签钱包

在正式介绍之前，我们先了解一些基本概念。

### 什么是多签钱包

多签钱包是一种需要**一把或多把**密钥（根据**权重**配置）才能授权的加密货币交易数字钱包。这意味着，跟普通钱包不同，多签钱包可以增加安全性。访问资金或发起链上交易需要多把私钥进行签名，而不是由单一的一把私钥。

### 多签钱包的机制

多签钱包的工作机制涉及**多个密钥**和 `M-of-N` 签名的概念。在 `M-of-N` 设置中，只有 `N` 个密钥中的 `M` 个对交易进行了签名，才能授权该交易。例如，在 `2-of-3` 多签钱包中，存在三个私钥，至少需要两个私钥才能授权交易。

### 多签钱包的优势

-   **提升安全性**：交易需要多个签名
-   **避免错误交易发生**：多个私钥持有人可以进行多重验证避免错误交易发生
-   **提高账户可恢复性**：通过设置合理阈值，若单一持有人私钥丢失，仍然可以使用账户资金

### 多签钱包的不足

- **复杂性**：多重签名钱包的设置、管理和使用要比传统的单签名钱包复杂得多
- **交易速度**：交易完成需要多方协调进行签名，带来一定的交易时延
- **协同风险**：多个参与者共同管理，可能存在信任风险

## 基于普通公钥的多签钱包

### 创建基于普通公钥的多签钱包

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

### 多签地址构造原理

通过调用`.toSuiAddress()`方法，可以得到多签钱包地址。

通过查看[源码](https://github.com/MystenLabs/sui/blob/mainnet-v1.26.2/sdk/typescript/src/multisig/publickey.ts#L187)，我们知道`Sui`多签地址的生成原理如下：

| 所占字节数 | 1            | 2    | 64    | 1    | 64    | 1    | 64    | 1    |
| ---------- | ------------ | ---- | ----- | ---- | ----- | ---- | ----- | ---- |
| **取值**   | 0x03         | 2    | xxx   | 1    | xxx   | 1    | Xxx   | 2    |
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

### 用户消息签名及验签

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
>   ```tsx
>   	/**
>   	 * Combines multiple partial signatures into a single multisig, ensuring that each public key signs only once
>   	 * and that all the public keys involved are known and valid, and then serializes multisig into the standard format
>   	 */
>   	combinePartialSignatures(signatures: string[]): string {
>   		if (signatures.length > MAX_SIGNER_IN_MULTISIG) {
>   			throw new Error(`Max number of signatures in a multisig is ${MAX_SIGNER_IN_MULTISIG}`);
>   		}
>   
>   		let bitmap = 0;
>   		const compressedSignatures: CompressedSignature[] = new Array(signatures.length);
>   
>   		for (let i = 0; i < signatures.length; i++) {
>   			let parsed = parseSerializedSignature(signatures[i]);
>   			if (parsed.signatureScheme === 'MultiSig') {
>   				throw new Error('MultiSig is not supported inside MultiSig');
>   			}
>   
>   			let publicKey;
>   			if (parsed.signatureScheme === 'ZkLogin') {
>   				publicKey = toZkLoginPublicIdentifier(
>   					parsed.zkLogin?.addressSeed,
>   					parsed.zkLogin?.iss,
>   				).toRawBytes();
>   			} else {
>   				publicKey = parsed.publicKey;
>   			}
>   
>   			compressedSignatures[i] = {
>   				[parsed.signatureScheme]: Array.from(parsed.signature.map((x: number) => Number(x))),
>   			} as CompressedSignature;
>   
>   			let publicKeyIndex;
>   			for (let j = 0; j < this.publicKeys.length; j++) {
>   				if (bytesEqual(publicKey, this.publicKeys[j].publicKey.toRawBytes())) {
>   					if (bitmap & (1 << j)) {
>   						throw new Error('Received multiple signatures from the same public key');
>   					}
>   
>   					publicKeyIndex = j;
>   					break;
>   				}
>   			}
>   
>   			if (publicKeyIndex === undefined) {
>   				throw new Error('Received signature from unknown public key');
>   			}
>   
>   			bitmap |= 1 << publicKeyIndex;
>   		}
>   
>   		let multisig: MultiSigStruct = {
>   			sigs: compressedSignatures,
>   			bitmap,
>   			multisig_pk: this.multisigPublicKey,
>   		};
>   		const bytes = bcs.MultiSig.serialize(multisig, { maxSize: 8192 }).toBytes();
>   		let tmp = new Uint8Array(bytes.length + 1);
>   		tmp.set([SIGNATURE_SCHEME_TO_FLAG['MultiSig']]);
>   		tmp.set(bytes, 1);
>   		return toB64(tmp);
>   	}
>   }
>   ```

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

### 交易签名及验签

#### （1）为多签地址领水







### 领水



### 调用合约接口

























































## 基于`zkLogin`的多签







## 在合约中验证多签地址

























































## 参考资料

https://sdk.mystenlabs.com/typescript/cryptography/multisig

https://mp.weixin.qq.com/s/3eqQbgk-u90U4A4LmyKO6w
