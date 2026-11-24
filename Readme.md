# FabLab: Model Storming (Diagrams as Code)

Modelagem do **Sistema FabLab**: reservas de máquinas, fila de prioridade, manutenção, estoque de insumos e compras.

> 📌 Decisões de modelagem e pendências: [`docs/DECISOES-E-PENDENCIAS.md`](docs/DECISOES-E-PENDENCIAS.md)

## Arquivos

| Diagrama | Arquivo |
|---|---|
| Contexto (C4 Nível 1) | [`c4-contexto-visual.mmd`](docs/architecture/c4-contexto-visual.mmd) · notação C4 oficial: [`c4-contexto.mmd`](docs/architecture/c4-contexto.mmd) |
| Contêineres (C4 Nível 2) | [`c4-containers-visual.mmd`](docs/architecture/c4-containers-visual.mmd) · notação C4 oficial: [`c4-containers.mmd`](docs/architecture/c4-containers.mmd) |
| Sequência: reserva com cenários de falha | [`sequencia-reserva.mmd`](docs/architecture/sequencia-reserva.mmd) |
| Sequência: compra de insumos | [`sequencia-compra.mmd`](docs/architecture/sequencia-compra.mmd) |
| Sequência: manutenção de máquina | [`sequencia-manutencao.mmd`](docs/architecture/sequencia-manutencao.mmd) |
| Classes: interfaces e injeção de dependência | [`classes-interfaces.mmd`](docs/architecture/classes-interfaces.mmd) |
| Modelo de domínio | [`modelo-dominio.mmd`](docs/architecture/modelo-dominio.mmd) |
| Componentes da API Backend (C4 Nível 3) | [`componentes.mmd`](docs/architecture/componentes.mmd) |
| Atividades: reserva | [`atividade-reserva.mmd`](docs/architecture/atividade-reserva.mmd) |
| Atividades: cancelamento | [`atividade-cancelamento.mmd`](docs/architecture/atividade-cancelamento.mmd) |
| Estados da Reserva | [`estados-reserva.mmd`](docs/architecture/estados-reserva.mmd) |
| Estados do Pedido de Compra | [`estados-pedido-compra.mmd`](docs/architecture/estados-pedido-compra.mmd) |
| ADR 0001: RabbitMQ + Outbox | [`docs/adr/0001-escolha-tecnologica.md`](docs/adr/0001-escolha-tecnologica.md) |
| Casos de uso | [`docs/architecture/casos-de-uso.pdf`](docs/architecture/casos-de-uso.pdf) |
| Visão de Domínio e Backlog | [`docs/visao-dominio-e-backlog.pdf`](docs/visao-dominio-e-backlog.pdf) |
| Plano de Chaos Engineering | [`docs/architecture/chaos_engineering.pdf`](docs/architecture/chaos_engineering.pdf) |
| Validação cross-diagramas | [`docs/architecture/validacao_cross_diagramas_fablab_dia27.pdf`](docs/architecture/validacao_cross_diagramas_fablab_dia27.pdf) |

Os arquivos `.mmd` são a fonte oficial. Os blocos abaixo são cópias deles para visualização no GitHub.

## 1. Contexto (C4 Nível 1)

Azul-escuro: pessoas. Azul: o Sistema FabLab. Cinza: sistemas externos.

```mermaid
flowchart TB
    classDef pessoa fill:#08427b,stroke:#052e56,color:#fff
    classDef sistema fill:#1168bd,stroke:#0b4884,color:#fff
    classDef externo fill:#8c8c8c,stroke:#6b6b6b,color:#fff

    sol["<b>Solicitante</b><br/><small>[Pessoa]</small><br/>Aluno ou projeto industrial<br/>que reserva máquinas"]:::pessoa
    adm["<b>Administrador/Técnico</b><br/><small>[Pessoa]</small><br/>Aprova reservas e cuida<br/>das máquinas e do estoque"]:::pessoa

    fab["<b>Sistema FabLab</b><br/><small>[Sistema de software]</small><br/>Reservas, fila de prioridade,<br/>manutenção, estoque e compras"]:::sistema

    login["<b>Sistema de Login</b><br/><small>[Sistema externo]</small>"]:::externo
    notif["<b>Serviço de Notificação</b><br/><small>[Sistema externo]</small>"]:::externo
    fin["<b>Sistema Financeiro</b><br/><small>[Sistema externo]</small>"]:::externo
    forn["<b>Portal do Fornecedor</b><br/><small>[Sistema externo]</small>"]:::externo

    sol -- "Reserva máquinas,<br/>acompanha a fila" --> fab
    adm -- "Aprova reservas,<br/>gerencia estoque" --> fab

    fab -- "Autentica usuários" --> login
    fab -- "Envia e-mails" --> notif
    fab <-- "Solicitação e<br/>aprovação de compra" --> fin
    fab <-- "Pedido e confirmação<br/>de entrega" --> forn
```

