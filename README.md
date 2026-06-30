# Quant-files

## 项目说明

本项目用于整合小组三位成员分别构建的股票因子，并将不同因子统一对齐到同一个数据表中。

最终输出两份内容一致的文件：

```text
factor_panel.pkl
factor_panel.csv
```

最终数据采用宽表形式：

* 每一行代表某只股票在某个调仓时间的记录；
* 每一个因子单独占一列；
* 所有股票统一属于中证800。

---

## 统一研究范围

```text
2016-01-01 至 2025-12-31
```

项目中的因子最高频率为月度，但不要求所有因子都按月更新。

允许包含：

* 月度因子；
* 季度因子；
* 半年度因子；
* 年度因子。

年度或其他低频因子可以只在对应时间更新，再对齐到月度数据表中。不同因子的原始日期不需要完全一致，但所有数据都必须位于统一研究时间范围内。

---

## 最终数据格式

最终数据表包含以下基础字段：

| 字段名              | 含义       |
| ---------------- | -------- |
| `symbol`         | 股票代码     |
| `company_name`   | 公司名称     |
| `index_name`     | 指数名称     |
| `index_code`     | 指数代码     |
| `rebalance_date` | 调仓时间     |
| 其他列              | 不同因子的因子值 |

固定字段：

```text
index_name = 中证800
index_code = 000906
```

示例：

| symbol | company_name | index_name | index_code | rebalance_date |  SUS |   SUG |   AQ |
| ------ | ------------ | ---------- | ---------- | -------------- | ---: | ----: | ---: |
| 000001 | 平安银行         | 中证800      | 000906     | 2025-01-27     | 0.52 | -0.16 | 0.83 |
| 600519 | 贵州茅台         | 中证800      | 000906     | 2025-01-27     | 1.14 |  0.62 | 0.37 |

---

## 单个因子文件要求

每位成员提交自己的因子文件，至少包含：

```text
symbol
rebalance_date
因子列
```

例如：

| symbol | rebalance_date |  SUS |
| ------ | -------------- | ---: |
| 000001 | 2025-01-27     | 0.52 |
| 600519 | 2025-01-27     | 1.14 |

一位成员负责多个因子时，也可以放在同一个文件中：

| symbol | rebalance_date |  SUS |   SUG |
| ------ | -------------- | ---: | ----: |
| 000001 | 2025-01-27     | 0.52 | -0.16 |
| 600519 | 2025-01-27     | 1.14 |  0.62 |

公共字段，例如公司名称、指数名称和指数代码，由最终合并脚本统一添加。

---

## 数据格式要求

### 股票代码

股票代码字段统一为：

```text
symbol
```

要求：

* 使用字符串格式；
* 统一为六位；
* 不包含 `.SH` 或 `.SZ` 等交易所后缀。

正确格式：

```text
000001
600519
300750
```

### 时间字段

时间字段统一为：

```text
rebalance_date
```

要求：

* 转换为日期格式；
* 日期范围位于 `2016-01-01` 至 `2025-12-31`；
* 不要求所有因子的日期完全相同；
* 同一个因子中，`symbol + rebalance_date` 必须唯一。

### 因子值

因子列要求：

* 每个因子单独占一列；
* 使用数值格式；
* 因子去极值（例如1%和99%缩头尾）
* 缺失值统一使用 `NaN`；
* 不使用 `0` 代替缺失值；
* 不包含正无穷或负无穷。

---

## 推荐目录结构

```text
multi-factor-alignment/
│
├── README.md
├── .gitignore
│
├── data/
│   ├── raw_factors/
│   │   ├── member_1/
│   │   ├── member_2/
│   │   └── member_3/
│   │
│   ├── reference/
│   │   ├── stock_name_mapping.pkl
│   │   └── monthly_dates.pkl
│   │
│   └── processed/
│       ├── factor_panel.pkl
│       └── factor_panel.csv
│
├── scripts/
│   ├── validate_factors.py
│   └── merge_factors.py
│
└── notebooks/
    └── alignment_check.ipynb
```

目录说明：

