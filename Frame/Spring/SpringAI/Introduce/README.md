# SpringAI 概述

Spring AI is an application framework for AI engineering. Its goal is to apply to the AI domain Spring ecosystem design principles such as portability and modular design and promote using POJOs as the building blocks of an application to the AI domain.

Spring AI是一个AI工程领域的应用程序框架；它的目标是将Spring生态系统的设计原则应用于人工智能领域，比如Spring生态系统的可移植性和模块化设计，并推广使用POJO来构建人工智能领域应用程序。

**Spring AI并不是要构建一个自己的AI大模型，而是对接各种AI大模型**。

官网：https://spring.io/

## Spring AI 特点

Spring AI提供的API支持跨人工智能提供商的 **聊天，文本到图像，和嵌入模型**等，同时支持同步和流API选项

1. **Chat Models 聊天模型**：

   - OpenAI

   - Azure Open AI

   - Amazon Bedrock

   - Cohere's Command

   - AI21 Labs' Jurassic-2

   - Meta's LLama 2

   - Amazon's Titan

   - Google Vertex AI Palm

   - Google Gemini

   - HuggingFace - access thousands of models, including those from Meta such as Llama2

   - Ollama - run AI models on your local machine

   - MistralAI

2. **Text-to-image Models 文本到图像模型**：

   - OpenAI with DALL-E

   - StabilityAI

3. **Transcription (audio to text) Models 转录 (音频到文本) 模型**：
   - OpenAI

4. **Embedding Models 嵌入模型**：

   - OpenAI

   - Azure OpenAI

   - Ollama

   - ONNX

   - PostgresML

   - Bedrock Cohere

   - Bedrock Titan

   - Google VertexAI

   - Mistal AI

5. **Vector Store API**提供了跨不同提供商的可移植性，其特点是提供了一种新颖的类似SQL的元数据过滤API，以保持可移植性

   矢量数据库：

   - Azure Vector Search

   - Chroma

   - Milvus

   - Neo4j

   - PostgreSQL/PGVector

   - PineCone

   - Redis

   - Weaviate

   - Qdrant

6. **用于AI模型和矢量存储的Spring Boot自动配置和启动器** (xxxx-spring-ai-starter)

7. **函数调用**，可以声明java.util.Function的OpenAI模型的函数实现，用于其提示响应。如果在应用程序上下文中注册为@Bean，则可以直接将这些函数作为对象提供，或者引用它们的名称。这一功能最大限度地减少了不必要的代码，并使人工智能模型能够要求更多信息来完成其响应；支持的模型有：

   - OpenAI
   - Azure OpenAI
   - VertexAI
   - Mistral AI

8. **用于数据工程的ETL框架**

   ETL框架的核心功能是使用Vector Store促进文档向模型提供者的传输。ETL框架基于Java函数式编程概念，可帮助您将多个步骤链接在一起；支持阅读各种格式的文档，包括PDF、JSON等；该框架允许数据操作以满足您的需求。这通常包括拆分文档以遵守上下文窗口限制，并使用关键字增强它们以提高文档检索效率；最后，处理后的文档存储在矢量数据库中，以便将来检索；

9. **广泛的参考文档、示例应用程序和研讨会/课程材料**

   未来的版本将在此基础上提供对其他人工智能模型的访问，例如，谷歌刚刚发布的Gemini多模式模态，一个评估人工智能应用程序有效性的框架，更方便的API，以及帮助解决“查询/汇总我的文档”用例的功能。有关即将发布的版本的详细信息，请查看GitHub