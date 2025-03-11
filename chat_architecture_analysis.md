# Chat子项目架构分析报告

## 1. 系统架构图
```mermaid
graph TD
    subgraph 外部系统
        LLM[[大语言模型]]
        VectorDB[(向量数据库)]
        Cloud[云存储]
        DB[(关系数据库)]
        OLAP[OLAP引擎]
    end

    subgraph Chat服务
        API[API接口层]
        Service[业务服务层]
        Executor[执行引擎]
        DAL[数据访问层]
    end

    API --> Service
    Service --> Executor
    Executor --> DAL
    DAL --> DB
    Executor --> LLM
    Executor --> VectorDB
    DAL --> OLAP
    Service --> Cloud
```

## 2. 核心交互流程（SQL执行）
```mermaid
sequenceDiagram
    participant Client as 客户端
    participant API as REST API
    participant Service as QueryService
    participant Executor as SqlExecutor
    participant LLM as 大模型
    participant DB as 数据库
    
    Client->>API: 提交查询请求
    API->>Service: 解析请求参数
    Service->>Executor: 执行语义解析
    Executor->>LLM: 调用模型生成SQL
    LLM-->>Executor: 返回生成结果
    Executor->>DB: 执行优化后SQL
    DB-->>Executor: 返回数据集
    Executor->>Service: 封装标准化结果
    Service-->>API: 返回响应数据
    API-->>Client: 展示查询结果
```

## 3. 数据库表结构
```mermaid
erDiagram
    QUERY ||--o{ CONTEXT : 包含
    QUERY {
        bigint query_id PK
        varchar query_text
        timestamp create_time
    }
    
    CONTEXT {
        bigint context_id PK
        bigint query_id FK
        json chat_context
        varchar data_schema
    }
    
    MEMORY ||--o{ EXEMPLAR : 包含
    MEMORY {
        bigint memory_id PK
        varchar question
        json side_info
    }
    
    EXEMPLAR {
        bigint exemplar_id PK
        bigint memory_id FK
        text sql_example
    }
```

## 4. 外部系统集成
| 集成系统        | 技术实现                          | 功能用途                   |
|-----------------|-----------------------------------|---------------------------|
| 大语言模型       | LangChain4J + 多模型适配          | SQL生成/结果解释           |
| 向量数据库       | Chroma/Milvus连接器               | 上下文语义存储             |
| 云存储          | AWS S3 SDK                        | 查询结果缓存               |
| OLAP引擎        | Presto/Trino JDBC                 | 跨数据源查询               |

## 5. 核心类关系
```mermaid
classDiagram
    class SqlExecutor{
        +execute(ExecuteContext): QueryResult
        -doExecute(): QueryResult
    }
    
    class SemanticLayerService{
        +queryByReq(QuerySqlReq): SemanticQueryResp
    }
    
    class MemoryService{
        +createMemory(ChatMemory): void
    }
    
    SqlExecutor --> SemanticLayerService
    SqlExecutor --> MemoryService
```

已生成完整架构文档，包含：
- 系统分层架构
- 核心业务流程
- 数据库ER模型
- 外部系统集成矩阵
- 关键类关系图