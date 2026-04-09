# FinAPI 

- 08/04/2026 escrita 
- 21/04/2026 publicação

---
# Contexto 

Resolvi reimplementar a financeAPI para colocar em prática Microserviços, era a aplicação que fazia mais sentido fazer isso, como exemplo. Aproveitei para mudar a stack e praticar novamente 
.Net com ASP.NET, e estudar Sistemas Distribuídos (Matéria na faculdade). 

---
# Objetivo

Mostrar que tenho conhecimentos equivalentes em .Net e que sei projetar sistemas que são microsservices.

---
# Post 

Microserviços

A financeAPI é o projeto onde mais venho evoluindo a cada dia. Atualmente estou estudando a matéria de Sistemas Distribuídos na faculdade e decidi reimplementá-la como microsserviços - era a aplicação que fazia mais sentido para esse exercício.

Dessa vez resolvi desenvolver em .Net com ASP.NET core e C#. A api tem injeção de dependência,  GraphQL, Kafka, JWT, OFX.

A arquitetura ficou dividida em 4 serviços:

- Identity — autenticação, geração e validação de JWT
- Finance — contas, transações, categorias, integração bancária, importação OFX, GraphQL
- Webhook — recebe eventos do Pluggy, publica no Kafka
- Gateway — ponto de entrada único, roteamento e propagação de identidade entre serviços

Cada serviço tem seu próprio banco de dados. A comunicação entre Finance e Webhook é assíncrona via Kafka. O Gateway propaga o usuário autenticado como header para os serviços internos — sem que eles precisem validar o JWT diretamente.

O resultado é um sistema onde cada peça pode evoluir, escalar ou falhar de forma independente.

No vídeo abaixo cada microsserviço está rodando em máquinas diferentes entre North Virgínia e Oregon na AWS. 

repositório : https://github.com/GustavoDaMassa/finAPIMS