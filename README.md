# go-fleet-telemetry




graph TD
    %% Cores e Estilos
    classDef external fill:#f9f9f9,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    classDef primary fill:#d4e6f1,stroke:#2980b9,stroke-width:2px;
    classDef core fill:#d5f5e3,stroke:#27ae60,stroke-width:2px;
    classDef secondary fill:#fcf3cf,stroke:#f39c12,stroke-width:2px;

    %% Dispositivos Externos
    subgraph External [Dispositivos IoT - Carros Turbi]
        S1(Carro 1)
        S2(Carro 2)
        SN(Carro N)
    end

    %% Porta de Entrada
    subgraph PrimaryLayer [Primary Adapters / Inbound]
        API[API REST / Webhook Handler]
    end

    %% Núcleo / Domínio em Go
    subgraph CoreLayer [Core / Application Layer em Go]
        CH((Go Channel<br>Bufferizado))
        WP{Worker Pool<br>Goroutines}
        UseCase[Caso de Uso:<br>Processar Telemetria]
        IdempotencyRule[Regra de Negócio:<br>Garantir Idempotência]
    end

    %% Banco de dados e Cache
    subgraph SecondaryLayer [Secondary Adapters / Outbound]
        CACHE[(Cache Status<br>Redis ou Map em Memória<br>com sync.RWMutex)]
        DB[(Banco de Dados<br>SQLite / PostgreSQL)]
    end

    %% Fluxo de Dados
    S1 & S2 & SN -- "HTTP POST (JSON com message_id)" --> API
    
    API -- "1. Envia Payload" --> CH
    CH -- "2. Consome de forma Concorrente" --> WP
    WP -- "3. Executa" --> UseCase
    UseCase -- "4. Aplica" --> IdempotencyRule

    IdempotencyRule -- "5a. Verifica message_id" --> CACHE
    IdempotencyRule -- "5b. Grava Telemetria<br>(Inicia Transação)" --> DB

    %% Tratamento de Duplicidade
    DB -. "Se falhar por Unique Constraint<br>identifica como duplicado" .-> IdempotencyRule
    
    %% Aplicação de Classes
    class External external;
    class PrimaryLayer primary;
    class CoreLayer core;
    class SecondaryLayer secondary;
