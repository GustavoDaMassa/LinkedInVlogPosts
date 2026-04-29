---
status: escrito
contexto: Refinamento de requisitos de um projeto em desenvolvimento com alertas por email. A pergunta "como garantimos que o email do usuário existe?" revelou que validação e verificação são problemas diferentes, com impactos diretos no modelo de dados e nos use cases.
objetivo: O leitor vai entender a diferença entre validar e verificar um email, conhecer as abordagens com detalhes técnicos e trade-offs reais, e saber como a decisão afeta o design do sistema.
---

# Validar email não é verificar email

Durante o refinamento de requisitos de um projeto que estou desenvolvendo, surgiu uma pergunta aparentemente simples:

> "Como garantimos que o email do usuário existe?"

A resposta instintiva é: "validando o formato". Mas formato e existência são dois problemas completamente diferentes. Confundir os dois é uma das formas mais silenciosas de quebrar uma feature em produção.

---

## Validação vs verificação

**Validação** responde: o email está no formato correto?

Isso é resolvido com uma regex ou anotação do framework (`@Email` no Bean Validation, `[EmailAddress]` no .NET, etc.). Custo zero. Mas não diz nada sobre se o endereço existe ou pertence a alguém.

**Verificação** responde: esse email pertence a alguém que tem acesso a ele?

Essa é a pergunta que realmente importa quando o email é o canal principal de comunicação do sistema.

---

## As abordagens

### 1. Validação sintática

Verifica apenas o formato via regex ou biblioteca. É o mínimo que qualquer sistema deveria fazer.

```
usuario@dominio.com   → válido
usuario@              → inválido
@dominio.com          → inválido
usuario@dominio       → depende da regex — RFC 5321 tecnicamente permite
```

A RFC 5321 define o formato completo de um endereço de email. A maioria das implementações usa um subconjunto — endereços tecnicamente válidos pela RFC como `"usuario com espaço"@dominio.com` raramente são aceitos na prática.

**Quando usar:** email é dado secundário — um campo de contato opcional, por exemplo.

**Por que não fecha o problema:** `qualquercoisa@inventado.xyz` passa na validação. O usuário cadastra, nunca recebe nada, e o sistema parece funcionar — mas não funciona.

---

### 2. Verificação de MX record

Consulta o DNS do domínio e verifica se existe um registro MX (Mail Exchanger) — ou seja, se há um servidor configurado para receber emails naquele domínio.

```
gmail.com        → MX: smtp.google.com → aceita
dominiofalso.xyz → sem MX             → rejeita
```

**Prós:** elimina domínios claramente inválidos sem enviar nenhum email; latência baixa (dezenas de ms), pode ser feito de forma síncrona antes de persistir o cadastro.

**Contras:** não verifica a caixa postal. `fantasma99999@gmail.com` passa — `gmail.com` tem MX válido, mas a caixa específica pode não existir. Domínios corporativos também podem bloquear ou responder de forma inconsistente a consultas externas.

---

### 3. Verificação SMTP (RCPT TO)

Abre uma conexão SMTP com o servidor de destino e executa o comando `RCPT TO` sem de fato enviar nenhum email — só para verificar se o servidor aceita entregas para aquele endereço.

```
EHLO verificador.com
MAIL FROM: <verify@verificador.com>
RCPT TO: <usuario@dominio.com>   ← 250 OK = caixa existe (em teoria)
QUIT
```

**Prós:** teoricamente verifica a caixa postal, não apenas o domínio.

**Contras sérios:**
- A maioria dos grandes provedores (Gmail, Outlook, Yahoo) retorna `250 OK` para qualquer endereço — independente de a caixa existir. Fazem isso para evitar enumeração de endereços.
- Servidores corporativos costumam bloquear a conexão ou retornar erros genéricos.
- Sua aplicação pode ser sinalizada como spam por tentar conexões SMTP em volume.

Na prática, não é confiável o suficiente para uso em produção.

---

### 4. Confirmação por email

Ao registrar, o sistema gera um token único com prazo de expiração, envia um email com o link de confirmação e aguarda o clique para confirmar.

```
Cadastro
  → gera UUID token + define expiração (ex: 24h)
  → persiste: isEmailVerified=false, token, expiresAt
  → envia email com link contendo o token
  → usuário clica
      → backend valida token + expiração
      → marca email como verificado
      → limpa token e expiresAt
```

**Prós:** única abordagem que garante que o email existe *e* que o usuário tem acesso a ele. Fecha o loop completamente.

**Contras:** adiciona uma etapa ao onboarding; requer lógica de expiração, reenvio e tratamento de tokens inválidos.

O caso de reenvio é frequentemente esquecido no design inicial — mas é essencial: emails caem em spam, tokens expiram, usuários erram o endereço no cadastro e precisam corrigir.

---

## Como a decisão afeta o design

Quando o email é canal crítico e a escolha é pela confirmação, o impacto vai além de um campo a mais no banco:

**Modelo de dados** — o `User` ganha campos para controlar o estado da verificação: a flag booleana, o token temporário e sua expiração. Após a confirmação, token e expiração são limpos.

**Use cases** — surgem pelo menos três novos fluxos: registro (gera e envia o token), verificação (valida e ativa), reenvio (invalida o anterior e gera um novo).

**Regras de negócio** — é preciso decidir o que um usuário não verificado pode ou não fazer. Uma abordagem comum é permitir o login mas restringir as features que dependem do email até a verificação ser concluída.

---

## O ponto central

A pergunta "como validamos o email?" parece técnica. Na verdade é uma pergunta de produto: *qual é o papel do email nessa aplicação?*

Se é dado secundário, validação sintática resolve. Se é canal crítico, confirmação por email é o único caminho que fecha o loop com garantia real — e a decisão muda o modelo de dados, os use cases e as regras de negócio antes de você escrever uma linha de código de feature.
