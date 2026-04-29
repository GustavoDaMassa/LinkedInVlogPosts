---
status: escrito
contexto: Refinamento de requisitos de um projeto em desenvolvimento revelou que validar e verificar email são problemas diferentes, com abordagens e impactos distintos no design do sistema.
objetivo: Criar consciência sobre a distinção entre validação e verificação de email de forma direta, convidando devs a refletirem sobre o papel do email nas suas aplicações.
---

Validação vs Verificação

Refinando requisitos de um projeto, me deparei com a pergunta: "como garantimos que o email do usuário existe?"

Em primeiro momento a resposta pode parecer simples - validando o formato do email. Mas formato e existência são problemas diferentes.

Validação sintática verifica se está escrito certo. Verificação garante que o endereço existe e além disso, que o usuário tem acesso a ele.

A verdade é que ambas são válidas, a escolha certa depende de uma pergunta de produto, não técnica: qual é o papel do email nessa aplicação?

Existem algumas abordagens para essa questão, de uma regex simples até um fluxo completo de confirmação por email. Quando elas são necessárias? O usuário precisa ser bloqueado completamente? [link do blog]
