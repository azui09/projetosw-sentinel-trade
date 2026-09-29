# Especificação de Requisitos — SentinelTrade

**Cliente:** Orion Capital
**Sistema:** SentinelTrade — Plataforma de Trade Financeiro de Alta Criticidade
**Tipo de documento:** Especificação de Requisitos de Software (ERS)
**Versão:** 1.0
**Data:** 15/09/2026

---

## 1. Introdução

### 1.1 Propósito
Este documento especifica os requisitos funcionais e não funcionais do **SentinelTrade**, plataforma distribuída, segura, escalável e tolerante a falhas para negociação de ações, ETFs e fundos imobiliários, contratada pela Orion Capital para substituir sistemas pouco integrados e melhorar rastreabilidade, controle de risco e auditoria.

### 1.2 Escopo
O sistema permitirá que investidores autorizados: façam login com MFA, consultem cotações e carteira, enviem e cancelem ordens de compra/venda, acompanhem o ciclo de vida das ordens e consultem histórico auditável. O sistema se integra a um provedor externo de cotações e a uma Bolsa/Corretora simulada, processando ordens de forma assíncrona, idempotente e resiliente a falhas.

### 1.3 Classificação de criticidade
Dado o impacto **financeiro, regulatório e reputacional** de falhas, todo requisito é classificado quanto à prioridade:

| Prioridade | Significado |
|---|---|
| **P0 – Crítico** | Bloqueia operação segura do sistema; falha implica risco financeiro/regulatório direto |
| **P1 – Alto** | Essencial ao negócio, mas com mitigação possível a curto prazo |
| **P2 – Médio** | Relevante para qualidade/experiência, não bloqueante |

### 1.4 Convenção de identificação
- `RF-XX` — Requisito Funcional
- `RNF-SEG-XX` — Requisito Não Funcional de Segurança
- `RNF-RES-XX` — Requisito Não Funcional de Resiliência
- `RNF-QUAL-XX` — Requisito Não Funcional de Qualidade

---

## 2. Requisitos Funcionais (RF)