<details>
<summary>Versão na notação C4 oficial do Mermaid (<code>c4-contexto.mmd</code>)</summary>

O layout automático do C4 no Mermaid é limitado e cruza algumas setas. O conteúdo é o mesmo da versão acima.

```mermaid
C4Context
    title Contexto (C4 Nível 1) - Sistema FabLab

    Person(solicitante, "Solicitante", "Reserva máquinas")
    Person(admin, "Administrador/Técnico", "Aprova reservas, gerencia estoque")
    System(fablab, "Sistema FabLab", "Reservas, fila, manutenção, estoque e compras")
    System_Ext(login, "Sistema de Login", "Autenticação")
    System_Ext(notificacao, "Serviço de Notificação", "E-mail")
    System_Ext(financeiro, "Sistema Financeiro", "Aprova compras")
    System_Ext(fornecedor, "Portal do Fornecedor", "Recebe pedidos")

    Rel(solicitante, fablab, "Usa")
    Rel(admin, fablab, "Usa")
    Rel(fablab, login, "Autentica")
    Rel(fablab, notificacao, "E-mails")
    BiRel(fablab, financeiro, "Compras")
    BiRel(fablab, fornecedor, "Pedidos")
```

</details>

## 2. Contêineres (C4 Nível 2)

Passo 1 do laboratório. Azul: contêineres do Sistema FabLab. Cinza: sistemas externos. Análise em [`auditoria-c4-containers.md`](docs/architecture/auditoria-c4-containers.md).

```mermaid
flowchart LR
    %% Azul = contêineres do Sistema FabLab | Cinza = sistemas externos
    classDef pessoa fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef externo fill:#8c8c8c,stroke:#6b6b6b,color:#fff

    sol["<b>Solicitante</b><br/><small>[Pessoa]</small>"]:::pessoa
    adm["<b>Administrador/Técnico</b><br/><small>[Pessoa]</small>"]:::pessoa

    web["<b>Aplicação Web</b><br/><small>[Contêiner: SPA]</small><br/>Telas do Solicitante<br/>e do Administrador"]:::container
    api["<b>API Backend</b><br/><small>[Contêiner: API REST]</small><br/>Regras de reservas, fila,<br/>máquinas, estoque e compras"]:::container
    db[("<b>Banco de Dados</b><br/><small>[PostgreSQL]</small><br/>Dados + tabela outbox")]:::container
    worker["<b>Worker de Integrações</b><br/><small>[Contêiner: segundo plano]</small><br/>Entrega os eventos aos<br/>sistemas externos"]:::container
    broker[["<b>Broker de Mensagens</b><br/><small>[RabbitMQ]</small><br/>Filas, retentativa e DLQ"]]:::container

    login["<b>Sistema de Login</b><br/><small>[Sistema externo]</small>"]:::externo
    notif["<b>Serviço de Notificação</b><br/><small>[Sistema externo]</small><br/>Envia os e-mails"]:::externo
    fin["<b>Sistema Financeiro</b><br/><small>[Sistema externo]</small><br/>Aprova ou recusa compras;<br/>responde à API por webhook"]:::externo
    forn["<b>Portal do Fornecedor</b><br/><small>[Sistema externo]</small><br/>Recebe pedidos; confirma<br/>entrega à API por webhook"]:::externo

    sol -- "Usa [HTTPS]" --> web
    adm -- "Usa [HTTPS]" --> web
    web -- "Chama [JSON/HTTPS]" --> api
    api -- "Autentica [OIDC]" --> login
    api -- "Grava dados e eventos<br/>na mesma transação [SQL]" --> db
    db -- "Eventos pendentes<br/>do outbox [SQL]" --> worker
    worker <-- "AMQP" --> broker
    worker -- "E-mails" --> notif
    worker -- "Solicitações<br/>de compra" --> fin
    worker -- "Pedidos<br/>aprovados" --> forn
```

