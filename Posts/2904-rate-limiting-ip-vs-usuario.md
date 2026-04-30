---
status: escrito
contexto: Arquitetando a proteção de uma API REST e precisando decidir onde e como aplicar rate limiting — a decisão que a maioria erra por proteger os endpoints errados.
objetivo: O leitor vai entender a diferença entre limitar por IP e por usuário, por que nenhum dos dois sozinho é suficiente, onde cada um deve ser aplicado, e os conceitos de janela que determinam a efetividade da proteção.
---

# Rate limiting — por IP, por usuário, ou os dois?

Estava definindo a estratégia de rate limiting de uma API quando percebi um erro clássico que muitos cometem.

A maioria protege os endpoints autenticados — os que têm JWT — e esquece os públicos.

O problema: **login e registro não têm JWT ainda. E são exatamente os mais vulneráveis.**

---

## Por que rate limiting existe

Sem limite de requisições, qualquer endpoint fica exposto a:

- **Força bruta** — testar milhares de combinações de senha no login
- **Abuso de recursos** — criar contas em massa, spam de requisições
- **DDoS simples** — sobrecarregar o servidor com volume

Rate limiting define um teto: X requisições por janela de tempo. Acima disso, a API responde 429 Too Many Requests.

---

## Opção 1 — Limitar por IP

Conta requisições pelo endereço IP de origem, independente de autenticação.

**Quando protege bem:**
- Endpoints públicos — login, registro, recuperação de senha
- Ataques de força bruta antes da autenticação

**Onde falha:**
- IPs são compartilhados — um escritório inteiro sai pelo mesmo IP. Um usuário legítimo pode ser bloqueado por culpa de outro
- Fácil de contornar com VPN ou proxy — trocar de IP resolve o bloqueio

**Detalhe importante:** quando a API está atrás de um proxy, nginx ou load balancer, o IP que chega ao servidor é o do proxy — não o do cliente real. O IP real vem no header `X-Forwarded-For`. Usar o IP errado significa que todos os usuários compartilham a mesma cota, inutilizando a proteção.

---

## Opção 2 — Limitar por usuário autenticado

Conta requisições pelo ID do usuário extraído do JWT.

**Quando protege bem:**
- Endpoints autenticados — evita abuso de API por usuários individuais
- Não é contornável com troca de IP — o identificador é o usuário, não o endereço

**Onde falha:**
- Não existe JWT antes da autenticação — login e registro ficam desprotegidos
- O atacante que ainda não tem conta não é "usuário" para o sistema

---

## A decisão certa: os dois, em lugares diferentes

Não é escolher um ou outro — é aplicar cada um onde faz sentido.

**Endpoints públicos → rate limiting por IP**
```
POST /auth/login        → máx. 10 tentativas por IP a cada 15 minutos
POST /auth/register     → máx. 5 registros por IP por hora
POST /auth/resend-email → máx. 3 reenvios por IP por hora
```

**Endpoints autenticados → rate limiting por usuário**
```
GET  /products          → máx. 60 requisições por usuário por minuto
POST /products          → máx. 20 criações por usuário por hora
```

---

## Fixed window vs Sliding window

São duas formas diferentes de contar a janela de tempo — e a diferença afeta diretamente a efetividade da proteção.

**Fixed window (janela fixa)**
A janela começa no início de cada período fixo — minuto 00, minuto 01, etc. Um usuário pode fazer 10 requisições nos últimos 5 segundos do minuto e mais 10 nos primeiros 5 segundos do próximo, totalizando 20 em 10 segundos sem violar o limite.

```
|--- minuto 1 ---|--- minuto 2 ---|
          [10 req][10 req]
                   ^ 20 requisições em 10s passam pelo limite
```

**Sliding window (janela deslizante)**
A janela sempre olha para os últimos X segundos a partir do momento atual. Não importa a fronteira do minuto — o limite é sempre verificado em relação ao passado recente.

```
momento atual → olha os últimos 60s → conta as requisições nesse período
```

Mais precisa, mais justa, e sem a brecha do boundary. O custo é levemente maior em termos de armazenamento — precisa guardar o timestamp de cada requisição em vez de apenas um contador.

Para proteção de login e endpoints críticos, sliding window é a escolha certa. Para endpoints de uso geral com limites generosos, fixed window é suficiente e mais simples.

---

## O erro que acontece na prática

A proteção por JWT parece completa. O desenvolvedor configura rate limiting nos endpoints que "importam" — os que manipulam dados, os que têm lógica de negócio.

Login fica sem proteção porque "é só um endpoint simples".

É exatamente aí que o atacante entra: testando combinações de senha sem nenhum obstáculo, requisição após requisição, até acertar.

---

## O princípio

> Proteja o acesso antes de proteger o recurso.

Rate limiting por usuário pressupõe que o usuário já existe e já está autenticado. Para tudo que acontece antes disso, só o IP está disponível — e precisa ser usado.

A camada de proteção correta depende do que você sabe sobre quem está fazendo a requisição naquele momento.
