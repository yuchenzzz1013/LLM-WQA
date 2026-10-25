# Unsloth 量化评估：数据集与指标对应

## 核心结论
Unsloth 的 KLD/PPL 保真度指标，**几乎全部在 WikiText-2 上测量**。

## 主测量集：WikiText-2
- 数据集：`Salesforce/wikitext` → `wikitext-2-raw-v1`，split=`test`
- 处理：保留空行，行间用 `\n` 拼接；切分为**非重叠 512 token 块**
- 参考规模：约 **565 块 / 288,715 token 位置**
- 对比：参考模型（BF16/FP16） vs 待测量化模型
- KLD 方向：`KL(参考 || 待测)`

## WikiText-2 上计算的指标
| 指标 | 含义 |
|---|---|
| **Mean KLD** | 所有 token 位置 KL 散度的平均值 |
| **99.9% KLD** | KL 散度的 99.9 百分位数，反映最坏情况 |
| **Same Top-1** | 两模型最高概率 token 一致的比例 |
| **PPL Change** | 待测模型 PPL 相对参考模型的变化 |

> 四项指标均来自**同一批 token 位置**，只是聚合方式不同。

## 其他数据集的作用
| 数据集 | 用途 |
|---|---|
| **C4** | 仅辅助计算 **PPL**，不用于 KLD |
| **FineWeb-Edu** | 仅辅助计算 **PPL**，不用于 KLD |
| **自定义校准集** | 仅用于量化 imatrix 校准，**不测 KLD** |
| **下游任务基准** | 仅验证真实能力，**不是 KLD/PPL 测量场所** |

## 只测 KLD 的最小做法
只需：
1. WikiText-2 test 文本；
2. 参考模型与待测模型；
3. 同一 tokenizer 下逐 token 取全词表分布；
4. 计算 **Mean KLD** 和 **99.9% KLD**。

Same Top-1 和 PPL 可从同一批 logits 顺带导出。