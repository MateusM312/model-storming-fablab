# Defesa de Arquitetura de Software — Pull Request

## Autor do PR
- **Nome do Estudante:** Mateus Mendes da Cunha
- **Artefato(s) de Responsabilidade:** c4-contexto.mmd, c4-contexto-visual.mmd, c4-containers.mmd, c4-containers-visual.mmd

## O que este PR introduz na Documentação Viva?
Adiciona o modelo C4 do Sistema FabLab (reservas de máquinas, fila de prioridade, manutenção, estoque e compras) em Mermaid, nos níveis 1 e 2. Cada nível tem duas versões: a notação C4 nativa do Mermaid (`C4Context` e `C4Container`) e uma versão visual em `flowchart`, com cores e descrições mais legíveis.
- [ ] **Visão Funcional / Estrutural:** Diagrama de Casos de Uso ou Diagrama de Classes e Interfaces.
- [ ] **Visão Comportamental:** Diagrama de Sequência, Atividades (com Raias/Fork-Join) ou Máquina de Estados.
- [x] **Visão Arquitetural Macro:** Modelo C4 (L1 e L2 em Mermaid).
- [ ] **Governança:** Registros de Decisão Arquitetural (ADR).

## Defesa das Decisões de Design (Justificativa Técnica)
1. **Escolha de Abstrações e Desacoplamento:**
   - No nível 1 o FabLab é uma caixa única, com dois atores (Solicitante e Administrador/Técnico) e quatro sistemas externos: Login, Notificação, Financeiro e Portal do Fornecedor.
   - No nível 2 o sistema se divide em Aplicação Web (SPA), API Backend, Banco de Dados (PostgreSQL), Worker de Integrações e Broker de Mensagens (RabbitMQ).
   - A API concentra as regras de negócio e só fala com o Login, de forma síncrona, e com o banco.
   - Toda integração de saída (e-mail, compras, pedidos) fica isolada no Worker. A API não conhece o protocolo nem a disponibilidade dos sistemas externos.
   - Os sistemas externos respondem à API por webhook (Financeiro e Portal do Fornecedor), o que mantém o fluxo de ida e volta consistente com os `BiRel` do nível 1.
2. **Dinâmica e Resiliência:**
   - A API grava os dados e o evento na mesma transação (padrão Transactional Outbox). Se o evento é gravado, ele não se perde, mesmo que um sistema externo esteja fora do ar.
   - O Worker lê os eventos pendentes, marca como enviados e os entrega aos sistemas externos pelo broker, com retentativa e DLQ (fila de mensagens mortas) para as falhas definitivas.
   - Assim, uma falha do Financeiro, do Portal do Fornecedor ou do serviço de e-mail não derruba as reservas nem as operações do dia a dia.
   - Os fluxos alternativos (`alt/else`) não aparecem nestes diagramas, porque o C4 é estático. Eles pertencem ao Diagrama de Sequência.
3. **Trade-offs:**
   - Prós: consistência entre dado e evento, tolerância a falhas de terceiros e API simples de manter.
   - Contras: mais componentes para operar (Worker e Broker) e entrega "pelo menos uma vez", o que exige consumidores idempotentes. Há também latência entre a gravação e a entrega do evento.
   - Os webhooks exigem validação da origem (assinatura ou token) na API.
   - Alternativa descartada: a API chamar os sistemas externos diretamente. É mais simples, mas acopla a disponibilidade da API à dos terceiros.

## Consistência Cross-Diagramas (Checklist de Autoverificação)
- [ ] Os nomes de métodos chamados no Diagrama de Sequência existem nas interfaces do Diagrama de Classes. *(N/A neste PR: não inclui sequência nem classes.)*
- [x] Os contêineres e componentes citados correspondem exatamente à nomenclatura do Modelo C4 L2/L3. *(L1 e L2 usam os mesmos nomes e descrições nas versões `.mmd` e visual. Não há L3 neste PR.)*
- [ ] As transições de estado refletem os gatilhos disparados pelas atividades da aplicação. *(N/A neste PR: não inclui máquina de estados.)*

## Validação da Sintaxe Diagrams-as-Code / Models-as-Code
- **Ferramenta de Compilação/Renderização Utilizada:** Mermaid Live
- **Status da Compilação:** [x] Código limpo e sem erros de sintaxe ao renderizar.