 # Home Server 
 - 07/04/2026 escrita
 - 08/04/2026 publicação

 --- 

 # Contexto: 

 Home server é meu servidor pessoal que Contrui com uma máquina antiga que tinha em casa, para minimizar custos com plataformas como a AWS e mesmo assim disponibilizar minhas aplicações pela web
 o objetivo não era um servidor profissional, mas sim um amador que resolve com eficiência o meu cenário, ele é resiliente a falhas como quedas de energia e atualizações das aplicações.
 ele é uma solução temporaria inclusive estudo e provisiono em plataformas em nuvem. 

 ---
 # Objetivo:  

Comentar sobre o servidor pessoal que construi como uma forma de resolver oque precisava com as ferramentas a disposição, entrar um pouco na questão de preocupação com reiniciações e falhas,
configurações e acesso remoto, citar watchtower, ssh, cloudflared tunnel e CGNAT. 

---
# Post : 

Quero apenas comentar uma ideia que tive recentemente, em que fugi um pouco do padrão de soluções. 

Na área de desenvolvimento temos várias plataformas onde podemos colocar nossas aplicações em produção. Utilizo a AWS e a Oracle Cloud, acontece que o plano free tier tem algumas limitações e eu não tinha a pretensão de aumentar o meu custo por enquanto.

tenho uma máquina antiga em casa - 16GB de RAM, resolvi configurá-la como meu próprio servidor pessoal de produção. 

Foi um desafio interessante, encontrei alguns obstáculos, o maior deles foi o CGNAT da operadora - sem IP público próprio, impossível abrir portas diretamente. A solução foi o Cloudflare Tunnel: uma conexão de saída do servidor para a Cloudflare, sem nenhuma porta aberta no roteador. Acesso remoto via SSH também passa pelo Cloudflare Tunnel. Funciona de qualquer lugar, sem VPN. 
E para deploy, uso Watchtower - um serviço que monitora o Docker Hub e atualiza o container assim que uma nova imagem é publicada.

O objetivo não era um servidor profissional, mas sim um amador que resolve com eficiência o meu cenário, é uma solução temporária onde consigo ter controle total, aproveitar o espaço em memória e disponibilizar com folga minhas aplicações pela web.

Documentação detalhada : https://github.com/GustavoDaMassa/HomeServer