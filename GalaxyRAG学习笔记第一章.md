# RAG技术学习心得第一章（LangChain+LlamaIndex实践）

## 一、学习路径与核心收获
本次学习聚焦RAG（检索增强生成）的落地实现，核心流程分为两步：
1. **LangChain实现**：手动拼接「文档加载→文本分块→嵌入向量化→向量索引→检索→LLM生成」全流程，理解了RAG的模块化逻辑；
2. **LlamaIndex简化**：通过高封装API，将LangChain的多步操作压缩为「全局配置→文档加载→索引构建→查询」4步，体会到低代码框架的效率优势。


## 二、典型Bug与解决方法
| 错误现象 | 原因 | 解决步骤 |
|----------|------|----------|
| `FileNotFoundError`（文档加载失败） | 相对路径基准目录错误 | 切换到`code/C1`目录执行代码，或使用绝对路径 |
| `NameError: 'TextLoader'未定义` | 未导入对应模块 | 先执行`from langchain_community.document_loaders import TextLoader` |
| `openai.AuthenticationError: 401` | API密钥无效/接口不兼容 | 替换为DeepSeek官方接口`https://api.deepseek.com/v1`，验证密钥有效性 |
| `SyntaxError`（多行代码错误） | 手动输入破坏代码块格式 | 完整输入函数调用代码块，保持统一缩进 |


## 三、个人思考
- LangChain的灵活性适合定制化RAG场景，但上手成本高；LlamaIndex的封装更适合新手快速落地，两者可根据项目复杂度选择；
- 虚拟环境（如`all-in-rag`）是隔离依赖的关键，避免了不同项目的包版本冲突。


## 四、LlamaIndex实践细节与体验
相比LangChain的“分步拼接”，LlamaIndex的核心优势是**“配置化落地”**：
1. **全局设置（Settings）**：一行代码统一配置LLM和嵌入模型，无需像LangChain那样重复初始化组件；
2. **自动分块**：`SimpleDirectoryReader`加载文档后，`VectorStoreIndex.from_documents`会自动完成文本分块（默认保留语义结构），省去了手动调用`RecursiveCharacterTextSplitter`的步骤；
3. **查询引擎封装**：`as_query_engine()`直接生成可调用的查询工具，内置了“问题向量化→相似性检索→上下文拼接”逻辑，最终仅需`query()`就能得到结果。

实际运行时，LlamaIndex的提示词模板（通过`get_prompts()`查看）会约束LLM“仅用上下文回答、不引用上下文本身”，这是避免回答“幻觉”的关键设计。


## 五、新增Bug与解决记录
| 错误现象 | 原因 | 解决步骤 |
|----------|------|----------|
| `ModuleNotFoundError: No module named 'llama_index'` | 依赖未安装到当前虚拟环境 | 在`all-in-rag`环境下执行`pip install llama-index` |
| 嵌入模型下载慢 | 未配置国内镜像 | 临时设置环境变量`os.environ['HF_ENDPOINT']='https://hf-mirror.com'` |
| 回答内容为空 | 检索到的上下文与问题不匹配 | 调整`similarity_search`的`k`值（从3改为5），增加检索的文本块数量 |


## 六、技术延伸思考
1. **生产环境适配**：
   - 本次用的`InMemoryVectorStore`是内存索引，重启程序会丢失；生产环境需替换为Pinecone/FAISS等持久化向量库；
   - LlamaIndex支持`StorageContext`配置持久化存储，可通过`persist()`保存索引到本地。
2. **性能优化**：
   - 文本分块的`chunk_size`需根据模型上下文长度调整（比如DeepSeek的上下文是8k，分块大小可设为2k）；
   - 嵌入模型可替换为更轻量的`bge-small-zh`，平衡速度与效果。


## 七、总结
本次RAG学习的核心是**“从理解原理到落地工具”**：
- 先通过LangChain拆分RAG的每个环节，搞懂“文本→向量→检索→生成”的底层逻辑；
- 再用LlamaIndex体验低代码工具的效率，理解封装框架的设计思路；
- 记录Bug和解决方法，既加深了对技术细节的理解，也为后续项目踩坑做了储备。