| ID | Requisito | Descrição | Prioridade | Critérios de Aceite |
|---|---|---|---|---|
| RF-01 | Cadastro e gestão de investidores | O sistema deve permitir cadastrar, consultar, atualizar e inativar investidores, contas, carteiras e limites financeiros associados. | P0 | Cadastro só é concluído após validação de dados obrigatórios (CPF/CNPJ, documento, perfil de risco); inativação preserva histórico para auditoria. |
| RF-02 | Autenticação com MFA | O sistema deve autenticar o investidor com login/senha e um segundo fator (TOTP, SMS ou app autenticador) antes de liberar acesso a operações financeiras. | P0 | Acesso a rotas de ordens só é permitido após validação bem-sucedida do 2º fator; falha no MFA bloqueia a sessão e gera log de auditoria. |
| RF-03 | Consulta de carteira | O investidor autenticado deve poder consultar a composição atual de sua carteira (ativos, quantidades, preço médio, valor de mercado). | P0 | Dados exibidos refletem a última posição consolidada; discrepância entre carteira e ordens executadas é sinalizada. |
| RF-04 | Consulta de cotações | O sistema deve exibir cotações em tempo quase real, recebidas de provedor externo, para os ativos negociáveis. | P1 | Cotação exibida não pode ter defasagem maior que o SLA definido (ex.: 5s); indisponibilidade do provedor é sinalizada ao usuário. |
| RF-05 | Envio de ordem de compra | O investidor deve poder enviar ordens de compra especificando ativo, quantidade, tipo (mercado/limitada) e preço, quando aplicável. | P0 | Ordem só é aceita para processamento após passar pelas validações de RF-08; caso contrário é rejeitada com motivo explícito. |
| RF-06 | Envio de ordem de venda | O investidor deve poder enviar ordens de venda especificando ativo, quantidade, tipo e preço, quando aplicável. | P0 | Ordem só é aceita se houver posição suficiente em carteira (RF-08); rejeições registram motivo. |
| RF-07 | Cancelamento de ordem | O investidor deve poder solicitar o cancelamento de uma ordem ainda não executada. | P0 | Cancelamento só é efetivado se a ordem estiver em status cancelável; caso já executada/parcialmente executada, sistema informa o estado real. |
| RF-08 | Validação de saldo, posição e limites | Antes de transmitir qualquer ordem à Bolsa/Corretora simulada, o sistema deve validar saldo disponível, posição em carteira (para vendas), limites de risco do investidor e situação do mercado (aberto/fechado). | P0 | Nenhuma ordem é transmitida sem validação completa; falha em qualquer critério resulta em rejeição prévia, sem envio externo. |
| RF-09 | Acompanhamento do ciclo de vida da ordem | O sistema deve manter e expor o status da ordem em todas as suas transições (Recebida, Validada, Enviada, Executada, Parcialmente Executada, Rejeitada, Cancelada, Falha). | P0 | Toda transição de estado é registrada com timestamp e origem (usuário/sistema/bolsa); histórico de estados é consultável. |
| RF-10 | Notificações ao investidor | O sistema deve notificar o investidor sobre execução, rejeição, cancelamento ou falha de suas ordens. | P1 | Notificação é enviada em até X segundos após a mudança de status; falha no envio é reprocessada sem duplicar a notificação. |
| RF-11 | Consulta de histórico de operações | O investidor deve poder consultar o histórico completo de suas ordens e execuções, com filtros por período, ativo e status. | P1 | Histórico é imutável e reflete fielmente os registros de auditoria; resultados paginados para grandes volumes. |
| RF-12 | Auditoria | O sistema deve registrar logs de auditoria imutáveis para toda ação relevante (login, MFA, envio/cancelamento de ordem, mudança de status, decisões de validação). | P0 | Logs não podem ser alterados ou excluídos por usuários finais; cada registro contém quem, o quê, quando e resultado. |
| RF-13 | Integração com provedor de cotações | O sistema deve consumir cotações de um provedor externo de forma assíncrona/contínua. | P1 | Interrupção do provedor não derruba o sistema; última cotação válida é preservada com indicação de "desatualizada". |
| RF-14 | Integração com Bolsa/Corretora simulada | O sistema deve enviar ordens validadas à Bolsa/Corretora simulada e processar as respostas (confirmação, execução, rejeição). | P0 | Toda comunicação é registrada; respostas fora do padrão esperado são tratadas sem quebrar o fluxo. |

---

## 3. Requisitos Não Funcionais de Segurança (RNF-SEG)

| ID | Requisito | Descrição | Prioridade | Critérios de Aceite |
|---|---|---|---|---|
| RNF-SEG-01 | Senhas protegidas por hash | Senhas de investidores devem ser armazenadas usando função de hash forte e adaptativa (ex.: bcrypt, Argon2) com salt único por usuário. | P0 | Nenhuma senha em texto claro é persistida ou logada; verificação de fator de trabalho compatível com boas práticas atuais. |
| RNF-SEG-02 | Autenticação multifator (MFA) | Obrigatoriedade de segundo fator de autenticação para acesso a funcionalidades sensíveis (login e operações financeiras). | P0 | 100% dos logins exigem MFA; tentativa de bypass é bloqueada e auditada. |
| RNF-SEG-03 | Controle de acesso por perfil (RBAC) | O sistema deve implementar controle de acesso baseado em perfis (investidor, operador, administrador, auditor), restringindo funcionalidades e dados conforme o papel. | P0 | Usuário não consegue acessar funcionalidade/dado fora de seu perfil, mesmo manipulando requisições diretamente. |
| RNF-SEG-04 | Atributos privados / encapsulamento de dados | Dados sensíveis (saldo, documentos, posições) devem ser expostos apenas via interfaces controladas, nunca diretamente. | P1 | Modelagem (classes/DTOs) não expõe atributos sensíveis publicamente sem camada de controle; testes de acesso direto falham como esperado. |
| RNF-SEG-05 | Validação de entradas | Todas as entradas de usuário e de integrações externas devem ser validadas quanto a tipo, formato, intervalo e consistência de negócio. | P0 | Entradas inválidas são rejeitadas antes de qualquer processamento de negócio; erros são padronizados. |
| RNF-SEG-06 | Tratamento seguro de exceções | Exceções não devem expor detalhes internos (stack trace, dados sensíveis) ao usuário final; devem ser tratadas e logadas de forma controlada. | P0 | Respostas de erro ao usuário são genéricas; logs internos contêm detalhe técnico suficiente para diagnóstico. |
| RNF-SEG-07 | Prevenção de injeção | O sistema deve prevenir ataques de injeção (SQL, comando, NoSQL, etc.) por meio de consultas parametrizadas e sanitização. | P0 | Testes de penetração/análise estática não identificam vetores de injeção exploráveis. |
| RNF-SEG-08 | Proteção contra reenvio/duplicidade de ordem | O sistema deve prevenir o processamento duplicado de uma mesma ordem (double-submit, retry do cliente, falha de rede). | P0 | Reenvio de uma mesma requisição (mesmo idempotency key) não gera duas ordens nem duas execuções. |
| RNF-SEG-09 | Ausência de segredos no repositório | Chaves, senhas, tokens e credenciais não podem estar hardcoded no código-fonte ou versionadas no repositório. | P0 | Varredura de segredos (secret scanning) no pipeline não encontra credenciais expostas; segredos são geridos via cofre (vault/secret manager). |
| RNF-SEG-10 | Criptografia em trânsito e em repouso | Dados sensíveis devem trafegar sob TLS e ser armazenados de forma criptografada quando aplicável (dados financeiros, documentos). | P0 | Comunicação externa e interna crítica usa TLS 1.2+; dados sensíveis em banco possuem criptografia em repouso. |

