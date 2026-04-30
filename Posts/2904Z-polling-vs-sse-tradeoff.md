---
status: escrito
contexto: Arquitetando notificações em tempo real em um sistema com eventos de baixa frequência.
objetivo: Provocar o leitor a questionar se a escolha técnica que está fazendo é proporcional ao problema, e direcionar para o blog com a análise completa.
---

Estava arquitetando as notificações de um sistema e precisei tomar uma decisão clássica:

Polling ou SSE?

O sistema precisa avisar o usuário quando um evento acontece no backend — sem ele recarregar a página.

Polling: o frontend pergunta ao servidor a cada X segundos "tem algo novo?". Simples, stateless, funciona com qualquer infra.

SSE: o frontend abre uma conexão e fica escutando. O servidor empurra a notificação assim que o evento ocorre. Zero atraso, zero requisição desnecessária.

No papel, SSE parece a escolha óbvia.

Mas na verdade, oque deve guiar o escolha 
é a frequência com que os eventos acontecem. Ignorar essa variável pode adicionar complexidade sem contrapartida real.

Escrevi uma análise completa com os prós e contras de cada abordagem - [link do blog]
