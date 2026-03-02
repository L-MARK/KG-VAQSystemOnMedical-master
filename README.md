本项目为本人毕设项目，仅供学习，请勿直接抄袭，如有疑问可直接联系作者。详细介绍请访问CSDN：https://blog.csdn.net/jiqiu12/article/details/140017540
其中项目包为KG-VAQSystemOnMedical-master.zip直接下载即可，json数据集较大，爬取自寻医问药网，存放在另一个名为“data”的branch中，不要忘了下载。
数据集下载完毕后直接放置KG-VAQSystemOnMedical-master文件目录下即可。
![image](https://github.com/jiqiu123/KG-VAQSystemOnMedical-master/assets/115466479/dc11f873-1d76-4977-9261-d039730009c5)

运行前，确保Neo4j数据库已安装并配置好环境，python中所需包也安装完毕。环境配置好后，先运行build_medicalgraph.py文件将数据入库并生成知识图谱，导入过程可能较久请耐心等待，如果想加快速度也可以删除一些数据集即可。
成功后，直接运行start.py然后进入网页输入网址：http://127.0.0.1:5000 即可跳转。

主页面：
![image](https://github.com/jiqiu123/KG-VAQSystemOnMedical-master/assets/115466479/d66c1092-4ad3-4e87-bd0e-4b892e10376d)
智能医疗小助手界面：
![image](https://github.com/jiqiu123/KG-VAQSystemOnMedical-master/assets/115466479/ee008da8-9add-4d4a-a55b-5234d1ab560d)
知识图谱可视化界面：
![image](https://github.com/jiqiu123/KG-VAQSystemOnMedical-master/assets/115466479/4e00675f-53d6-456b-ae6c-94c8776b30d4)

## 环境配置需求

### 系统依赖

| 依赖项 | 版本要求 | 说明 |
|--------|----------|------|
| Python | >= 3.6（推荐 3.8+） | 项目使用 Python 3 编写 |
| Neo4j  | >= 3.5（推荐 4.x） | 图数据库，需提前安装并启动 |

### Python 包依赖

所有 Python 依赖均列于项目根目录的 `requirements.txt` 文件中，执行以下命令一键安装：

```bash
pip install -r requirements.txt
```

| 包名 | 版本要求 | 用途 |
|------|----------|------|
| flask | >= 2.0.0 | Web 框架，提供 HTTP 服务 |
| py2neo | == 2021.2.3 | Python 操作 Neo4j 图数据库 |
| pyahocorasick | >= 1.4.0 | 高效关键词匹配（问题分类器） |
| pymongo | >= 3.12.0 | 数据准备脚本使用（可选，仅 prepare_data 目录） |
| lxml | >= 4.6.0 | 数据准备脚本使用（可选，仅 prepare_data 目录） |

### Neo4j 配置

Neo4j 默认连接信息（可在各 `.py` 文件中修改）：

- HTTP 地址：`http://localhost:7474/`
- Bolt 地址：`bolt://localhost:7687`
- 用户名：`neo4j`
- 密码：`<your-neo4j-password>`（请将各源文件中的 `password` 字段替换为您安装 Neo4j 时设置的实际密码）

