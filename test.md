```mermaid
flowchart LR
  %% 外部使用者與入口
  subgraph Users[外部/內部使用者與入口]
    U1[企業員工/客服/管理層/外部客戶]
    CH[多通路介面<br/>Web / LINE / Teams / Slack]
  end

  %% 外部企業系統
  subgraph ES[企業內部系統]
    ERP[ERP]
    CRM[CRM]
    KMS[知識文件庫/檔案庫]
  end

  %% 外部雲端/模型與監控
  subgraph Ext[外部服務]
    LLMCloud[雲端/外部 LLM 供應商<br/>Azure OpenAI / DeepSeek / 其他]
  end

  subgraph Observability[監控與可觀測性]
    ELK[(ELK Stack<br/>Elasticsearch / Logstash / Kibana)]
  end

  %% AI Gateway 核心
  subgraph AIGW[AI Gateway（大展航圖）]
    direction TB

    subgraph SecGov[安全與治理層]
      Guard[LLM 護欄<br/>Prompt/Response 安全審查]
      NER[NER 脫敏/遮罩]
      AuthZ[AuthZ Pre-check<br/>RLS/ACL 權限繼承檢查]
      APIGW[統一 API Gateway<br/>金鑰/路由/限流/LB]
    end

    subgraph Retrieval[智慧檢索層]
      ETL[文件前處理/抽取/切片]
      KG[知識圖譜構建<br/>節點/邊+權限標籤]
      VecDB[(向量索引<br/>PostgreSQL/pgvector or Elasticsearch)]
      GraphDB[(圖資料庫<br/>Apache AGE)]
      GraphRAG[GraphRAG 檢索編排<br/>語意+多跳推理]
    end

    subgraph Dialog[對話與協作層]
      Router[Semantic Router<br/>語意分流/模型選擇/工具派送]
      Cache[Semantic Cache<br/>相似問答命中]
      Agents[AI Agents<br/>A2A/MCP 工具協作]
    end

    subgraph GovObs[治理與可觀測性]
      LB[負載平衡/模型分流]
      Cost[成本追蹤/併發統計]
      Audit[審計/存取日誌]
    end
  end

  %% 通訊與資料流
  U1 --> CH
  CH --> APIGW

  %% Gateway 流入安全治理
  APIGW --> Guard
  Guard --> NER
  NER --> AuthZ

  %% 授權通過後路由
  AuthZ --> Router
  Router --> Cache
  Router --> GraphRAG
  Router --> Agents
  Router --> LLMCloud

  %% 檢索資料面
  GraphRAG --> VecDB
  GraphRAG --> GraphDB

  %% 知識建置流程（離線/批次）
  KMS --> ETL --> KG
  KG --> VecDB
  KG --> GraphDB

  %% 企業系統整合
  ES --- APIGW
  Agents --- ES

  %% 回覆返回路徑（簡化）
  LLMCloud --> Router
  GraphRAG --> Router
  Cache --> Router
  Router --> APIGW --> CH --> U1

  %% 可觀測性與治理
  APIGW -->|日誌/指標| ELK
  Guard -->|審查紀錄| ELK
  AuthZ -->|授權審計| ELK
  Router -->|路由/延遲| ELK
  GraphRAG -->|檢索指標| ELK
  LB -->|系統指標| ELK
  Cost -->|成本數據| ELK
  Audit -->|存取/審計| ELK
```


```mermaid
MK
flowchart LR
  %% 外部實體
  U[使用者群體<br/>員工/客服/管理層/外部客戶]
  CH[通路介面<br/>Web/LINE/Teams/Slack]
  SYS[企業內部系統<br/>ERP/CRM/知識文件庫]
  LLM[外部/雲端 LLM 服務]
  MON[監控與報表平臺<br/>ELK Stack]

  %% 系統（單一處理）
  G[(AI Gateway（大展航圖）)]

  %% 使用者互動
  U -->|"查詢/請求/回饋"| CH
  CH -->|"使用者請求（已連線）"| G
  G -->|"回應/通知/結果"| CH
  CH -->|"回覆展示"| U

  %% 對企業系統的整合
  SYS -->|"企業資料/知識文件/設定"| G
  G -->|"讀取/同步需求/查詢請求"| SYS

  %% 對外部 LLM 的交互
  G -->|"模型請求/推理任務"| LLM
  LLM -->|"推理結果/生成內容"| G

  %% 監控與治理輸出
  G -->|"日誌/指標/成本/審計事件"| MON
```
