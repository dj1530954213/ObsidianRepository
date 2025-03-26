```
graph TD
    %% 主要模块
    A[tokio-modbus] --> B[lib.rs]
    B --> C[client]
    B --> D[server]
    B --> E[frame]
    B --> F[codec]
    B --> G[service]
    B --> H[error.rs]
    B --> I[slave.rs]
    B --> J[prelude.rs]

    %% client模块细分
    C --> C1[client/mod.rs]
    C --> C2[client/tcp.rs]
    C --> C3[client/rtu.rs]
    C --> C4[client/sync]
    C4 --> C4A[client/sync/tcp.rs]
    C4 --> C4B[client/sync/rtu.rs]
    C4 --> C4C[client/sync/mod.rs]

    %% server模块细分
    D --> D1[server/mod.rs]
    D --> D2[server/tcp.rs]
    D --> D3[server/rtu.rs]
    D --> D4[server/rtu_over_tcp.rs]
    D --> D5[server/service.rs]

    %% frame模块细分
    E --> E1[frame/mod.rs]
    E --> E2[frame/tcp.rs]
    E --> E3[frame/rtu.rs]

    %% codec模块细分
    F --> F1[codec/mod.rs]
    F --> F2[codec/tcp.rs]
    F --> F3[codec/rtu.rs]

    %% service模块细分
    G --> G1[service/mod.rs]
    G --> G2[service/tcp.rs]
    G --> G3[service/rtu.rs]

    %% 依赖关系
    A -.->|依赖| K[tokio]
    A -.->|依赖| L[async-trait]
    A -.->|依赖| M[bytes]

    %% 接口和核心类型
    C1 -->|定义| N[Client trait]
    C1 -->|定义| O[Reader trait]
    C1 -->|定义| P[Writer trait]
    C1 -->|定义| Q[Context struct]

    E1 -->|定义| R[Request enum]
    E1 -->|定义| S[Response enum]
    E1 -->|定义| T[FunctionCode enum]
    E1 -->|定义| U[ExceptionCode enum]

    %% 功能实现
    C2 -->|实现| V[TCP客户端]
    C3 -->|实现| W[RTU客户端]
    D2 -->|实现| X[TCP服务器]
    D3 -->|实现| Y[RTU服务器]
    D4 -->|实现| Z[RTU over TCP服务器]

    %% 数据流向
    V -->|使用| F2
    W -->|使用| F3
    X -->|使用| F2
    Y -->|使用| F3
    Z -->|使用| F2
    Z -->|使用| F3

    %% 特征集合
    subgraph 特征系统
        AA[默认特征]
        AB[rtu特征]
        AC[tcp特征]
        AD[sync特征]
        AE[rtu-server特征]
        AF[tcp-server特征]
        AG[rtu-over-tcp-server特征]
    end

    %% 特征影响
    AB -->|启用| C3
    AB -->|启用| E3
    AB -->|启用| F3
    AC -->|启用| C2
    AC -->|启用| E2
    AC -->|启用| F2
    AD -->|启用| C4
    AE -->|启用| D3
    AF -->|启用| D2
    AG -->|启用| D4

    %% 类型系统
    subgraph 核心类型
        BA[Address type]
        BB[Coil type]
        BC[Word type]
        BD[Quantity type]
        BE[Result type]
    end

    %% 实现详情
    N -->|抽象| O
    N -->|抽象| P
    Q -->|实现| N
    Q -->|实现| O
    Q -->|实现| P

    %% 客户端服务器通信
    subgraph 通信流程
        CA[客户端请求]
        CB[编码请求]
        CC[网络传输]
        CD[服务器接收]
        CE[解码请求]
        CF[处理请求]
        CG[生成响应]
        CH[编码响应]
        CI[网络返回]
        CJ[客户端接收]
        CK[解码响应]
        CL[处理响应]
    end

    CA --> CB --> CC --> CD --> CE --> CF --> CG --> CH --> CI --> CJ --> CK --> CL

    %% 样式
    classDef module fill:#f9f,stroke:#333,stroke-width:2px;
    classDef trait fill:#bbf,stroke:#333,stroke-width:1px;
    classDef implementation fill:#bfb,stroke:#333,stroke-width:1px;
    classDef type fill:#fbb,stroke:#333,stroke-width:1px;
    classDef flow fill:#ddd,stroke:#333,stroke-width:1px;

    class A,B,C,D,E,F,G,H,I,J module;
    class N,O,P trait;
    class V,W,X,Y,Z implementation;
    class BA,BB,BC,BD,BE,R,S,T,U type;
    class CA,CB,CC,CD,CE,CF,CG,CH,CI,CJ,CK,CL flow;
```