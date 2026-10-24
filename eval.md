# Unsloth 量化后评估步骤（基于 Dynamic 3.0 UD 系列公开方法）

## 步骤 1：构建 Divergence-300@32 保留测试集

1. 从以下来源抽取 300 条“未见过的”保留样本：
   - Terminal-Bench 2.1
   - DeepSWE
   - Harbor
   - MathArena 2025-26
   - 非拉丁语提示
   - 长文档提示
2. 强制隔离：这 300 条样本必须与 imatrix 校准数据零重叠，专门用于检验量化模型是否过拟合校准集[reference1][reference2]。
3. 保存样本清单、来源标签和哈希，写入评估报告。

## 步骤 2：执行 32-token greedy 轨迹解码

1. 对同一 prompt，让 BF16 基线和量化模型分别做 32 个 token 的 greedy argmax 解码。
2. 不使用采样，固定每一步选择概率最高的 token，确保轨迹可复现。
3. 逐 token 对比两条轨迹，计算一致率。
4. 核心逻辑：Top-1 只衡量单步 argmax，32-token 轨迹一致率才能暴露长程推理中的偏移累积和循环问题[reference3]。

## 步骤 3：计算 KL 散度

1. 在标准文本上，计算量化模型输出分布与 BF16 分布之间的 KL 散度。
2. Unsloth 将 KL 散度视为“报告量化误差的黄金标准之一”，因为它与答案翻转（flips）高度相关：MMLU 可能不下降，但部分错误答案会翻转为正确，掩盖真实偏移[reference4]。

## 步骤 4：报告 Top-1 与 MMLU 作为参照

1. Top-1 准确率：量化模型与 BF16 在下一 token argmax 上的一致率。Unsloth 宣称 UD-Q2_K_XL 的 Top-1 比下一个最佳方案高约 8%，文件大小为 9.83GB[reference6]。
2. 5-shot MMLU：用于确认模型在通用知识上没有彻底失效。例如 UD-Q2_K_XL 的 MMLU 约 68.70，BF16 基线约 71.50，下降 -2.80[reference7]。
3. 明确边界：Top-1 和 MMLU 是底线参照，不是核心决策指标；核心决策看 Divergence-300@32 和 KL 散度。

## 步骤 5：生成 Unsloth 风格的评估报告

报告必须包含以下元组，否则结果不可比：

1. BF16 基线 checkpoint 与推理引擎版本。
2. Divergence-300@32 轨迹一致率。
3. KL 散度。
4. Top-1 准确率。
5. 5-shot MMLU。