<details>
<summary>Versão na notação C4 oficial do Mermaid (<code>c4-containers.mmd</code>)</summary>

O layout automático do C4 no Mermaid é limitado e cruza algumas setas. O conteúdo é o mesmo da versão acima.

```mermaid
C4Container
    title Contêineres (C4 Nível 2) - Sistema FabLab

    Person(solicitante, "Solicitante", "Reserva máquinas")
    Person(admin, "Administrador/Técnico", "Aprova reservas, gerencia estoque")

    System_Boundary(fablab, "Sistema FabLab") {
        Container(web, "Aplicação Web", "SPA", "Telas do sistema")
        Container(api, "API Backend", "API REST", "Regras de negócio")
        ContainerDb(db, "Banco de Dados", "PostgreSQL", "Dados + outbox")
        Container(worker, "Worker de Integrações", "Segundo plano", "Entrega eventos com retentativa")
        ContainerQueue(broker, "Broker de Mensagens", "RabbitMQ", "Filas e DLQ")
    }

    System_Ext(login, "Sistema de Login", "OIDC")
    System_Ext(notificacao, "Serviço de Notificação", "E-mail")
    System_Ext(financeiro, "Sistema Financeiro", "Aprova compras")
    System_Ext(fornecedor, "Portal do Fornecedor", "Recebe pedidos")

    Rel(solicitante, web, "Usa")
    Rel(admin, web, "Usa")
    Rel(web, api, "JSON/HTTPS")
    Rel(api, login, "Autentica")
    Rel(api, db, "SQL")
    Rel(worker, db, "Lê outbox")
    Rel(worker, broker, "AMQP")
    Rel(worker, notificacao, "E-mails")
    Rel(worker, financeiro, "Compras")
    Rel(worker, fornecedor, "Pedidos")
```

</details>

## 3. Sequência: reserva com cenários de falha

Passo 2 do laboratório. **FALHA 1:** banco não responde em 3 s. **FALHA 2:** Serviço de Notificação fora do ar, com retentativa pelo Broker de Mensagens.

```mermaid
sequenceDiagram
    autonumber
    actor Sol as Solicitante
    actor Adm as Administrador/Técnico
    participant API as API Backend
    participant DB as Banco de Dados
    participant Worker as Worker de Integrações
    participant Broker as Broker de Mensagens
    participant Not as Serviço de Notificação

    Note over Sol,API: Usuários já autenticados no Sistema de Login (RF24)

    rect rgb(240, 246, 255)
    Note over Sol,DB: 1. Pedido de reserva
    Sol->>API: Solicitar reserva da máquina no período (RF02)
    API->>DB: Verificar manutenção e conflitos (timeout 3 s)

    alt FALHA 1: banco não responde em 3 s
        API-->>Sol: Erro 503 "tente novamente" (nada foi gravado)
        Note over Sol,API: Circuit breaker evita sobrecarregar o banco.<br/>Reenviar com a mesma chave não duplica a reserva.
    else Máquina em manutenção no período
        API-->>Sol: Reserva não permitida (RF13)
    else Período livre
        API->>DB: Gravar reserva PENDENTE + evento no outbox
        API-->>Sol: Aguardando aprovação
    else Conflito de horário (RF03)
        API->>DB: Gravar reserva EM_FILA + evento no outbox (RF04)
        API-->>Sol: Em fila, posição N (RF05)
    end
    end

    rect rgb(245, 245, 245)
    Note over Adm,DB: 2. Decisão do Administrador/Técnico
    alt Aprovar (RF06)
        Adm->>API: Aprovar reserva
        API->>DB: Marcar APROVADA + evento no outbox
    else Negar (RF07)
        Adm->>API: Negar com justificativa (RF08)
        API->>DB: Marcar NEGADA + promover próxima da fila + evento no outbox
    end
    end

    rect rgb(240, 250, 240)
    Note over Worker,Not: 3. Notificação em segundo plano (RF25)
    Worker->>DB: Buscar eventos pendentes do outbox
    Worker->>Broker: Publicar na fila de notificações
    Broker->>Worker: Entregar mensagem
    Worker->>Not: Enviar e-mail ao Solicitante

    alt FALHA 2: Serviço de Notificação indisponível
        Not--xWorker: Sem resposta / erro 503
        Worker->>Broker: Devolver para a fila de retentativa
        Note over Worker,Broker: Até 5 tentativas com espera crescente.<br/>Depois: dead-letter queue e alerta ao Administrador.<br/>A reserva já está salva e não é afetada.
    else E-mail enviado
        Not-->>Sol: E-mail com o resultado
        Worker->>Broker: Confirmar processamento (ACK)
    end
    end
```

