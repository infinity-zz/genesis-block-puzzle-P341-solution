# Genesis 创世块钱包谜题（P341）完整解法

> 状态：**已解**（2026-10-20 本地完整重建）｜地址 `bc1qfkhx02v89u2qyyyljeczw6hu9sr437y44t7ae5yf09thrdukfqesnjg2wj`
> 注：链上奖金（119 笔注资共 947,220 sats）已于 2026-10-17 被独立求解者 `bc1qdu8mh9qjtjhy89mvhmcgf66ul6ld92rdr6qzad` 全部 sweep；本库于 2026-10-20 独立重建全部密钥材料，与链上见证逐字节互证。

## 一、结论：双共创者配方

两个共创者（PUB1 / PUB2）使用**完全相同的输入**，唯一区别是 PBKDF2 轮数：

| 项 | PUB1 | PUB2 |
|---|---|---|
| cut | `GB[262:400]`（138 hex 字符） | 同左 |
| entropy | `SHA256(cut)[:16]` | 同左 |
| 助记词 | `scan amateur fit scrap light over fire flip cushion reject hurt ugly` | 同左 |
| passphrase | `Satoshi Nakamoto` | 同左 |
| PBKDF2-HMAC-SHA512 轮数 | **181505**（= 创世时间 18:15:05 UTC） | **200913** |
| account | 1231006505（创世区块 `time` 字段原值） | 同左 |
| 派生路径 | `m/48'/0'/1231006505'/2'/0/0` | 同左 |
| 公钥 | `027b497b1d82986ebe11bca71108ca43446c00edb069cc839638a0b76529130906` | `033614b02e8e3aeeed9e14ac67839f85d54b694eafd91ca07f1afeb06510832ec1` |
| 私钥 | `60d09998a128dcd2642cc7f16db24f1525cf18564e976a3e87fa5e48e24874e9` | `4b18979d14cb062679c4b16a9b04bd163138720aa4ffc4c6245e9cb8203b93c6` |
| 主指纹 | `0852014c` | `040e2791` |

cut 文本即 The Times 头条的 hex 编码：`5468652054696d65732030332f4a616e2f32303039204368616e63656c6c6f72206f6e206272696e6b206f66207365636f6e64206261696c6f757420666f722062616e6b73`
（= "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks"）。

**托管结构**：P2WSH（witness v0，32 字节程序），脚本 `OP_2 <PUB1> <PUB2> OP_2 OP_CHECKMULTISIG`
= `5221027b…09306` `21033614…32ec1` `52ae`，sha256 = `4dae67a9872f1402109f9670276afc2c0758f895aafddcd089795771b7964833`，即地址的见证程序。

## 二、六重独立证据

1. **主指纹复现**（Python/coincurve，非 C 二进制）：PUB2 seed → BIP32 主钥 → FP `040e2791` ✓
2. **PUB1 控制组**：同一 cut/PP、181505 轮 → FP `0852014c` → 路径 `m/48'/0'/1231006505'/2'/0/0` → PUB1 公钥逐字节一致 ✓
3. **PUB2 公钥命中**：200913 轮同路径 → `033614b0…832ec1` ✓
4. **地址端到端**：`sha256(OP_2 <PUB1> <PUB2> OP_2 OP_CHECKMULTISIG)` = 地址 bech32 解码程序（校验和验证通过，PUB1 在前排序）✓
5. **链上见证脚本**：sweep 交易见证栈第 4 元素与本重建脚本逐字节一致 ✓
6. **链上签名**：BIP143 sighash `84dc9c8b…b8567` 下，sig1 对 PUB1 验证通过、sig2 对 PUB2 验证通过，交叉赋值均失败 ✓

## 三、关键方法：PBKDF2 轮数穷举

此前所有扫描（88 个候选轮数、181505/2048 全空间、24 词 entropy × salt 变种）全负，因为 **200913 不在任何人工候选列表里**。突破手段：

- `pub2_exhaustive.c`：全 C 管线扫描器（SHA256 → BIP39 → PBKDF2 链式 HMAC → libsecp256k1 → RIPEMD160），433 个 138 字符窗口 × 轮数 1–2,100,000 穷举。
- 校准（三条真值锚点）：`verify 262 181505 "Test Example"` → `68ab4226`（作者 blk 971770 测试向量）；`verify 262 181505 "Satoshi Nakamoto"` → `0852014c`（PUB1）；扫描模式控制组窗口 262 @181505 命中 PUB1。
- 单窗口全轮数扫描 79 秒，在第一个窗口（262）即命中 **轮数 200913**。

### 排过的坑（记录在案）

- C 源里 GB 字符串 5 处 `2d`（连字符）转录错误（"Chancellor-on-brink"），以 `genesis_block.hex` 为准重建后校准才通过。
- 手打地址串转录错误导致 bech32 解出 20 字节程序；地址一律从 `addr_stats.json` 权威加载。
- coincurve 21.0 的 `PublicKey.verify` **返回布尔值、验证失败不抛异常**；`hasher=None` 表示不预哈希。包装器必须检查返回值。
- BIP143 验证需先剥掉 DER 末尾的 sighash 类型字节（libsecp256k1 严格 DER 解析）。
- `gcc ... 2>&1 | head -20` 会因 SIGPIPE 杀死 gcc 致二进制不更新，且 `$?` 取到的是 head 的退出码——编译验证必须重定向 stderr 到文件。
- 并行 Edit 同一文件会互相覆盖（本会话两次踩中），同文件的多次修改必须顺序执行。

## 四、复现

```bash
# 四步全过：主指纹 / PUB1 控制组 / PUB2 公钥 / P2WSH 地址
/home/z/.workbuddy/binaries/python/envs/default/bin/python3 data/_genesis/verify_pub2_hit.py

# 链上签名最终锚点（BIP143）
/home/z/.workbuddy/binaries/python/envs/default/bin/python3 data/_genesis/verify_sweep_sigs.py
```

产物：`data/_genesis/PUB2_SOLUTION.json`（机器可读）、`tools/inject_genesis_solution.py`（题库注入源）。
题库条目：P341（`谜题库/genesis/P341-genesis-已解/`），已入 `私钥汇总.md`。
