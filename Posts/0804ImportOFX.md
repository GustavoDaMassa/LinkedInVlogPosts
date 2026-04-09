# Suporte a importação de extrato por OFX e dashBoards Dinâmicos. 

- 08/04/2026 escrita
- 16/04/2026 publicação

---

# Contexto 

A financeApi antes só suportava a criação de transações manualmente, ou por webhook, no entanto o webhook funciona só para ambiente sandbox, então um novo módulo foi a importação de transações 
através do extrato bancário em formato OFX que muitas bancos ainda têm suporte e que era a alternativa anterior ao Open Finance. junto com essa atualização vem a criação de dashboards dinâmicos 
em todos os filtros que o usuário realizar se assim ele quiser. 

--- 
# Objetivo 

Apresentar a nova funcionalidade, contextualizar sobre OFX e que o Open Finance está funcionando mas não em produção e apresentar os dashboards dinâmicos. 

--- 
# Post 

Open Financial Exchange - OFX

A financeAPI já suportava duas formas de registrar as suas transações: manualmente, ou de forma automática com o Open Finance via webhook.

No entanto a integração com o Open Finance funciona apenas no ambiente sandbox, em produção, exige aprovação do Banco Central, o que está fora do meu alcance atual.

A solução foi implementar o suporte a importação de extratos bancários em formato OFX,
que é um padrão de intercâmbio de dados financeiros que existia antes do Open Finance. A maioria dos bancos brasileiros ainda exporta nesse formato. 

O módulo de importação aceita o arquivo via upload, faz o parse (suportando OFX 1.x SGML e 2.x XML), e importa as transações automaticamente para a conta selecionada. Duplicatas são detectadas e ignoradas.

Junto com essa nova funcionalidade, os dashboards, antes estáticos e carregados no início de cada login, agora podem ser gerados a cada filtro que o usuário realizar. 

Assim ele consegue fazer uma análise mais dinâmica e personalizada de acordo com os filtros que ele preferir.