## 4. Sequência: compra de insumos

```mermaid
sequenceDiagram
    autonumber
    actor Adm as Administrador/Técnico
    participant API as API Backend
    participant DB as Banco de Dados
    participant Worker as Worker de Integrações
    participant Fin as Sistema Financeiro
    participant Forn as Portal do Fornecedor

    rect rgb(240, 246, 255)
    Note over Adm,DB: 1. Solicitação
    Adm->>API: Solicitar compra de insumo (RF18)
    API->>DB: Gravar pedido SOLICITADO + evento no outbox
    API-->>Adm: Pedido registrado
    end

    rect rgb(245, 245, 245)
    Note over Worker,Fin: 2. Aprovação financeira
    Worker->>Fin: Enviar solicitação (via Broker de Mensagens)
    Note over Worker,Fin: Se o Financeiro estiver fora do ar,<br/>o Worker tenta de novo mais tarde
    Fin->>API: Decisão por webhook

    alt Recusada (RF20)
        API->>DB: Marcar RECUSADO + avisar o Administrador
    else Aprovada (RF19)
        API->>DB: Marcar APROVADO + evento no outbox
    end
    Note over Fin,API: Resposta repetida é ignorada (idempotência)
    end

    rect rgb(240, 250, 240)
    Note over Worker,Forn: 3. Pedido e entrega (só se aprovado)
    Worker->>Forn: Enviar pedido aprovado (RF21)
    Forn->>API: Confirmar entrega por webhook (RF22)
    API->>DB: Marcar ENTREGUE + somar ao estoque (RF23)
    Note over Forn,API: Confirmação repetida não soma o estoque de novo
    end
```

## 5. Sequência: manutenção de máquina

```mermaid
sequenceDiagram
    autonumber
    actor Adm as Administrador/Técnico
    participant API as API Backend
    participant DB as Banco de Dados

    Adm->>API: Marcar máquina em manutenção no período (RF12)
    API->>DB: Gravar manutenção
    API->>DB: Buscar reservas já feitas para esse período
    Note over API,DB: Daqui em diante, novas reservas no período<br/>são recusadas (RF13)

    alt Há reservas no período
        API-->>Adm: Manutenção registrada + lista de reservas afetadas
        Note over Adm,API: Essas reservas não podem mais ser aprovadas
        Adm->>API: Negar cada uma com justificativa (RF07, RF08)
        Note over API: O Solicitante é avisado por e-mail,<br/>como em qualquer negação
    else Nenhuma reserva no período
        API-->>Adm: Manutenção registrada
    end
```

## 6. Classes: interfaces e injeção de dependência

Cada serviço recebe as dependências como interfaces pelo construtor.

