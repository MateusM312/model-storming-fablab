# Decisões e pendências da modelagem: Sistema FabLab

**Atualizado em:** 24/11/2026
**Referências:**
- roteiro do professor "IA e Modelagem Agêntica: Laboratório Prático de Auditoria e Automação de Arquitetura" (Passos 1 a 4);
- [Visão de Domínio e Backlog](visao-dominio-e-backlog.pdf);
- [Validação cross-diagramas](architecture/validacao_cross_diagramas_fablab_dia27.pdf);
- [Plano de Chaos Engineering](architecture/chaos_engineering.pdf).

Este documento explica **o que foi corrigido, por que cada decisão foi tomada e o que a equipe ainda precisa fazer**. Cada item indica a fonte que o justifica:

| Fonte | Significado |
|---|---|
| **Roteiro** | Exigência explícita do roteiro do professor |
| **Domínio** | Regra de negócio ou requisito (RF) do documento de Visão de Domínio e Backlog |
| **Validação** | Problema apontado no relatório de validação cross-diagramas |
| **Chaos** | Hipótese definida no plano de Chaos Engineering |
| **Decisão** | Interpretação de modelagem que nenhum documento define. **A equipe precisa confirmar** |

---

## 1. Situação das exigências do roteiro

| Exigência | Arquivo | Situação |
|---|---|---|
| Passo 1: C4 Nível 2 em `C4Container` e análise em 3 tópicos | [`c4-containers.mmd`](architecture/c4-containers.mmd) (notação oficial), [`c4-containers-visual.mmd`](architecture/c4-containers-visual.mmd) (versão legível), [`auditoria-c4-containers.md`](architecture/auditoria-c4-containers.md) | ✅ Feito |
| Passo 2: `sequenceDiagram` com banco > 3 s, e-mail indisponível, `alt/else` e retentativa por mensageria | [`sequencia-reserva.mmd`](architecture/sequencia-reserva.mmd) | ✅ Feito |
| Passo 3: ADR no padrão Nygard em `docs/adr/` | [`0001-escolha-tecnologica.md`](adr/0001-escolha-tecnologica.md) | ✅ Feito (**decisão a confirmar**, ver §3) |
| Passo 4: nomes iguais entre C4 e Sequência | Todos os diagramas | ✅ Conferido por busca automática |
| Passo 4: sintaxe compilável | Todos os `.mmd` e o `Readme.md` | ✅ Renderizam no Mermaid CLI 11.4.2. ⚠️ Conferir no GitHub após o push |
| Passo 4: interfaces e injeção de dependência | [`classes-interfaces.mmd`](architecture/classes-interfaces.mmd), [`modelo-dominio.mmd`](architecture/modelo-dominio.mmd), [`componentes.mmd`](architecture/componentes.mmd) | ✅ Feito |
| Passo 4: pastas `/docs/architecture/` e `/docs/adr/` | Estrutura do repositório | ✅ Feito |

## 2. O que foi corrigido

### Diagramas reescritos

| Antes | Depois | Fonte |
|---|---|---|
| `containers.mmd`: módulos internos da API modelados como contêineres e integrações externas síncronas | `c4-containers.mmd`: só contêineres implantáveis (Aplicação Web, API Backend, Worker de Integrações, Broker de Mensagens, Banco de Dados). Integrações de saída via RabbitMQ | Roteiro, Chaos exp. 2, 7, 8 |
| `sequencia.mmd` era flowchart; a sequência do Readme terminava em "cole aqui o restante" | `sequencia-reserva.mmd`: os dois cenários de falha, negação com justificativa e notificação assíncrona, em 3 etapas (pedido, decisão, notificação) | Roteiro, Validação itens 1, 8 |
| "Máquina disponível?" verificado antes de "há conflito?", então o 2º solicitante nunca entrava na fila | Disponibilidade **por período**: manutenção leva a recusa, conflito leva à fila, período livre leva a pendente | Domínio regras 1, 2 e 7, Validação item 3 |
| `adr.mmd` era cópia do diagrama de contêineres | ADR real em `docs/adr/0001-escolha-tecnologica.md` | Roteiro, Validação item 2 |
| `interface.mmd` era flowchart imitando UML, sem `ICompraService` | `classes-interfaces.mmd`: `classDiagram` com construtores injetados, `ICompraService` e operações que faltavam. Entidades e enums ficaram em `modelo-dominio.mmd` | Roteiro, Validação itens 4, 6, 11, 17 |
| `componentes.mmd` sem COMP → EST, COMP → NOTI e outbox | Ligações adicionadas, nomes padronizados, Worker e Broker incluídos | Validação itens 4, 5, 9, 15 |
| `atividade.mmd` com a mesma falha de lógica e cancelamento sem verificação | `atividade-reserva.mmd` corrigido; cancelamento separado em `atividade-cancelamento.mmd`, com verificação de status | Validação itens 3, 7 |

### Diagramas novos

