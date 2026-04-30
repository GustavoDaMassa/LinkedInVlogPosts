---
status: escrito
contexto: Arquitetando a proteção de uma API REST e percebendo o erro clássico de proteger os endpoints errados.
objetivo: Provocar o leitor a revisar onde está aplicando rate limiting na própria API.
---

A maioria das APIs tem rate limiting nos endpoints autenticados.

E esquece que login e registro não têm JWT ainda.

São exatamente os mais vulneráveis e ficam sem proteção.

Rate limiting por IP protege endpoints públicos. É o único identificador disponível antes da autenticação.

Rate limiting por usuário protege endpoints autenticados. Não é contornável com troca de IP.

Nenhum dos dois sozinho é suficiente. A escolha certa é aplicar cada um onde faz sentido.

Tem ainda uma diferença que determina se um atacante consegue burlar o limite no exato momento em que a janela vira.

[link do blog]
