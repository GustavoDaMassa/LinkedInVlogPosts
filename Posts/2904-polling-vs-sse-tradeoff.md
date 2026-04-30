---
status: escrito
contexto: Arquitetando notificações em tempo real em um sistema com eventos de baixa frequência. A pergunta era simples: como o frontend fica sabendo que algo aconteceu no backend sem o usuário recarregar a página?
objetivo: O leitor vai entender as duas abordagens principais para notificações em tempo real, a variável que decide entre elas, e sair com um princípio aplicável a qualquer sistema: sofisticação técnica deve ser proporcional ao problema.
---

# Polling vs SSE — quando a complexidade não se justifica

Estava arquitetando as notificações de um sistema quando a pergunta surgiu: como o frontend fica sabendo, em tempo real, que um evento aconteceu no backend?

Duas abordagens entraram na mesa. A análise que fiz me lembrou um princípio que vale repetir.

---

## O problema

O sistema precisa avisar o usuário quando um determinado evento ocorre no backend. O usuário está com o frontend aberto. Como a informação chega sem ele recarregar a página?

Vale entender primeiro por que o backend não pode simplesmente "mandar uma requisição" para o frontend. HTTP é um protocolo onde sempre é o cliente que inicia a conexão — o browser não é um servidor, não tem porta aberta, não tem IP fixo. O backend não tem como "ligar" para o browser por conta própria. É por isso que as abordagens abaixo existem: todas contornam essa limitação fazendo o cliente iniciar e manter a conexão de alguma forma.

---

## Abordagem 1 — Polling

O frontend pergunta ao backend periodicamente se há algo novo.

```
Frontend → GET /notifications (a cada 30s)
Servidor → "nada novo"
Frontend → GET /notifications (a cada 30s)
Servidor → "nada novo"
Frontend → GET /notifications (a cada 30s)
Servidor → "aqui está sua notificação"
```

**Como funciona:** requisição HTTP normal, repetida em intervalo fixo. O servidor responde e fecha a conexão. Stateless por natureza.

**Prós:**
- Implementação trivial — é só uma requisição HTTP com `setInterval`
- Stateless — o servidor não mantém nada entre as chamadas
- Funciona com qualquer infraestrutura sem configuração adicional
- Fácil de debugar, monitorar e escalar horizontalmente

**Contras:**
- A maioria das requisições retorna vazio — desperdício proporcional ao intervalo
- Atraso na notificação igual ao intervalo escolhido
- Com muitos usuários simultâneos, o volume de requisições cresce linearmente

---

## Abordagem 2 — SSE (Server-Sent Events)

O frontend abre uma conexão HTTP e fica escutando. O servidor empurra dados quando tiver algo novo.

```
Frontend → GET /notifications/stream (conexão fica aberta)
Servidor → (silêncio)
Servidor → (evento ocorre) "data: {tipo, mensagem}" → frontend recebe na hora
Servidor → (silêncio)
```

**Como funciona:** conexão HTTP persistente e unidirecional — só o servidor fala. Nativo no browser via `EventSource`. Diferente do WebSocket, não requer handshake especial nem protocolo separado.

**Prós:**
- Notificação instantânea — zero atraso entre o evento e a entrega
- Sem requisições desnecessárias — o servidor só fala quando tem algo
- Eficiente com alto volume de notificações
- Mais simples que WebSocket para casos unidirecionais

**Contras:**
- Conexão HTTP permanente — o servidor mantém uma conexão aberta por usuário conectado
- Requer integração com pub/sub interno (Redis, etc.) para que o worker que detectou o evento avise o servidor de API que está com a conexão do usuário aberta
- Limites de conexões abertas precisam de atenção na infra
- Mais complexo de debugar e monitorar

---

## A variável que decide

Ambas as abordagens resolvem o problema. O que determina a escolha correta é uma única variável: **a frequência dos eventos**.

| Frequência de eventos | Polling | SSE |
|---|---|---|
| Alta (mensagens, atualizações ao vivo) | Ineficiente — maioria das requests vazia | Ideal |
| Média (notificações ocasionais) | Aceitável com intervalo curto | Razoável |
| Baixa (alertas raros, relatórios) | Ideal | Complexidade desnecessária |

Para sistemas com eventos frequentes — um chat, um dashboard de métricas ao vivo, uma colaboração em tempo real — SSE (ou WebSocket para bidirecional) é claramente o caminho. O volume de eventos justifica manter a conexão aberta.

Para sistemas onde os eventos são raros — alertas pontuais, notificações que chegam algumas vezes por dia — polling com intervalo curto resolve com atraso imperceptível e zero complexidade adicional.

---

## O que aprendi na prática

No sistema que estava arquitetando, os eventos são raros por design — alguns por dia, no máximo. SSE manteria uma conexão aberta por usuário para entregar algo que acontece poucas vezes no dia. O custo de complexidade — integração com pub/sub, gestão de conexões abertas, monitoramento adicional — não entregava nenhum benefício real ao usuário.

Polling a cada 1 minuto: atraso máximo de 60 segundos numa notificação esperada em horas. Imperceptível.

A decisão foi polling — não por desconhecimento do SSE, mas por entender que a ferramenta certa é a proporcional ao problema.

---

## O princípio

> Sofisticação técnica deve ser proporcional à complexidade do problema.

Escolher SSE num sistema de baixa frequência não é "pensar no futuro" — é adicionar complexidade sem contrapartida real hoje. Quando o volume crescer e o polling se tornar ineficiente, a migração para SSE é localizada e justificada.

A pergunta certa não é "qual abordagem é mais moderna?" mas "qual escala com o meu volume de eventos?"