```mermaid
classDiagram
    direction TB

    class IReservaService {
        <<interface>>
        +solicitar(maquinaId, periodo, chave) Reserva
        +aprovar(reservaId)
        +negar(reservaId, justificativa)
        +cancelar(reservaId)
    }
    class ReservaService {
        +ReservaService(IMaquinaService, IFilaPrioridade, INotificacao, IAutenticacao)
    }
    IReservaService <|.. ReservaService

    class IMaquinaService {
        <<interface>>
        +consultarDisponibilidade(maquinaId, periodo)
        +estaEmManutencao(maquinaId, periodo) bool
        +marcarManutencao(maquinaId, periodo) Reserva[]
        +cadastrar(maquina)
        +atualizar(maquina)
    }
    class IFilaPrioridade {
        <<interface>>
        +enfileirar(reservaId) int
        +consultarPosicao(reservaId) int
        +reordenar(maquinaId, novaOrdem)
        +promoverProxima(maquinaId, periodo)
    }
    class INotificacao {
        <<interface>>
        +notificar(destinatario, evento)
    }
    class IAutenticacao {
        <<interface>>
        +autenticar(credenciais) Perfil
        +verificarPermissao(perfil, funcao) bool
    }

    ReservaService ..> IMaquinaService : injetada
    ReservaService ..> IFilaPrioridade : injetada
    ReservaService ..> INotificacao : injetada
    ReservaService ..> IAutenticacao : injetada

    class ICompraService {
        <<interface>>
        +solicitarCompra(insumoId, quantidade) PedidoCompra
        +registrarDecisaoFinanceira(pedidoId, aprovado)
        +confirmarEntrega(pedidoId, entregaId)
    }
    class CompraService {
        +CompraService(IEstoqueService, INotificacao)
    }
    class IEstoqueService {
        <<interface>>
        +registrarConsumo(insumoId, quantidade, reservaId)
        +consultarEstoque() Insumo[]
        +registrarEntrada(insumoId, quantidade)
    }
    ICompraService <|.. CompraService
    CompraService ..> IEstoqueService : injetada
    CompraService ..> INotificacao : injetada

    class MaquinaService
    class FilaPrioridadeService
    class NotificacaoOutboxService
    class ControleAcessoService
    class EstoqueService
    IMaquinaService <|.. MaquinaService
    IFilaPrioridade <|.. FilaPrioridadeService
    INotificacao <|.. NotificacaoOutboxService
    IAutenticacao <|.. ControleAcessoService
    IEstoqueService <|.. EstoqueService
```

## 7. Modelo de domínio

```mermaid
classDiagram
    direction LR

    class Maquina {
        id
        tipo
        estado : EstadoMaquina
    }
    class Reserva {
        id
        periodo
        status : StatusReserva
        posicaoFila
        justificativa
    }
    class Insumo {
        id
        nome
        unidade
        quantidadeEmEstoque
    }
    class PedidoCompra {
        id
        quantidade
        status : StatusPedido
    }

    class StatusReserva {
        <<enumeration>>
        PENDENTE
        EM_FILA
        APROVADA
        NEGADA
        CANCELADA
    }
    class EstadoMaquina {
        <<enumeration>>
        DISPONIVEL
        RESERVADA
        EM_USO
        EM_MANUTENCAO
    }
    class StatusPedido {
        <<enumeration>>
        SOLICITADO
        APROVADO
        RECUSADO
        ENVIADO_FORNECEDOR
        ENTREGUE
    }

    Reserva "*" --> "1" Maquina : reserva
    PedidoCompra "*" --> "1" Insumo : repõe
    Reserva ..> StatusReserva
    Maquina ..> EstadoMaquina
    PedidoCompra ..> StatusPedido
```

## 8. Componentes da API Backend (C4 Nível 3)

Cada seta mostra a interface usada.

```mermaid
flowchart LR
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef componente fill:#85bbf0,stroke:#5d82a8,color:#0b2545
    classDef externo fill:#8c8c8c,stroke:#6b6b6b,color:#fff

    web["<b>Aplicação Web</b><br/><small>[Contêiner]</small>"]:::container

    subgraph api["API Backend: componentes (todos persistem no Banco de Dados)"]
        aces["<b>Controle de Acesso</b><br/><small>ControleAcessoService</small>"]:::componente
        res["<b>Reservas</b><br/><small>ReservaService</small>"]:::componente
        maq["<b>Máquinas e Manutenção</b><br/><small>MaquinaService</small>"]:::componente
        fila["<b>Fila de Prioridade</b><br/><small>FilaPrioridadeService</small>"]:::componente
        comp["<b>Compras</b><br/><small>CompraService</small>"]:::componente
        noti["<b>Notificações</b><br/><small>NotificacaoOutboxService<br/>grava eventos no outbox</small>"]:::componente
        est["<b>Estoque de Insumos</b><br/><small>EstoqueService</small>"]:::componente
    end

    login["<b>Sistema de Login</b><br/><small>[Sistema externo]</small>"]:::externo

    web -- "IAutenticacao" --> aces
    web -- "IReservaService" --> res
    web -- "IMaquinaService" --> maq
    web -- "ICompraService" --> comp
    web -- "IEstoqueService" --> est

    res -- "IMaquinaService" --> maq
    res -- "IFilaPrioridade" --> fila
    res -- "INotificacao" --> noti
    comp -- "IEstoqueService" --> est
    comp -- "INotificacao" --> noti

    aces -- "OIDC" --> login
```

