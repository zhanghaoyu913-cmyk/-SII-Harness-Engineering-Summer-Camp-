# SII Harness Engineering Summer Camp

> 上海创智学院 / SII Harness Engineering 夏令营任务相关代码与学习记录  
> 本仓库主要用于个人学习、方法复盘与经验分享，不用于任何商业用途。

## 项目简介

本仓库整理了我在 **Harness Engineering** 任务中的本地调试环境、基础框架、LLM 调用接口以及 `MyHarness` 实现入口。该任务的核心目标是在给定训练样本流的条件下，设计一个能够在测试阶段完成文本分类 / 意图识别的 Harness 系统。

仓库内容主要围绕以下问题展开：

- 如何理解 Harness Engineering 的任务形式；
- 如何设计 `update(text, label)` 阶段的外部记忆机制；
- 如何设计 `predict(text)` 阶段的候选收窄、样例检索与 LLM 提示词；
- 如何在有限 prompt token 预算下提升预测稳定性；
- 如何通过本地 DEV 集进行调试、验证和误差分析。

本仓库并不声称提供最优解，也不保证在正式评测中取得特定分数，仅作为个人探索过程的整理与分享。

## 仓库结构

```text
.
├── data/                               # 本地调试数据
├── tokenizer/                          # 本地 token 计数所需 tokenizer
├── Harness Engineering 考核说明（2026年夏）.pdf
├── harness_base.py                     # 官方提供的 Harness 基类
├── llm_client.py                       # OpenAI-Compatible LLM 调用接口
├── requirements.txt                    # Python 依赖
├── run.py                              # 本地调试评测脚本
└── solution.py                         # MyHarness 实现入口
```

## 环境配置

建议使用 Python 3.10 或以上版本。

```bash
pip install -r requirements.txt
```

当前依赖主要包括：

```text
openai
transformers
numpy
```

## LLM 接口配置

本项目中的 `llm_client.py` 使用 OpenAI-Compatible 格式调用模型。使用前需要根据自己的服务修改以下字段：

```python
BASE_URL = "http://your-endpoint/v1"
API_KEY = "EMPTY"
MODEL = "Qwen3-8B"
```

如果使用本地 vLLM、硅基流动、DeepSeek 或其他兼容 OpenAI API 的服务，只需要替换对应的 endpoint、key 和 model 名称即可。

## 本地运行

默认运行：

```bash
python run.py
```

调整并发数：

```bash
python run.py --workers 20
```

调整运行轮数：

```bash
python run.py --runs 4
```

调整 prompt token 上限：

```bash
python run.py --max-prompt-tokens 2048
```

也可以显式指定训练集和验证集：

```bash
python run.py \
  --train data/train_dev.jsonl \
  --dev data/test_dev.jsonl \
  --workers 20 \
  --runs 4 \
  --max-prompt-tokens 2048
```

## 方法思路

本项目的核心实现位于 `solution.py` 中的 `MyHarness` 类。

整体思路可以概括为：

1. **训练阶段记忆构建**

   在 `update(text, label)` 阶段读取训练样本，并维护外部记忆结构，例如：

   - label 到样本的映射；
   - token / char n-gram 特征；
   - label profile；
   - 高频词、关键词、短语统计；
   - 用于测试阶段检索的候选样例池。

2. **测试阶段候选收窄**

   在 `predict(text)` 阶段，先通过轻量级规则或相似度方法缩小候选 label 范围，避免将所有类别一次性塞入 prompt。

3. **检索增强提示词**

   从训练记忆中选取与当前输入最相关、最有区分度的样例，构造成少样本提示词，让 LLM 在候选 label 内做判断。

4. **输出规范化**

   对 LLM 输出进行清洗，尽量保证最终返回值严格等于某个合法 label，避免因为多余解释、标点或格式错误导致判错。

## 注意事项

- 请不要过拟合本地 DEV 集。
- 正式评测数据与本地调试数据可能不同。
- `prompt token` 超过预算后会被截断，因此需要主动控制提示词长度。
- `predict()` 应返回最终标签字符串，而不是解释性文本。
- 不建议在正式方案中过度依赖单一规则，应关注泛化能力与稳定性。

## 非商业声明

本仓库仅用于：

- 个人学习；
- 技术交流；
- 方法复盘；
- Harness Engineering 任务形式的理解与讨论。

本仓库 **不用于任何商业用途**，包括但不限于：

- 商业培训；
- 付费课程；
- 商业咨询；
- 代写、代考、刷分服务；
- 任何以盈利为目的的再分发或二次包装。

未经作者许可，请勿将本仓库内容用于商业化场景。

## 学术诚信声明

本仓库的分享目的在于记录个人学习过程和工程探索经验，而不是鼓励任何违反规则的行为。

使用者应当遵守相关考试、夏令营、评测平台或主办方的规定，包括但不限于：

- 不通过任何方式获取测试集标签；
- 不硬编码测试集答案；
- 不利用漏洞绕过评测流程；
- 不进行刷分、作弊或其他不正当竞争行为；
- 不将本仓库内容直接作为个人原创成果提交。

如果你正在参加相关考核，请以官方规则为准，独立完成自己的实现与报告。

## 版权与引用说明

仓库中如包含主办方提供的说明文件、框架代码或示例数据，其版权归原作者或原机构所有。本仓库仅作学习记录与非商业分享。

如相关内容不适合公开展示，或涉及版权、隐私、规则冲突等问题，请联系作者，我会及时处理或删除。

## 免责声明

本仓库内容仅代表个人学习与探索过程，不代表上海创智学院、SII 或任何相关机构的官方立场。

本仓库中的代码、实验结果和方法分析仅供参考，不保证在任何评测环境中取得特定结果。使用本仓库所产生的任何后果由使用者自行承担。

## Author

Haoyu Zhang

本项目主要用于记录我对 Harness Engineering 任务的理解、实现与反思。欢迎交流讨论，但请尊重学术诚信与非商业分享原则。