| Arquivo | Conteúdo | Fonte |
|---|---|---|
| [`c4-contexto-visual.mmd`](architecture/c4-contexto-visual.mmd) e [`c4-contexto.mmd`](architecture/c4-contexto.mmd) | C4 Nível 1 (versão legível e notação oficial) | Validação item 16 |
| [`sequencia-compra.mmd`](architecture/sequencia-compra.mmd) | Solicitar compra → Financeiro aprova ou recusa → Fornecedor → entrega → estoque. Com retentativa e idempotência | Domínio RF16, RF18–RF23, regras 9–12; Chaos exp. 7, 8 |
| [`sequencia-manutencao.mmd`](architecture/sequencia-manutencao.mmd) | Marcar manutenção; reservas afetadas no período são sinalizadas ao Administrador/Técnico | Domínio RF12, RF13; Chaos exp. 5 |
| [`estados-reserva.mmd`](architecture/estados-reserva.mmd) | Ciclo de vida da Reserva | Validação item 11 |
| [`estados-pedido-compra.mmd`](architecture/estados-pedido-compra.mmd) | Ciclo de vida do Pedido de Compra | Domínio RF18–RF23, Validação item 11 |

### Legibilidade dos diagramas

- **C4 em duas versões.** O C4 nativo do Mermaid (`C4Context`/`C4Container`) ignora as configurações de layout e cruza setas. Testamos 1, 3 e 4 colunas e o resultado foi idêntico. Por isso cada diagrama C4 tem:
  - a **versão legível** (`*-visual.mmd`), em flowchart com as cores e rótulos do C4 ([Pessoa], [Contêiner], [Sistema externo]), usada no Readme e na apresentação;
  - a **versão na notação oficial** (`c4-contexto.mmd`, `c4-containers.mmd`), mantida porque o roteiro pede `C4Container`/`Container()`/`Rel()`. No Readme ela fica recolhida.
  - **As duas precisam ser mantidas iguais em conteúdo.**
- **Sequências enxutas.** A de reserva caiu de 60 para 20 passos. Autenticação, consulta de disponibilidade, cancelamento e reordenação da fila saíram dela e estão nos diagramas de atividades e de estados.
- **Classes divididas** em `classes-interfaces.mmd` (interfaces e injeção de dependência) e `modelo-dominio.mmd` (entidades e enums).
- **Componentes** sem as 7 setas repetidas para o banco. O título do grupo diz que todos os componentes persistem no Banco de Dados.

### Organização

- Todos os diagramas ficam em `docs/architecture/` e o ADR em `docs/adr/`. **Roteiro**
- Foram removidas as duplicatas (`docs/diagrams/Atividade/`, `Atividade.zip`, `chaos_engineering.pdf` repetido e `adr.mmd`). Todas foram conferidas por MD5 antes de apagar.
- O diagrama de casos de uso e a visão de domínio foram copiados para o repositório (`docs/architecture/casos-de-uso.pdf`, `docs/visao-dominio-e-backlog.pdf`). **Validação item 16**
- O `Readme.md` virou um índice. Os `.mmd` são a fonte oficial, e os blocos do Readme são cópias geradas a partir deles. **Validação itens 12, 19**

### Nomes padronizados (usados em todos os diagramas)

| Tipo | Nome |
|---|---|
| Sistema | Sistema FabLab |
| Atores | Solicitante, Administrador/Técnico |
| Contêineres | Aplicação Web, API Backend, Worker de Integrações, Broker de Mensagens (RabbitMQ), Banco de Dados (PostgreSQL) |
| Sistemas externos | Sistema de Login, Serviço de Notificação, Sistema Financeiro, Portal do Fornecedor |

## 3. Decisões que a equipe precisa confirmar

Os documentos do projeto não definem estes pontos. Escolhi a opção mais coerente com as regras de negócio. Se a equipe discordar, os diagramas indicados devem ser ajustados.

