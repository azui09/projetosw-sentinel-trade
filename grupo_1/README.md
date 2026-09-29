# SentinelTrade

Especificação e modelagem de uma plataforma de negociação de ativos financeiros (ações, ETFs e fundos imobiliários) para a corretora fictícia Orion Capital.

## Integrantes

| Nome | RA |
|---|---|
| Felipe Kosorus | 10737605 |
| Enzo Grutila De Oliveira | 10737003 |
| João Vitor Santana Silveira | 10738220 |

## Sumário

- [Visão do projeto](#visão-do-projeto)
- [Contexto](#contexto)
- [Requisitos do sistema](#requisitos-do-sistema)

---

## Visão do projeto

Trade é a operação de compra e venda de ativos financeiros em mercados organizados ou em plataformas de negociação. No mercado de capitais, o termo normalmente se refere à negociação de instrumentos como ações, fundos imobiliários (FIIs), ETFs, títulos, derivativos e outros ativos disponibilizados por corretoras e bolsas de valores.

O objetivo de uma operação de trade pode ser investir recursos no médio e longo prazo, formar patrimônio, obter renda por proventos ou buscar resultado financeiro a partir das oscilações de preço dos ativos. A compra ocorre quando o investidor espera valorização ou deseja compor sua carteira; a venda pode ocorrer para realizar lucro, reduzir exposição, atender a uma necessidade de liquidez ou limitar perdas.

Uma plataforma de trade conecta o investidor à corretora e, por intermédio dela, aos ambientes de negociação. Ela permite consultar cotações, analisar ativos, visualizar saldo e posição em carteira, enviar ordens de compra e venda, acompanhar sua execução e manter o histórico das operações realizadas.

### Principais funcionalidades de um sistema de trade

- Consulta de preços, variações e informações dos ativos
- Gestão de conta, saldo financeiro e carteira de investimentos
- Envio de ordens de compra, venda, alteração e cancelamento
- Validação de saldo, quantidade disponível, preço, horário de negociação e limites de risco
- Acompanhamento do estado da ordem: criada, validada, enviada, parcialmente executada, executada, rejeitada ou cancelada
- Registro de operações e cálculo de posição financeira do investidor
- Notificações sobre execução, rejeição, oscilação relevante ou indisponibilidade
- Controles de autenticação, autorização, auditoria e rastreabilidade
- Integração com provedores de cotações, corretoras, bolsas e serviços de liquidação

### Por que confiabilidade é crítica

Em razão de seu impacto financeiro e regulatório, um sistema de trade exige elevada confiabilidade. Uma ordem duplicada, uma cotação incorreta, a ausência de validação de saldo ou a falha na auditoria podem causar prejuízos ao investidor, à corretora e à integridade do mercado. Por isso, plataformas dessa natureza precisam adotar mecanismos de:

- Segurança
- Consistência de dados
- Prevenção a fraudes
- Tolerância a falhas
- Registro auditável de todas as operações relevantes

---

## Contexto

A corretora fictícia **Orion Capital** opera uma plataforma digital de negociação de ações, ETFs e fundos imobiliários. Atualmente, suas operações dependem de sistemas pouco integrados, dificultando a rastreabilidade das ordens, o controle de risco e a auditoria.

A Orion Capital contratou nossa equipe para especificar e modelar o **SentinelTrade**, plataforma que deverá atender aos requisitos descritos abaixo.

---

## Requisitos do sistema

O SentinelTrade deverá:

1. Autenticar usuários com MFA (autenticação multifator)
2. Manter dados de investidores, contas, carteiras, ativos e limites financeiros
3. Receber cotações em tempo quase real por meio de um provedor externo
4. Permitir ordens de compra, venda, cancelamento e consulta
5. Validar saldo, posição em carteira, limite de risco e situação do mercado antes de transmitir a ordem
6. Integrar-se a uma Bolsa/Corretora simulada
7. Acompanhar o ciclo de vida das ordens
8. Registrar logs de auditoria imutáveis
9. Notificar o investidor sobre execução, rejeição, cancelamento ou falha
10. Operar com mecanismos de recuperação, indisponibilidade controlada e prevenção de duplicidade de ordens
