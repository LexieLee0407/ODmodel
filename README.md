# ODmodel- data
纽约州普查区级空间交互数据集说明文档
New York State Census Tract Spatial Interaction Dataset (NYS-CT-SID)
1. 数据集概览 (Dataset Overview)
本数据集描述了美国纽约州（New York State）全境在普查区 (Census Tract) 粒度下的人口分布、地理空间位置及区域间的人群流动网络。数据经过预处理和清洗，采用 Python 原生序列化格式 (.pkl) 存储，旨在支持空间分析、城市计算、交通预测及流行病传播模拟等任务。
- 地理范围：美国纽约州全境 (FIPS State Code: 36)，包含纽约市五大区（Manhattan, Brooklyn, Queens, Bronx, Staten Island）及上州地区。
- 空间粒度：Census Tract（人口普查区），这是美国人口普查局定义的用于统计的小型统计分区，通常包含 1,200 至 8,000 人。
- 数据格式：Python Pickle (.pkl) 二进制文件。
- 坐标系：WGS84 (EPSG:4326)。

---
2. 文件清单与详细结构 (File Descriptions)
数据集由三个核心文件组成，共同构建了一个属性图网络 $G = (V, E, A)$，其中 $$V$$ 为节点集合，$E$ 为边集合，$A$ 为节点属性。
2.1. centroid.pkl (地理中心点数据)
- 定义：定义了网络节点的几何位置。
- 数据结构：Dictionary {String: List[Float]}
- Key (键)：11位 FIPS 代码 (字符串)。唯一标识一个普查区。
- Value (值)：[Longitude, Latitude] (浮点数列表)。
  - 索引 0：经度 (Longitude)，范围约为 -79.7° 至 -71.8°。
  - 索引 1：纬度 (Latitude)，范围约为 40.5° 至 45.0°。
- 技术说明：该坐标代表普查区的几何中心（Centroid），用于在可视化中定位节点或计算节点间的欧几里得/哈瑟辛距离。
2.2. population.pkl (人口属性数据)
- 定义：定义了网络节点的权重属性（节点质量）。
- 数据结构：Dictionary {String: Integer}
- Key (键)：11位 FIPS 代码 (与 centroid.pkl 严格对应)。
- Value (值)：常住人口数量 (整数)。
- 技术说明：
  - 该数据通常源自 ACS (American Community Survey) 或 Decennial Census。
该数据通常源自 ACS（American Community Survey） 或 Decennial Census。
  - 注意：部分区域（如公园、机场、工业区）人口可能为 0，在使用该数据作为分母计算人均指标时需进行异常值处理。
2.3. od2flow.pkl (OD 流量矩阵数据)
- 定义：定义了网络的边及其权重（交互强度）。
- 数据结构：Dictionary {Tuple(String, String): Float/Int}
- Key (键)：(Origin_ID, Destination_ID) 元组。
  - Origin_ID：出发地普查区 FIPS 代码。
  - Destination_ID：目的地普查区 FIPS 代码。
- Value (值)：流量 (Flow)。
  - 代表特定时间窗口内（如日均通勤、全天出行总量）从 O 点移动到 D 点的人次或概率权重。
- 拓扑特征：
  - 稀疏性：并非所有节点两两之间都有流量，仅记录存在交互的边。
  - 自环 (Self-loops)：存在 Origin == Destination 的键，代表区域内部的流动（如居家办公、社区内短途出行）。
  - 有向性：图是有向图，即 (A, B) 的流量不一定等于 (B, A) 的流量。