## 9. Atividades: reserva

```mermaid
flowchart TB
    classDef decisao fill:#fff8dc,stroke:#8a6d00
    classDef inicio fill:#000,stroke:#000,color:#fff
    classDef acao fill:#eef4fb,stroke:#5d82a8,color:#0b2545

    INI(("início")):::inicio

    subgraph SOL["Solicitante"]
        A1["Consultar disponibilidade (RF01)"]:::acao
        A2["Solicitar reserva (RF02)"]:::acao
    end

    subgraph SIS["Sistema FabLab"]
        D1{"Máquina em manutenção<br/>no período?"}:::decisao
        A3["Recusar a reserva"]:::acao
        D2{"Conflito de horário?<br/>(RF03)"}:::decisao
        A4["Colocar na fila de<br/>prioridade (RF04)"]:::acao
        A5["Encaminhar para aprovação"]:::acao
        A8["Registrar reserva APROVADA"]:::acao
        A9["Registrar justificativa e<br/>promover a próxima da fila"]:::acao
        A10["Notificar o Solicitante<br/>por e-mail (RF25)"]:::acao
    end

    subgraph ADM["Administrador/Técnico"]
        D3{"Aprovar?"}:::decisao
        A6["Aprovar (RF06)"]:::acao
        A7["Negar com justificativa<br/>(RF07, RF08)"]:::acao
    end

    FIM1(("fim")):::inicio
    FIM2(("fim")):::inicio

    INI --> A1 --> A2 --> D1
    D1 -- sim --> A3 --> FIM1
    D1 -- não --> D2
    D2 -- sim --> A4 --> A5
    D2 -- não --> A5
    A5 --> D3
    D3 -- sim --> A6 --> A8 --> A10
    D3 -- não --> A7 --> A9 --> A10
    A10 --> FIM2
```

## 10. Atividades: cancelamento

```mermaid
flowchart TB
    classDef decisao fill:#fff8dc,stroke:#8a6d00
    classDef inicio fill:#000,stroke:#000,color:#fff
    classDef acao fill:#eef4fb,stroke:#5d82a8,color:#0b2545

    INI(("início")):::inicio

    subgraph SOL["Solicitante"]
        C1["Cancelar reserva (RF09)"]:::acao
    end

    subgraph SIS["Sistema FabLab"]
        D1{"Reserva já negada<br/>ou cancelada?"}:::decisao
        C2["Informar que não<br/>pode cancelar"]:::acao
        C3["Liberar o horário e<br/>promover a próxima da fila"]:::acao
        C4["Notificar os envolvidos<br/>por e-mail (RF25)"]:::acao
    end

    FIM1(("fim")):::inicio
    FIM2(("fim")):::inicio

    INI --> C1 --> D1
    D1 -- sim --> C2 --> FIM1
    D1 -- não --> C3 --> C4 --> FIM2
```

## 11. Estados da Reserva

```mermaid
stateDiagram-v2
    direction TB

    [*] --> PENDENTE : solicitada,<br/>período livre
    [*] --> EM_FILA : solicitada,<br/>com conflito

    EM_FILA --> PENDENTE : chegou a vez
    PENDENTE --> APROVADA : aprovada
    PENDENTE --> NEGADA : negada
    EM_FILA --> NEGADA : negada

    PENDENTE --> CANCELADA : cancelada
    EM_FILA --> CANCELADA : cancelada
    APROVADA --> CANCELADA : cancelada

    APROVADA --> [*]
    NEGADA --> [*]
    CANCELADA --> [*]

    note right of NEGADA
        Sempre com justificativa (RF08)
    end note
```

## 12. Estados do Pedido de Compra

```mermaid
stateDiagram-v2
    direction LR
    [*] --> SOLICITADO : solicitar compra (RF18)
    SOLICITADO --> APROVADO : Financeiro aprova (RF19)
    SOLICITADO --> RECUSADO : Financeiro recusa (RF20)
    APROVADO --> ENVIADO_FORNECEDOR : pedido encaminhado (RF21)
    ENVIADO_FORNECEDOR --> ENTREGUE : Fornecedor confirma entrega (RF22)<br/>estoque atualizado (RF23)
    RECUSADO --> [*]
    ENTREGUE --> [*]
```
