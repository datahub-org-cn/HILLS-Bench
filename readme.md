# HILLS-Bench

HILLS-Bench 是面向高原病相关任务的大语言模型中英文测评题库，包含 **1,000 道配对题目**。中文版和英文版各有 1,000 条记录，以相同的 `id` 对应；两个语言版本合计 2,000 条记录，并非 2,000 道独立题目。

HILLS-Bench is a bilingual question dataset for evaluating large language models on high altitude illness. It contains **1,000 paired questions**, provided as 1,000 Chinese records and 1,000 English records with matching identifiers.

## 数据文件 / Data files

| 文件 / File | 内容 / Contents |
| --- | --- |
| [`Bench/Bench-zh.xlsx`](Bench/Bench-zh.xlsx) | 中文题库，1,000 道题 / Chinese questions, 1,000 records |
| [`Bench/Bench-en.xlsx`](Bench/Bench-en.xlsx) | 英文题库，1,000 道题 / English questions, 1,000 records |
| [`guidelines/guidelines.xlsx`](guidelines/guidelines.xlsx) | 13 条指南、共识及参考网页的来源索引 / Source index of 13 guidelines, consensus documents and reference webpages |

两个题库工作簿的工作表名均为 `HILLS-Bench`。`guidelines` 目录仅提供来源索引，不包含文献全文。

Both question workbooks use the sheet name `HILLS-Bench`. The `guidelines` directory contains the source index rather than full-text documents.

## 任务分类 / Task categories

| 中文分类 | English category | 每个语言版本题数 / Questions per language |
| --- | --- | ---: |
| 场景诊断 | Scenario-based diagnosis | 250 |
| 急救决策 | Emergency care decision-making | 250 |
| 风险沟通 | Risk communication | 250 |
| 知识回忆 | Knowledge recall | 250 |
| 合计 / Total | | **1,000** |

## 字段说明 / Data schema

| 字段 / Field | 说明 / Description |
| --- | --- |
| `id` | 题目编号，`HILLS-0001` 至 `HILLS-1000`；用于连接两个语言版本 / Identifier used to join the language versions |
| `question` | 题干 / Question stem |
| `options` | 包含五个字符串的 Python 列表字面量，以单元格文本保存；选项标签统一为 `A.` 至 `E.` / Five option strings stored as a Python list literal in one cell, labelled `A.` through `E.` |
| `correct_answer` | 正确答案字母，取值为 `A` 至 `E` / Answer key, one letter from `A` to `E` |
| `dimension` | 题目所属分类 / Task category |

来源索引包含 `guideline_id`、`title`、`url`、`doi` 四列。`url` 为原始发布机构或出版商页面；没有 DOI 的参考网页在 `doi` 列留空。来源索引与题库分别保存，当前题目记录中不包含逐题文献编号。

The source index has four columns: `guideline_id`, `title`, `url` and `doi`. URLs point to the original publisher or issuing organization. The DOI field is blank for reference webpages without a DOI. The bibliography is stored separately; individual question records do not contain source identifiers.

## 编制与翻译审核 / Preparation and translation review

题目由 **1 名研究者编写，另 1 名研究者校验**。中英文翻译使用 **`gpt-6-astra`，温度设置为 `0`**。翻译结果由 **2 名研究者按分类进行分层抽检，各类别抽检 20%**，即每类 50 道、共 200 道配对题目。

One researcher wrote the questions and another researcher checked them. Translation between Chinese and English used **`gpt-6-astra` at a temperature of `0`**. Two researchers reviewed translations using stratified spot-checking: **20% of each category**, equivalent to 50 paired questions per category and 200 paired questions overall.

两个语言版本使用相同的题目编号、选项顺序和答案字母。选项标签已统一为 `A.`、`B.`、`C.`、`D.`、`E.`。

The language versions share question identifiers, option order and answer keys. Option labels are standardized as `A.`, `B.`, `C.`, `D.` and `E.`.

## 读取示例 / Reading the data

以下示例需要 Python、`pandas` 和 `openpyxl`，在仓库根目录运行。`options` 不是 JSON；使用 `ast.literal_eval` 解析，不使用 `eval`。

Run the following example from the repository root with Python, `pandas` and `openpyxl` installed. The `options` field is a Python list literal rather than JSON; parse it with `ast.literal_eval`.

```python
import ast
import pandas as pd

en = pd.read_excel("Bench/Bench-en.xlsx", sheet_name="HILLS-Bench")
zh = pd.read_excel("Bench/Bench-zh.xlsx", sheet_name="HILLS-Bench")

en["options"] = en["options"].map(ast.literal_eval)
zh["options"] = zh["options"].map(ast.literal_eval)

paired = zh.merge(en, on="id", suffixes=("_zh", "_en"), validate="one_to_one")
print(en.groupby("dimension").size())
print(paired.loc[0, ["id", "question_zh", "question_en"]])
```

## 测评使用 / Evaluation use

可将题干和五个选项提供给待测模型，并将输出的答案字母与 `correct_answer` 比较，按整体、分类及语言分别统计准确率。配对 `id` 也可用于比较同一题在中英文下的表现。

For evaluation, provide the question and its five options to a model, then compare its predicted letter with `correct_answer`. Report accuracy overall, by category and by language. Matching identifiers also support comparisons between the Chinese and English versions of the same question.

测评时应记录题库提交版本、模型名称和版本、提示词、生成参数、答案提取规则及无效输出处理方式。应在评测输入中排除 `correct_answer`。仓库目前提供题库及参考来源索引，不包含模型测评结果或官方评分脚本。

Record the dataset commit, model name and version, prompt, generation settings, answer extraction rules and handling of invalid outputs. Exclude `correct_answer` from model inputs. This repository provides the questions and source index; it does not currently include model evaluation results or an official scoring script.

## 作者与资助 / Authors and funding

Qiang Li, Miaofu Ning, Shenghui Cao, Guofu Yin, Yingkao Shen, JinYun Shu

EHV Power Transmission Company of China Southern Power Grid Co., Ltd., Dali Bureau, Dali 671014, China

第一作者及通讯作者 / First and corresponding author: **Qiang Li**.

This work was supported by China Southern Power Grid Science and Technology Innovation Project (No. CGYKJXM20240185).

## 许可与引用 / License and citation

本仓库尚未指定数据许可证。文献和参考网页的权利归相应权利人所有。引用数据时，请注明 HILLS-Bench、仓库地址、所用提交版本及访问日期。

A dataset license has not yet been specified. Referenced publications and webpages remain subject to their respective rights holders' terms. When citing the dataset, include HILLS-Bench, the repository URL, the commit used and the access date.

Repository: <https://github.com/datahub-org-cn/HILLS-Bench>
