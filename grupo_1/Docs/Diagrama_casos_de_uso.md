# Diagrama de Casos de Uso — SentinelTrade


## Atores

| Ator | Tipo | Descrição |
|---|---|---|
| Investidor | Ator principal | Usuário final da plataforma. Envia ordens, consulta cotações, gerencia carteira e acompanha o histórico de suas operações |
| Administrador | Ator interno | Responsável por cadastros, definição de perfis de acesso e gestão de contas/limites financeiros |
| Operador de Risco | Ator interno | Monitora e gerencia limites de risco associados às contas |
| Auditor | Ator interno | Consulta a trilha de auditoria, sem poder de alteração sobre os registros |
| Provedor de Cotações | Sistema externo | Envia cotações de ativos em tempo quase real para o SentinelTrade |
| Bolsa/Corretora Simulada | Sistema externo | Recebe as ordens transmitidas pelo sistema e retorna confirmações de execução |

## Casos de uso

| Caso de uso | Categoria | Ator(es) | Relacionamento | Descrição |
|---|---|---|---|---|
| Cadastrar investidor | Autenticação e Acesso | Administrador | Inclui: Registrar log de auditoria | Cadastro de novo investidor na plataforma |
| Autenticar com MFA | Autenticação e Acesso | Investidor, Administrador, Operador de Risco, Auditor | Inclui: Registrar log de auditoria | Login com múltiplo fator |
| Bloquear acesso após tentativas inválidas | Autenticação e Acesso | — | Estende: Autenticar com MFA · Inclui: Registrar log de auditoria | Bloqueio automático após tentativas de login inválidas |
| Definir perfil de acesso | Autenticação e Acesso | Administrador | Inclui: Registrar log de auditoria | Configuração de perfis e permissões de acesso |
| Gerenciar contas e limites financeiros | Cadastro e Dados | Administrador, Operador de Risco | Inclui: Registrar log de auditoria | Gestão de contas, saldo e limites de risco |
| Gerenciar carteira de ativos | Cadastro e Dados | Investidor | Inclui: Registrar log de auditoria | Gestão da carteira de ativos do investidor |
| Receber cotações | Cotações | Provedor de Cotações | — | Recebimento de cotações em tempo quase real |
| Consultar cotações | Cotações | Investidor | — | Visualização das cotações recebidas |
| Enviar ordem (compra/venda) | Ordens | Investidor | Inclui: Validar ordem, Notificar investidor, Transmitir ordem à Bolsa/Corretora | Envio de ordem de compra ou venda de ativos |
| Validar ordem | Ordens | — | Incluído por: Enviar ordem | Validação de saldo, posição, limite de risco e situação do mercado |
| Notificar investidor | Ordens | Investidor | Incluído por: Enviar ordem | Notificação sobre execução, rejeição, cancelamento ou falha |
| Transmitir ordem à Bolsa/Corretora | Ordens | Bolsa/Corretora Simulada | Incluído por: Enviar ordem, Cancelar ordem · Estendido por: Detectar indisponibilidade e acionar contingência | Transmissão da ordem ao ambiente de negociação externo |
| Cancelar ordem | Ordens | Investidor | Inclui: Registrar log de auditoria, Transmitir ordem à Bolsa/Corretora | Cancelamento de ordem ainda não executada |
| Consultar status e histórico de ordens | Ordens | Investidor | — | Consulta do estado atual e histórico de ordens |
| Detectar indisponibilidade e acionar contingência | Resiliência | — | Estende: Transmitir ordem à Bolsa/Corretora | Detecção de falha de comunicação e acionamento de contingência |
| Registrar log de auditoria | Auditoria | — | Incluído por: Cadastrar investidor, Autenticar com MFA, Bloquear acesso, Definir perfil de acesso, Gerenciar contas e limites financeiros, Gerenciar carteira de ativos, Cancelar ordem | Registro imutável de ações críticas do sistema |
| Consultar trilha de auditoria | Auditoria | Auditor | — | Consulta completa do histórico de auditoria |