---

## 4. Requisitos Não Funcionais de Resiliência (RNF-RES)

| ID | Requisito | Descrição | Prioridade | Critérios de Aceite |
|---|---|---|---|---|
| RNF-RES-01 | Time-out de integração | Toda chamada a serviços externos (provedor de cotações, Bolsa/Corretora simulada) deve ter tempo limite definido, evitando bloqueio indefinido. | P0 | Nenhuma chamada externa trava o processamento além do timeout configurado; timeout gera fallback/erro tratado. |
| RNF-RES-02 | Retentativa controlada | Falhas transitórias em integrações devem ser reprocessadas com política de retry (ex.: backoff exponencial, número máximo de tentativas). | P0 | Retentativas não geram efeitos colaterais duplicados (ver RNF-RES-03); número de tentativas e intervalos é configurável. |
| RNF-RES-03 | Idempotência | Operações críticas (envio de ordem, cancelamento, notificação) devem ser idempotentes, usando chave de idempotência única por requisição. | P0 | Reprocessar a mesma operação (mesma chave) não altera o estado além do efeito original nem gera duplicidade. |
| RNF-RES-04 | Fila de mensagens para processamento assíncrono | Envio de ordens, notificações e integrações externas devem ser processados via fila de mensagens, desacoplando produtor e consumidor. | P0 | Pico de mensagens não derruba o sistema; mensagens não processadas permanecem na fila (ou DLQ) para reprocessamento. |
| RNF-RES-05 | Modo de indisponibilidade segura (fail-safe) | Diante de falha de componente crítico (ex.: Bolsa indisponível), o sistema deve operar em modo degradado seguro, bloqueando novas ordens em vez de aceitá-las sem validação. | P0 | Em cenário de indisponibilidade simulada, nenhuma ordem é enviada sem validação completa; usuário é informado da indisponibilidade. |
| RNF-RES-06 | Recuperação de falhas | O sistema deve se recuperar automaticamente de falhas de instância/nó, sem perda de ordens em processamento. | P0 | Reinício de um nó/serviço não perde mensagens em fila nem deixa ordens em estado inconsistente. |
| RNF-RES-07 | Consistência de dados | O sistema deve garantir consistência entre saldo, posição em carteira e status da ordem, mesmo em cenários de falha parcial (consistência eventual controlada onde aplicável). | P0 | Testes de caos/falha parcial não resultam em saldo ou posição divergentes do histórico de execuções auditado. |

---

## 5. Requisitos Não Funcionais de Qualidade (RNF-QUAL)

