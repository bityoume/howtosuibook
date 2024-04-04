# SUI Move开发必知必会——如何迁移到SUI Move 2024？

*rzexin 2024.04.04*

## 前言

`SUI Move`在2024年迎来重大更新，引入了许多新特性，包括：**方法语法（`method syntax`）**、**位置域（`positional fields`）**、**循环标签（`loop labels`）**等。这些更新为`Move`编程语言引入了新的定义数据和调用函数的方式，使在`Sui`上构建应用程序更加高效灵活，也为未来要推出的新功能铺平道路。

更新内容包括了：

- **新特性（`New features`）**:可以向前兼容的特性，即新旧语法都能进行正常编译
  - 
- **不兼容更新（`Breaking changes `）**：无法向前兼容，即不再支持旧语法，若编译会报错，历史代码可以`sui move migrate`命令自动迁移到新的写法
  - 

## 如何进行自动迁移

### 自动迁移方法

- 使用`sui 1.22.0`及以上版本
- 在合约根目录里执行`sui move migrate`命令
  - 终端会显示要进行的更改的合约差异，如果接受更改，会自动将现存**早期版本（`legacy`）**的合约，迁移成**新版合约（`2024.beta`）**代码，并会更新`Move.toml`文件，同时也会生成一个`migration.patch`文件，将变更差异记录在其中

### 自动迁移修改内容



### 自动迁移实操

```bash
$ sui --version
sui 1.22.0-0362997459

$ sui move migrate
Please select one of the following editions:
1) 2024.beta
2) legacy
Selection (default=1): 1

Would you like the Move compiler to migrate your code to Move 2024? (Y/n) 
Generated changes . . .
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING nfts_num
The following changes will be made.
============================================================
--- sources/num.move
+++ sources/num.move
@@ -7 +7 @@
-    struct Num has key, store {
+    public struct Num has key, store {
@@ -12 +12 @@
-    struct NumIssuerCap has key {
+    public struct NumIssuerCap has key {
============================================================
Apply changes? (Y/n) 

Updating "sources/num.move" . . .

Changes complete
Wrote patchfile out to: ./migration.patch

Recorded edition in 'Move.toml'
```

## SUI Move 2024语法实战

> 以下会通过一个合约示例，展示`SUI Move 2024`主要新增的特性，考虑到合约逻辑实现的连贯性，会混合使用**新特性**和**不兼容更新**的语法。
>
> 为了一目了然会标题旁做一下标记，以区分：**新特性（🎉）**、**不兼容更新（💥）**，并附上官方文档链接，方便大家查看。

### 可嵌套`use`别名🎉

> [Nested `use` and standard library defaults](https://github.com/tnowacki/sui/blob/main/docs/content/guides/developer/advanced/move-2024-migration.mdx?nested-use-and-standard-library-defaults)

支持嵌套使用别名，可以带来代码编写的简洁。

```rust
module bityoume::sui_move_2024 {
    // legacy
    // use sui::balance; 
    // use sui::coin::{Self, Coin};

    // 2024.beta
    use sui::{balance, coin::{Self, Coin}};
}
```

### 常用标准库默认引入🎉

> [Nested `use` and standard library defaults](https://github.com/tnowacki/sui/blob/main/docs/content/guides/developer/advanced/move-2024-migration.mdx?nested-use-and-standard-library-defaults)

以下声明将会自动包含在每个模块中，不需要再写了。这些定义很常用，几乎每个模块都会用到，默认引入，可以减少开发者重复去编写。

```rust
// legacy
use std::vector;
use std::option::{Self, Option};
use sui::object::{Self, ID, UID};
use sui::transfer;
use sui::tx_context::{Self, TxContext};

// 2024.beta
<空>
```

### 数据类型可见性💥

> [Datatype visibility requirements](https://github.com/tnowacki/sui/blob/main/docs/content/guides/developer/advanced/move-2024-migration.mdx?nested-use-and-standard-library-defaults#datatype-visibility-requirements)



## 参考资料

https://blog.sui.io/move-2024-migration-guide/

https://github.com/tnowacki/sui/blob/main/docs/content/guides/developer/advanced/move-2024-migration.mdx?ref=blog.sui.io

https://blog.sui.io/move-edition-2024-update/

https://docs.sui.io/guides/developer/advanced/move-2024-migration

https://medium.com/building-on-sui/move-towards-rust-the-evolution-of-sui-move-in-2024-aedeeb3def87

## 更多

欢迎关注微信公众号：**Move中文**，开启你的 **Sui Move** 之旅！

![image-20240402223437320](assets/move_cn.png)