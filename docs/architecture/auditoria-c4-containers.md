# Auditoria do C4 Nível 2 (Contêineres): Sistema FabLab

Diagrama corrigido: [`c4-containers-visual.mmd`](c4-containers-visual.mmd) (versão legível) e [`c4-containers.mmd`](c4-containers.mmd) (notação C4 oficial)
Decisão relacionada: [ADR 0001](../adr/0001-escolha-tecnologica.md)

## Análise

1. **Acoplamento síncrono com sistemas externos.** A API chamava o Serviço de Notificação, o Sistema Financeiro e o Portal do Fornecedor diretamente por HTTPS. Uma queda do e-mail derrubava a aprovação de reservas.
   **Correção:** as integrações de saída passam pelo **Broker de Mensagens** (RabbitMQ) e pelo **Worker de Integrações**, com retentativa e dead-letter queue. Só o **Sistema de Login** continua síncrono e falha fechado.

2. **Pontos únicos de falha (SPOF) e consistência.** A **API Backend** e o **Banco de Dados** são instâncias únicas, e publicar eventos fora da transação podia perder notificações.
   **Correção:** os eventos são gravados no **outbox** na mesma transação. As chamadas ao banco têm timeout de 3 s com circuit breaker, e as requisições de reserva usam chave de idempotência para que o reenvio não duplique. *Fica como recomendação para produção: mais de uma instância da API atrás de um balanceador e réplica do PostgreSQL.*

3. **Granularidade incorreta do nível 2.** O diagrama anterior listava "Módulo de Reservas", "Módulo de Máquinas" etc. como contêineres, embora fossem componentes da mesma API (a própria descrição dizia "Componente do backend").
   **Correção:** o nível 2 mostra apenas unidades implantáveis: **Aplicação Web, API Backend, Worker de Integrações, Broker de Mensagens e Banco de Dados**. Os módulos estão no diagrama de componentes ([`componentes.mmd`](componentes.mmd)).
