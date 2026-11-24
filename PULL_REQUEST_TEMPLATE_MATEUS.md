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
    Rel(web, api, "Chama", "JSON/HTTPS")
    Rel(api, login, "Autentica", "OIDC")
    Rel(api, db, "Grava dados e eventos na mesma transação", "SQL")
    Rel(worker, db, "Lê eventos pendentes e marca como enviados", "SQL")
    Rel(worker, broker, "Publica e consome eventos", "AMQP")
    Rel(worker, notificacao, "Envia e-mails")
    Rel(worker, financeiro, "Envia solicitações de compra")
    Rel(worker, fornecedor, "Envia pedidos aprovados")
    Rel(financeiro, api, "Responde aprovação ou recusa de compra", "Webhook/HTTPS")
    Rel(fornecedor, api, "Confirma entrega do pedido", "Webhook/HTTPS")