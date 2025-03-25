- 对于 NumPy，你应该能够熟练地创建、操作数组，进行基本的数学运算，理解广播机制。
- 对于 Pandas，你应该能够独立地创建和操作 Series 和 DataFrame，进行简单的数据清洗和处理。

jupter用一个新的虚拟环境

conda create --name <虚拟环境名称> python=3.8

conda activate <虚拟环境名称>

pip install ipykernel

python -m ipykernel install --user --name=pytorch

常用，注意，jupte打开时默认用的是base，需要收到activate要的虚拟环境

conda env list

conda list 当前环境的所有包

conda env create -f environment.yml，创建一个 Conda 环境，环境的配置信息存储在一个名为 `environment.yml` 的文件中。（安装指定的软件包和它们的版本。）

**新clone一个项目**

1.创建虚拟环境 conda create -n myenv python=3.8 

 2.激活虚拟环境 conda activate myenv

3.cd 路径

np.arange(9.)这个`9.`是用来表示数字类型的。

# **gpt**

embedding：嵌入：把某种对象映射到一个低维空间，如词嵌入是将词语映射到实数向量空间的技术

chain：**上一个步骤的输出是下一个步骤的输入**

**项目目标**：基于个人知识库的问答助手

**核心功能**

1. 上传文档、创建知识库；
2. 选择知识库，检索用户提问的知识片段；
3. 提供知识片段与提问，获取大模型回答；
4. 流式回复；
5. 历史对话记录

加载本地文档 -> 读取文本 -> 文本分割 -> 文本向量化 -> question向量化 -> 在文本向量中匹配出与问句向量最相似的 top k个 -> 匹配出的文本作为上下文和问题一起添加到 prompt中 -> 提交给 LLM生成回答

1.数据库构建，用工具读取pdf，doc等文本，将较长的文本切分为较小的文本

2.对分割后的文档进行向量化，使语义相似的文本片段具有接近的向量表示。然后，存入向量数据库，这个流程正是创建 `索引(index)` 的过程这样，当用户提出问题时，可以先将问题转换为向量，在数据库中快速找到语义最相关的文档片段。然后将这些文档片段与问题一起传递给语言模型，生成回答。

3.Langchain 中文本分割器都根据 `chunk_size` (块大小)和 `chunk_overlap` (块与块之间的重叠大小)进行分割

- chunk_size 指每个块包含的字符或 Token（如单词、句子等）的数量
- chunk_overlap 指两个块之间共享的字符数量，用于保持上下文的连贯性，避免分割丢失上下文信息