# 运行 MINE

运行 MINE 的步骤如下：

1. 在 [`evaluation.py`](evaluation.py) 文件中，将 `"YOUR_OPENAI_KEY"` 替换为你的实际密钥。

2. 使用你的知识图谱（KG）生成器，从 [`essays.json`](essay.json) 中的每篇文章生成一个知识图谱（KG）。生成的知识图谱应为 JSON 文件，其结构需与 [`example.json`](example.json) 相同。

3. 将这些知识图谱按顺序命名为 `1.json`、`2.json`、...、`106.json`，并将它们放置在本目录下的 `KGs/` 文件夹中。

4. 运行 `python evaluation.py`。

5. 在 `KGs/` 文件夹中查找生成的文件 `1_result.json`、...、`106_result.json`。