| # | Decisão | Por quê | Onde aparece |
|---|---|---|---|
| D1 | **RabbitMQ com Transactional Outbox** como decisão do ADR 0001 | O Passo 2 exige mensageria e o roteiro cita RabbitMQ como exemplo. O plano de chaos exige outbox (exp. 2) e idempotência (exp. 7, 8). Segundo a validação, um pacote anterior da equipe já usava RabbitMQ | ADR, C4, sequências, componentes |
| D2 | Banco lento (> 3 s): **falhar rápido** com erro controlado; a retentativa por fila vale só para o e-mail | Colocar a própria reserva numa fila deixaria o usuário sem resposta. O cliente reenvia com a mesma chave de idempotência e não duplica a reserva (chaos exp. 3, 4) | `sequencia-reserva.mmd` |
| D3 | Aprovar uma reserva **não** muda o estado global da máquina para "reservada"; a reserva APROVADA ocupa só o seu período | Com estado global, o 2º solicitante nunca entraria na fila, o que contradiz as regras 1 e 2 | sequência, atividade, estados |
| D4 | Negar ou cancelar **promove a próxima reserva da fila** para PENDENTE | O horário fica livre. A atividade original já tinha "liberar horário e atualizar fila" | sequência, atividade, estados |
| D5 | Não é possível cancelar uma reserva já NEGADA ou CANCELADA | A validação (item 7) apontou que o cancelamento não verificava o status | atividade |
| D6 | Financeiro e Fornecedor são **sistemas externos** que respondem por **webhook** | Já eram sistemas externos no C4 original. Os atores do caso de uso são as pessoas que usam esses sistemas. O webhook permite a resposta assíncrona e idempotente | C4, compra, componentes |
| D7 | Sistema de Login continua **síncrono** e falha fechado | Sem autenticação nenhuma operação é possível (chaos exp. 1) | ADR, C4 |
| D8 | Ao marcar manutenção, as reservas afetadas são **sinalizadas** e o Administrador/Técnico as nega com justificativa. A aprovação é bloqueada se houver manutenção no período | O plano de chaos (exp. 5) diz "sinalizadas ao administrador, sem ficarem aprovadas por engano". A negação reaproveita o fluxo que já existe | `sequencia-manutencao.mmd` |
| D9 | Uma **constraint de exclusão** no banco impede duas reservas APROVADAS sobrepostas na mesma máquina | É a garantia citada no plano de chaos (exp. 4) contra reservas duplicadas sob concorrência | `sequencia-reserva.mmd` |
| D10 | "Saldo de créditos" **não** foi modelado | Aparece só como exemplo genérico no roteiro. Não existe no domínio do FabLab | — |
| D11 | Tecnologia da Web e da API ficou como "SPA" e "API REST" | O original só tinha exemplos ("ex.: React", "ex.: Node.js / Spring Boot"). A escolha é da equipe | C4 |
| D12 | C4 mantido em **duas versões** (visual e notação oficial) | A notação oficial é exigida pelo roteiro, mas o layout do Mermaid é ilegível. Se o professor aceitar só a versão visual, a oficial pode ser removida | C4 |
| D13 | No diagrama de contêineres, as respostas do Financeiro e do Fornecedor por **webhook** aparecem na descrição das caixas, sem setas próprias | Setas de volta para a API embaralhavam o layout. A informação continua no diagrama e na sequência de compra | `c4-containers-visual.mmd` |

## 4. O que ainda precisa ser feito

### Obrigatório (só a equipe pode fazer)

- [ ] **Revisar e aprovar as decisões D1 a D13** acima. Em especial D1, porque o ADR está com status "Aceito".
- [ ] **Exportar de novo o diagrama de contexto do draw.io.** O arquivo "Diagrama de contexto.drawio.pdf" contém o diagrama de casos de uso. Como alternativa, usar o [`c4-contexto.mmd`](architecture/c4-contexto.mmd).
- [ ] **Limpar o `chaos_engineering.pdf`.** O final tem texto de conversa com IA ("Quer que eu detalhe algum experimento…", "minha resposta anterior foi cortada").
- [ ] **Abrir o Readme no GitHub** e conferir se todos os diagramas renderizam. O C4 no GitHub é experimental.
- [ ] **Preparar a defesa (23 a 27/11).** Saber explicar o ADR (por que RabbitMQ, quais os trade-offs) e os cenários de falha da sequência.
- [ ] **Se o professor pedir o processo:** rodar os prompts SCALE do roteiro num agente de IA web e mostrar o resultado no Mermaid Live Editor.

### Regras que os documentos não definem (perguntar ao professor ou decidir em equipe)

- [ ] **Compra recusada:** o modelo só registra RECUSADO e notifica. É possível solicitar de novo? Existe um limite?
- [ ] **Atraso do fornecedor:** não há prazo nem ação definida. Hoje o pedido fica ENVIADO_FORNECEDOR indefinidamente.
- [ ] **Estados da Máquina:** o domínio lista "disponível, reservada, em uso, em manutenção", mas não diz quando a máquina entra ou sai de "em uso" nem como sai da manutenção. Por isso **não foi feito o diagrama de estados da Máquina**.
- [ ] **Registro de consumo (RF15) e reposição (Política de Reposição):** não há gatilho definido (ex.: estoque mínimo) para sugerir compra.
- [ ] **Caso de uso "Solicitar compra «include» Aprovar compra":** o próprio documento de domínio recomenda confirmar com o professor.

### Opcional

- [ ] Definir a stack real (Web e API) e registrá-la num ADR 0002.
- [ ] Registrar como recomendação de produção: mais de uma instância da API e réplica do PostgreSQL (citado na auditoria do C4).

## 5. Como foi verificado

- **Sintaxe:** todos os `.mmd` e todos os blocos do `Readme.md` renderizaram sem erro no Mermaid CLI 11.4.2. Os blocos do Readme foram comparados aos `.mmd` e são idênticos.
- **Nomes:** busca automática confirmou que os contêineres e sistemas externos têm o mesmo nome no C4, nas sequências e nos componentes, e que nenhum nome antigo ("FabLab Manager", "Portal Web", "LoginService", "Usuário Acadêmico") restou nos diagramas.
- **Arquivos removidos:** comparados por MD5 com as cópias mantidas antes de apagar.
- **Limitação:** a renderização no GitHub só pode ser confirmada após o push.