---
3. 数据编码规范 (Data Dictionary)
理解 FIPS 代码 (Federal Information Processing Standards) 是使用本数据集的关键。所有的 Key 均遵循 11 位数字字符串格式 SSCCCTTTTTT：
暂时无法在飞书文档外展示此内容
常用纽约市 (NYC) 县代码对照表：
- 061: New York County (曼哈顿 Manhattan)
061：纽约县（曼哈顿 Manhattan）
- 047: Kings County (布鲁克林 Brooklyn)
047：布鲁克林（布鲁克林 Brooklyn）
- 081: Queens County (皇后区 Queens)
081：皇后区（皇后区 Queens）
- 005: Bronx County (布朗克斯 The Bronx)
005：布朗克斯县（布朗克斯 The Bronx）
- 085: Richmond County (史泰登岛 Staten Island)
085：里士满县（史泰登岛 Staten Island）
其他常见代码：
- 001: Albany County (州首府周边)
- 029: Erie County (布法罗周边)

---
4. 逻辑架构与关系模型 (Logical Schema)
该数据集构建了一个典型的空间交互网络 (Spatial Interaction Network)。
1. 节点一致性：centroid.pkl 和 population.pkl 的 Key 集合应当是高度重合的（通常是一一对应），构成了图的节点集合 $V$。
2. 边的一致性：od2flow.pkl 中的所有 Origin 和 Destination ID 都必须存在于 centroid.pkl 的 Key 集合中（除了极少数可能因版本差异导致的孤立点）。
3. 网络模型：
  - 节点 (Node): Census Tract (由 centroid 定位，由 population 加权)。
  - 边 (Edge): Human Mobility (由 od2flow 定义权重)。

---
5. 测试数据集数据结构
提取了一份包含 5 个节点（普查区）的小型测试数据集。这份数据保持了与源文件完全一致的数据结构，并且直接来源于真实的纽约州数据（包含曼哈顿区域以及周边存在交互的区域）。
5.1.1 Centroid Data (中心点坐标)
数据含义：普查区ID映射到 [经度, 纬度]（注意：根据源数据，这里是经度在前，纬度在后，或者反之，具体请参照你模型对经纬度的处理。原始数据中看起来是 [-77.xxx, 43.xxx]，即 [Lon, Lat]）。
Python  蟒蛇
# test_centroid_data
centroid = {
    '36061000100': [-74.0461708, 40.6900008],       # 曼哈顿 (自由女神像附近区域)'36055011703': [-77.4008057, 43.1028109],       # 门罗县 (Monroe County)'36069050101': [-77.4582906, 43.0059497],       # 安大略县 (Ontario County)'36055014003': [-77.68674509999998, 43.216090200000004], # 门罗县 (Monroe County)'36075021602': [-76.5295124, 43.4510055]        # 奥斯威戈县 (Oswego County)
}
5.1.2 Population Data (人口数据)
数据含义：普查区ID映射到常住人口数量。
Python  蟒蛇
# test_population_data
population = {
    '36061000100': 0,     # 某些商业区或特定区域（如公园/地标）可能常住人口极少或为0'36055011703': 4251,
    '36069050101': 3298,
    '36055014003': 4641,
    '36075021602': 2821
}
5.1.3 OD to Flow Data (OD流量数据)
数据含义：(起点ID, 终点ID) 元组映射到流量数值。
- 注意：这里只保留了起点和终点都在上述 5 个节点内的流量数据，形成了一个闭环网络。
Python  蟒蛇
# test_od2flow_data
od2flow = {
    ('36055011703', '36061000100'): 14.0,   # 从 门罗县 到 曼哈顿
    ('36055011703', '36069050101'): 381.0,  # 门罗县 -> 安大略县 (邻近交互强)
    ('36055011703', '36055014003'): 29.0,
    ('36055011703', '36075021602'): 14.0,
    ('36055011703', '36055011703'): 6787.0, # 区域内部流动 (Self-loop)
    ('36055014003', '36069050101'): 34.0,
    ('36055014003', '36055014003'): 4008.0,
    ('36055014003', '36055011703'): 34.0,
    ('36069050101', '36069050101'): 2999.0,
    ('36069050101', '36055011703'): 35.0,
    ('36075021602', '36075021602'): 3324.0
}

