# Defesa de Arquitetura de Software — Pull Request

## Autor do PR
- **Nome do Estudante:** Nicolas Vieira Marques
- **Artefato(s) de Responsabilidade:** [estados-pedido-compra.mmd, estados-reserva.mmd, modelo-dominio.mmd, sequencia-compra.mmd]

## O que este PR introduz na Documentação Viva?
Descreva sucintamente as alterações e os novos modelos adicionados à pasta `/docs/`:
- [x] **Visão Funcional / Estrutural:** Atualização do Modelo de Domínio (`modelo-dominio.mmd`) com a inclusão da classe `Usuario`, enum `PerfilUsuario` (RF24) e da classe de associação `ConsumoInsumo` (RF15).
- [x] **Visão Comportamental:** Refatoração dos Diagramas de Máquina de Estados (`estados-reserva.mmd` e `estados-pedido-compra.mmd`) adicionando a fase operacional de uso da máquina e tratamentos de exceção do fornecedor, além do acoplamento do `Serviço de Notificação` (RF25) no Diagrama de Sequência (`sequencia-compra.mmd`).
- [x] **Visão Arquitetural Macro:** Alinhamento dos fluxos assíncronos e integrações externas com as personas e sistemas mapeados no Modelo C4 Nível 1.
- [x] **Governança:** Mapeamento de rastreabilidade ponta a ponta dos Requisitos Funcionais (RF01 a RF25).

## Defesa das Decisões de Design (Justificativa Técnica)

### Diagnóstico de Erros e Deficiências nos Modelos Anteriores:
1. **`estados-reserva.mmd`:** O ciclo de vida da reserva era interrompido abruptamente no estado `APROVADA`, desconsiderando o momento em que o usuário opera a máquina (`EM_USO`) e finaliza o serviço (`CONCLUIDA`), momento fundamental em que ocorre o débito de insumos do estoque (RF15). Faltavam também a transição para reordenação excepcional da fila pelo Administrador (RF14) e as referências aos RFs.
2. **`modelo-dominio.mmd`:** O diagrama não possuía a entidade `Usuario`, inviabilizando a checagem de perfil de acesso (RF24) e impedindo identificar os autores de reservas e solicitações de compra. A classe `Insumo` conectava-se apenas a `PedidoCompra`, sem relação com `Reserva` (RF15). Faltavam ainda atributos essenciais como `motivoManutencao` na classe `Maquina` (RF12) e `justificativaRecusa` em `PedidoCompra` (RF20).
3. **`sequencia-compra.mmd`:** O fluxo ignorava a participação do `Serviço de Notificação` (RF25), deixando os administradores sem alertas passivos em caso de recusa do financeiro ou de disponibilização do insumo no estoque.
4. **`estados-pedido-compra.mmd`:** O fluxo assumia que todos os pedidos enviados ao fornecedor seriam entregues com sucesso, sem tratar falhas de estoque no fornecedor ou explicitar a obrigatoriedade de justificativa na recusa financeira (RF20).

---

### Fundamentação Técnica das Decisões de Design:

1. **Escolha de Abstrações e Desacoplamento:**
   - **Modelagem de Autoria e Permissões (`Usuario` e `PerfilUsuario`):** A inclusão da entidade `Usuario` conectada às solicitações de reserva e compras garante a rastreabilidade e a aplicação das regras do RF24 (controle de acesso por perfil: Solicitante, Administrador/Técnico, Financeiro, Fornecedor).
   - **Classe de Associação `ConsumoInsumo`:** Em vez de uma ligação direta e genérica N:M entre `Reserva` e `Insumo`, foi modelada a classe `ConsumoInsumo`. Essa abstração isola a quantidade consumida e o timestamp do registro (RF15), permitindo que a API efetue a atualização precisa do estoque (RF17).
   - **Extensão do Ciclo de Vida da Reserva (`EM_USO` $\rightarrow$ `CONCLUIDA`):** Reflete com precisão o estado do mundo real da oficina do FabLab, permitindo distinguir máquinas reservadas de máquinas efetivamente em execução.

2. **Dinâmica e Resiliência:**
   - **Tratamento de Exceção no Fluxo Financeiro e Suprimentos (`alt/else`):** No diagrama de sequência de compras, o fluxo alternativo garante o tratamento explícito de recusas pelo setor financeiro (RF20) com acionamento imediato do serviço de notificação ao administrador.
   - **Garantia de Idempotência:** A notação explícita nos retornos por webhook do Financeiro e Fornecedor garante que reenvios de mensagens de confirmação não dupliquem lançamentos no estoque (RF23).
   - **Notificação Assíncrona Ativa (RF25):** A adição do ator de `Notificação` desonera a API principal de manter conexões presas para reportar status aos usuários.

3. **Trade-offs:**
   - **Classe de Associação `ConsumoInsumo` vs. Atributo Direto:**
     - *Prós:* Histórico completo auditável de insumos gastos por projeto/reserva, com suporte a múltiplos insumos numa mesma utilização.
     - *Contras:* Adiciona complexidade na persistência relacional.
     - *Decisão:* Adotado pela necessidade crítica de controle de materiais (filamentos, chapas de acrílico/mdf) do FabLab.
   - **Arquitetura Event-Driven (Outbox/Worker) vs. Chamadas Síncronas:**
     - *Prós:* Evita falhas em cascata caso o Portal do Fornecedor ou o Sistema Financeiro estejam instáveis.
     - *Contras:* Requer a manutenção de componentes adicionais (Worker e Tabela Outbox).
     - *Decisão:* Adotado para resiliência de integrações externas.

## Consistência Cross-Diagramas (Checklist de Autoverificação)
Antes de solicitar a revisão de um colega, confirme o alinhamento entre os modelos:
- [x] Os nomes de métodos chamados no Diagrama de Sequência existem nas interfaces do Diagrama de Classes.
- [x] Os contêineres e componentes citados correspondem exatamente à nomenclatura do Modelo C4 L2/L3.
- [x] As transições de estado refletem os gatilhos disparados pelas atividades da aplicação.

## Validação da Sintaxe Diagrams-as-Code / Models-as-Code
- **Ferramenta de Compilação/Renderização Utilizada:** VS Code Mermaid Preview / Mermaid Live Editor
- **Status da Compilação:** [x] Código limpo e sem erros de sintaxe ao renderizar.
