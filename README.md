kg-gen: 从任意文本生成知识图谱

欢迎使用！`kg-gen` 帮助您通过 AI 从任何纯文本中提取知识图谱。它可以处理小型和大型文本输入，也可以处理对话格式的消息。

为什么要生成知识图谱？`kg-gen` 非常适合以下场景：

- 创建用于 RAG（检索增强生成）的图谱
- 为模型[训练]和[测试]生成合成数据图
- 将任何文本结构化为图谱
- 分析源文本中概念之间的关系

我们通过 [LiteLLM](https://docs.litellm.ai/docs/providers) 支持基于 API 和本地模型的提供商，包括 OpenAI、Ollama、Anthropic、Gemini、Deepseek 等。我们还使用 [DSPy](https://dspy.ai/) 生成结构化输出。

- 通过运行 [`tests/`](https://github.com/stair-lab/kg-gen/tree/main/tests) 中的脚本试用。
- 在 [`MINE/`](https://github.com/stair-lab/kg-gen/tree/main/MINE) 中查看运行我们 KG 基准测试 MINE 的说明。
- 阅读论文：[KGGen: Extracting Knowledge Graphs from Plain Text with Language Models](https://arxiv.org/abs/2502.09956)

## 1. 支持的模型

传入一个 `model` 字符串即可使用您选择的模型。模型调用通过 LiteLLM 路由，通常 LiteLLM 的格式为 `{model_provider}/{model_name}`。具体格式请参见 [https://docs.litellm.ai/docs/providers](https://docs.litellm.ai/docs/providers)。

以下是一些可以传入的模型示例：

- `openai/gpt-4o`
- `gemini/gemini-2.0-flash`
- `ollama_chat/deepseek-r1:14b`

## 2. 快速开始

安装模块：

```bash
pip install kg-gen
```

然后导入并使用 `kg-gen`。您可以以以下两种格式提供文本输入：

1. 单个字符串  
2. 消息对象列表（每个对象包含角色和内容）

以下是一些示例代码片段：

```python
from kg_gen import KGGen

# 使用可选配置初始化 KGGen
kg = KGGen(
  model       ="openai/gpt-4o",  # 默认模型
  temperature =0.0,              # 默认温度
  api_key     ="YOUR_API_KEY"    # 如果已在环境中设置或使用本地模型，则可选
)

# 示例 01：带上下文的单个字符串
text_input = "Linda 是 Josh 的母亲。Ben 是 Josh 的兄弟。Andrew 是 Josh 的父亲。"
graph_01 = kg.generate(
  input_data=text_input,
  context="家庭关系"
)

# 输出: 
# entities={'Linda', 'Ben', 'Andrew', 'Josh'} 
# edges={'是兄弟', '是父亲', '是母亲'} 
# relations={('Ben', '是兄弟', 'Josh'), 
#           ('Andrew', '是父亲', 'Josh'), 
#           ('Linda', '是母亲', 'Josh')}
```

## 3. 更多示例 

- 分块
- 聚类、
- 传入消息数组

```python
# 示例 02：带分块和聚类的大文本
with open('large_text.txt', 'r') as f:
  large_text = f.read()
  
# 示例输入文本:
# """
# 神经网络是一种机器学习模型。深度学习是机器学习的一个子集，
# 使用多层神经网络。监督学习需要训练数据来学习模式。
# 机器学习是一种 AI 技术，使计算机能够从数据中学习。
# AI，也称为人工智能，与更广泛的人工智能领域相关。
# 神经网络（NN）通常用于 ML 应用。机器学习（ML）已彻底改变了
# 许多研究领域。
# ...
# """

graph_02 = kg.generate(
  input_data=large_text,
  chunk_size=5000,  # 按 5000 字符分块处理文本
  cluster=True      # 聚类相似的实体和关系
)

# 输出:
# entities={'神经网络', '深度学习', '机器学习', 'AI', '人工智能','监督学习', '无监督学习', '训练数据', ...} 

# edges={'是一种', '需要', '是子集', '使用', '与...相关', ...} 

# relations={('神经网络', '是一种', '机器学习'),
#           ('深度学习', '是子集', '机器学习'),
#           ('监督学习', '需要', '训练数据'),
#           ('机器学习', '是一种', 'AI'),
#           ('AI', '与...相关', '人工智能'), ...}

# entity_clusters={
#   '人工智能': {'AI', '人工智能'},
#   '机器学习': {'机器学习', 'ML'},
#   '神经网络': {'神经网络', '神经网', 'NN'}
#   ...
# }

# edge_clusters={
#   '是一种': {'是一种', '是一类', '是一种类型'},
#   '与...相关': {'与...相关', '与...连接', '与...关联'}
#  ...}
# }

# 示例 03：消息数组
messages = [
  {"role": "user", "content": "法国的首都是哪里？"}, 
  {"role": "assistant", "content": "法国的首都是巴黎。"}
]
graph_3 = kg.generate(input_data=messages)
# 输出: 
# entities={'巴黎', '法国'} 
# edges={'有首都'} 
# relations={('法国', '有首都', '巴黎')}

# 示例 04：合并多个图谱
text1 = "Linda 是 Joe 的母亲。Ben 是 Joe 的兄弟。"
# 输入文本 02: 也叫 Joe。

text2 = "Andrew 是 Joseph 的父亲。Judy 是 Andrew 的姐姐。Joseph 也叫 Joe。"

graph4_a = kg.generate(input_data=text1)
graph4_b = kg.generate(input_data=text2)

# 合并图谱
combined_graph = kg.aggregate([graph4_a, graph4_b])

# 可选：对合并后的图谱进行聚类
clustered_graph = kg.cluster(
  combined_graph,
  context="家庭关系"
)

# 输出:
# entities={'Linda', 'Ben', 'Andrew', 'Joe', 'Joseph', 'Judy'} 
# edges={'是母亲', '是父亲', '是兄弟', '是姐姐'} 
# relations={('Linda', '是母亲', 'Joe'),
#           ('Ben', '是兄弟', 'Joe'),
#           ('Andrew', '是父亲', 'Joe'),
#           ('Judy', '是姐姐', 'Andrew')}
# entity_clusters={
#   'Joe': {'Joe', 'Joseph'},
#   ...
# }
# edge_clusters={ ... }
```

## 4. 功能

### 4.1. 分块大文本

对于大文本，可以指定 `chunk_size` 参数，将文本分成较小的块进行处理：

```python
graph = kg.generate(
  input_data=large_text,
  chunk_size=5000  # 按 5000 字符分块处理
)
```

### 4.2. 聚类相似的实体和关系

可以在生成过程中或之后聚类相似的实体和关系：

```python
# 在生成过程中
graph = kg.generate(
  input_data =text,
  cluster    =True,
  context    ="可选上下文以指导聚类"
)

# 或在生成之后
clustered_graph = kg.cluster(
  graph,
  context="可选上下文以指导聚类"
)
```

### 4.3. 合并多个图谱

您可以使用 aggregate 方法合并多个图谱：

```python
graph1 = kg.generate(input_data=text1)
graph2 = kg.generate(input_data=text2)
combined_graph = kg.aggregate([graph1, graph2])
```

### 4.4. 消息数组处理

在处理消息数组时，kg-gen：

1. 保留每条消息的角色信息
2. 保持消息顺序和边界
3. 可以提取以下实体和关系：
   - 消息中提到的概念之间
   - 角色（说话者）与概念之间
   - 对话中多条消息之间

例如，给定以下对话：

```python
messages = [
  {"role": "user", "content": "法国的首都是哪里？"},
  {"role": "assistant", "content": "法国的首都是巴黎。"}
]
```

生成的图谱可能包括以下实体：

- "user"
- "assistant" 
- "法国"
- "巴黎"

以及以下关系：

- (user, "询问", "法国")
- (assistant, "回答", "巴黎")
- (巴黎, "是首都", "法国")

## 5. API 参考

### 5.1. KGGen 类

#### 5.1.1. 构造函数参数

- `model`: str = "openai/gpt-4o" - 要使用的生成模型
- `temperature`: float = 0.0 - 模型采样的温度
- `api_key`: Optional[str] = None - 模型访问的 API 密钥

#### 5.1.2. generate() 方法参数

- `input_data`: Union[str, List[Dict]] - 文本字符串或消息字典列表
- `model`: Optional[str] - 覆盖默认模型
- `api_key`: Optional[str] - 覆盖默认 API 密钥
- `context`: str = "" - 数据上下文的描述
- `chunk_size`: Optional[int] - 要处理的文本块大小
- `cluster`: bool = False - 是否在生成后聚类图谱
- `temperature`: Optional[float] - 覆盖默认温度
- `output_folder`: Optional[str] - 保存部分进度的路径

#### 5.1.3. cluster() 方法参数

- `graph`: Graph - 要聚类的图谱
- `context`: str = "" - 数据上下文的描述
- `model`: Optional[str] - 覆盖默认模型
- `temperature`: Optional[float] - 覆盖默认温度
- `api_key`: Optional[str] - 覆盖默认 API 密钥

#### 5.1.4. aggregate() 方法参数

- `graphs`: List[Graph] - 要合并的图谱列表

## 6. 许可证

MIT 许可证。