* `data/raw_factors/`：三位成员分别上传自己的因子文件；
* `data/reference/`：统一的股票名称、日期等公共数据；
* `data/processed/`：最终合并结果；
* `scripts/`：数据检查和合并脚本；
* `notebooks/`：临时分析和人工检查。

---

## 因子对齐流程

### 1. 各成员整理因子文件

每位成员将自己的因子整理为：

```text
symbol + rebalance_date + 因子列
```

并保存为 `.pkl` 或 `.csv` 文件。

### 2. 检查单个因子文件

合并前检查：

* 股票代码是否为六位字符串；
* 日期是否位于统一时间范围；
* 是否存在重复记录；
* 因子列是否为数值；
* 是否存在无穷值；
* 因子缺失率是否合理。

### 3. 建立月度主表

最终结果以月度数据表为基础，主键为：

```text
symbol + rebalance_date
```

月度因子可以直接按股票代码和日期合并。

年度、季度等低频因子，可以按照其实际更新日期对齐到月度表，并在下一次更新前保留最近一期可用值。

低频因子不能提前使用未来数据。

### 4. 合并所有因子

所有因子按照以下两个字段进行合并：

```text
symbol
rebalance_date
```

推荐使用左连接保留完整主表，而不是直接使用内连接删除存在缺失因子的股票。

### 5. 添加公共字段

合并完成后统一添加：

```text
company_name
index_name
index_code
```

最终列顺序建议为：

```text
symbol
company_name
index_name
index_code
rebalance_date
因子1
因子2
因子3
...
```

### 6. 导出最终文件

最终从同一个 DataFrame 导出：

```text
data/processed/factor_panel.pkl
data/processed/factor_panel.csv
```

CSV 文件建议使用：

```text
utf-8-sig
```

编码，方便使用 Excel 打开中文公司名称。

---

## GitHub 协作方式

主分支统一使用：

```text
main
```

每位成员建立自己的分支，例如：

```text
member-1-factors
member-2-factors
member-3-factors
```

每位成员主要修改自己的目录：

```text
data/raw_factors/member_1/
data/raw_factors/member_2/
data/raw_factors/member_3/
```

推荐操作流程：

```bash
git checkout main
git pull origin main

git checkout member-1-factors
git merge main
```

完成修改后：

```bash
git add .
git commit -m "Add aligned SUS and SUG factors"
git push origin member-1-factors
```

然后在 GitHub 上创建 Pull Request，检查后再合并到 `main`。

---

## 文件命名规范

推荐使用清晰的因子名称：

```text
SUS.pkl
SUG.pkl
AQ.pkl
TurnoverShock20.pkl
RSRS_ADJ.pkl
```

不建议使用：

```text
data1.pkl
new.pkl
final2.pkl
最新版.pkl
修改后.pkl
```

正式合并结果固定命名为：

```text
factor_panel.pkl
factor_panel.csv
```

---

## 最终检查清单

提交最终文件前确认：

* [ ] 研究时间范围为 `2016-01-01` 至 `2025-12-31`；
* [ ] 股票代码全部为六位字符串；
* [ ] 指数名称全部为 `中证800`；
* [ ] 指数代码全部为字符串 `000906`；
* [ ] 每个因子单独占一列；
* [ ] `symbol + rebalance_date` 不存在重复；
* [ ] 因子列均为数值类型；
* [ ] 因子均处理过极端值
* [ ] 缺失值统一为 `NaN`；
* [ ] 不存在正无穷或负无穷；
* [ ] 年度和季度因子没有提前使用未来数据；
* [ ] 公司名称能够正确匹配；
* [ ] PKL 和 CSV 来自同一个最终 DataFrame；
* [ ] 三位成员均确认自己负责的因子版本。
[SUC (1).csv](https://github.com/user-attachments/files/29498723/SUC.1.csv)

---

## 最终交付

```text
factor_panel.pkl
factor_panel.csv
```

本项目只负责完成不同因子的格式统一、时间对齐和数据合并。后续的因子标准化、中性化、加权、筛选和回测，应在最终对齐数据的基础上单独完成。