| ID | Requisito | Descrição | Prioridade | Critérios de Aceite |
|---|---|---|---|---|
| RNF-QUAL-01 | Auditabilidade | Toda ação relevante do sistema deve ser rastreável a um usuário, sistema ou processo, com registro imutável. | P0 | Auditoria permite reconstruir a sequência completa de eventos de qualquer ordem, do envio à liquidação/rejeição. |
| RNF-QUAL-02 | Confidencialidade | Dados de investidores e operações devem ser acessíveis apenas a partes autorizadas. | P0 | Testes de acesso não autorizado a dados de terceiros falham conforme esperado. |
| RNF-QUAL-03 | Integridade | Dados financeiros (saldo, posição, ordens) não podem ser corrompidos ou alterados indevidamente em nenhuma etapa do processamento. | P0 | Mecanismos de validação/checksum ou transações garantem que dados persistidos correspondem às operações realizadas. |
| RNF-QUAL-04 | Disponibilidade | O sistema deve manter alta disponibilidade durante o pregão, com SLA definido (ex.: 99,9%). | P0 | Monitoramento demonstra atendimento ao SLA definido; indisponibilidades são registradas e justificadas. |
| RNF-QUAL-05 | Desempenho | O sistema deve processar validação e envio de ordens dentro de um tempo de resposta aceitável para operações de mercado. | P1 | Tempo de resposta da validação de ordem fica abaixo do limite definido (ex.: < 500ms) sob carga esperada. |
| RNF-QUAL-06 | Rastreabilidade | Requisitos, componentes e testes devem ser rastreáveis entre si, permitindo análise de impacto de mudanças. | P2 | Matriz de rastreabilidade (Seção 6) cobre 100% dos requisitos funcionais críticos. |
| RNF-QUAL-07 | Manutenibilidade | A arquitetura deve favorecer manutenção e evolução, com baixo acoplamento entre módulos (autenticação, ordens, cotações, auditoria, notificações). | P2 | Componentes podem ser modificados/implantados de forma independente, sem exigir deploy monolítico completo. |

---

## 6. Matriz de Rastreabilidade (resumo)

| Requisito Funcional | Requisitos Não Funcionais relacionados |
|---|---|
| RF-02 (Autenticação MFA) | RNF-SEG-01, RNF-SEG-02, RNF-QUAL-01 |
| RF-05 / RF-06 (Envio de ordens) | RNF-SEG-05, RNF-SEG-08, RNF-RES-01 a 04, RNF-QUAL-01, RNF-QUAL-03, RNF-QUAL-05 |
| RF-07 (Cancelamento) | RNF-RES-03, RNF-QUAL-01 |
| RF-08 (Validações) | RNF-SEG-05, RNF-RES-05, RNF-RES-07, RNF-QUAL-03 |
| RF-09 (Ciclo de vida da ordem) | RNF-RES-06, RNF-RES-07, RNF-QUAL-01, RNF-QUAL-06 |
| RF-12 (Auditoria) | RNF-QUAL-01, RNF-QUAL-02, RNF-QUAL-03, RNF-SEG-09 |
| RF-13 / RF-14 (Integrações externas) | RNF-RES-01 a 06, RNF-QUAL-04, RNF-QUAL-05 |

---

## 7. Premissas e Restrições

- A Bolsa/Corretora e o provedor de cotações são **simulados**, com contratos de API previamente definidos entre as equipes.
- O sistema deve ser projetado sob arquitetura **distribuída**, com componentes desacoplados por mensageria (ex.: fila/broker).
- Requisitos regulatórios específicos (ex.: CVM, LGPD) devem ser considerados como base para os requisitos de auditoria e confidencialidade, ainda que o cenário seja fictício.
- Este documento serve de base para a modelagem (casos de uso, diagramas de classes, componentes e sequência) nas etapas seguintes do projeto.

---

## 8. Próximos artefatos sugeridos

1. Diagrama de casos de uso (atores: Investidor, Operador, Administrador, Auditor, Sistema Externo de Cotações, Bolsa/Corretora Simulada).
2. Diagrama de classes (Investidor, Conta, Carteira, Ativo, Ordem, LimiteDeRisco, LogAuditoria, Notificação).
3. Diagrama de componentes (Gateway de Autenticação, Serviço de Ordens, Serviço de Cotações, Serviço de Auditoria, Serviço de Notificações, Fila/Broker, Integração Bolsa).
4. Diagrama de sequência do fluxo crítico: **Envio de Ordem de Compra** (do investidor à confirmação/rejeição).
5. Diagrama de estados do ciclo de vida da Ordem.
