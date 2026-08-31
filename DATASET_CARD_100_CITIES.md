# 100-city OD dataset card / 百城市级 OD 数据说明

更新日期：2026-08-31

## 1. 数据范围

当前 `main` 分支含 100 个美国市级行政单元目录，以及一个历史 `New York/` 州级目录。百城数据包括 97 个 `*_city`、1 个 `*_municipality`、1 个 `*_town` 和 1 个 `*_urban_county`，因此不应统一称为 metropolitan areas。

百城全量共有：

- 100 个行政单元；
- 14,767 个 Census tract；
- 8,139,469 个矩阵 OD 对；
- 3,328,058 个正流 OD 对；
- 每城 27–2,167 个节点，中位数 90。

## 2. 每个城市的文件

### `nodes.csv`

| 列 | 类型 | 说明 |
|---|---|---|
| `node_index` | integer | 从 0 开始，定义 `flow.pkl` 的行列顺序 |
| `ct_code` | string-like | Census tract GEOID；读取时必须按字符串处理 |
| `population` | integer/float | 常住人口 |
| `area_km2` | float | tract 面积，平方千米 |
| `longitude` | float | WGS84 经度 |
| `latitude` | float | WGS84 纬度 |

注意：3,761 个原始 `ct_code` 只有 10 位，原因是数值导出丢失了州 FIPS 的前导零。规范读取方式是字符串化后仅对 10 位纯数字执行 `zfill(11)`，不要保存为整数。

### `flow.pkl`

- Python pickle，内容为 NumPy `float32` 方阵；
- shape 为 `(N, N)`，`N` 等于同目录 `nodes.csv` 行数；
- 行表示 origin，列表示 destination；
- 非负、加权、有向，包含对角线 self-loop，也包含零值；
- 仓库约定行列顺序与 `node_index` 一致。

Pickle 反序列化可能执行代码，只应加载可信来源文件。

## 3. 数据语义与来源状态

Git 历史处理目录名为 `COVID19USFlows_2019_12week_mean_CT_all_OD_lonlat`。原始来源家族指向 [GeoDS COVID19USFlows](https://github.com/GeoDS/COVID19USFlows)，其官方说明为利用 SafeGraph 匿名手机访问轨迹推断的日/周 population mobility flows，覆盖 tract、county 和 state。对应数据论文为 Kang et al., *Scientific Data* 7, 390 (2020)：[论文页面](https://www.nature.com/articles/s41597-020-00734-5)。

因此当前建议称为“人口流动/访问流 OD”，不要称为 LODES 通勤 OD。

下列信息尚未由本仓库中的生成脚本独立证明，使用前须向处理者确认：

1. 2019 年具体哪 12 周；
2. `flow.pkl` 使用上游 `pop_flows`、`visitor_flows`，还是二次缩放结果；
3. 12 周均值、缺失值和零值的处理；
4. 城市边界版本及 tract 纳入规则；
5. `population` 的来源和年份；
6. 自环的业务语义；
7. 衍生数据的再发布许可。

## 4. 审计摘要

- 100 城之间无重复 GEOID（补齐前导零后）；
- 所有矩阵 shape、有限值和非负性通过；
- 逐城正边密度：0.284–0.997，中位数 0.896；
- 全体 OD 对微平均正边密度：0.409；
- 自环流量占比：0.096–0.232，中位数 0.141；
- 前 1% OD 对流量份额：0.102–0.482，中位数 0.214；
- 有向非对称比 `||F-F^T||_1 / ||F||_1`：0.751–1.362。

由于中小城市通常很密，不能把“稀疏网络”当作全数据统一事实；稀疏性应按城市规模分层分析。

## 5. 最小读取示例

```python
import pickle
from pathlib import Path

import numpy as np
import pandas as pd

city_dir = Path("Albuquerque_city")
nodes = pd.read_csv(city_dir / "nodes.csv", dtype={"ct_code": "string"})
nodes["ct_code"] = nodes["ct_code"].str.zfill(11)

# Pickle 只能从可信来源加载。
with (city_dir / "flow.pkl").open("rb") as handle:
    flow = pickle.load(handle)

assert isinstance(flow, np.ndarray)
assert flow.shape == (len(nodes), len(nodes))
assert np.isfinite(flow).all() and (flow >= 0).all()
```

## 6. 与历史纽约州数据的区别

`New York/` 下的 `centroid.pkl`、`population.pkl` 和 `od2flow.pkl` 是旧的州级 dictionary schema；它不属于上述 100 城，不应与百城矩阵直接混用。旧 schema 的详细说明保留在主 README 中。

## 7. 许可证提示

本仓库当前未提供仓库级 `LICENSE` 文件。上游 GeoDS 仓库声明 MIT，不等于自动确认所有衍生数据的再发布权限；公开论文数据或打包下载前应由维护者补充许可证说明。
