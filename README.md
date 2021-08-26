# SAIMON
SAIMON - Secure Anycast Inside My Own Network

Qual é a Inicial Ideia do Projeto?

Um rede de nodos Anycast[1], de Servidores de DNS recursivo, Semi-abertos[2], Auto-Hospedados[3] por qualquer ASN do mundo que queira participar do Projeto[4].

A premissas do projeto:
- Entregar um serviço de DNS Recursivo:
   - Altamente resiliente
   - Altamente performático
   - Altamente seguro
   - Altamente respeitador da privacidade[5]


[1] Essa rede Anycast seria em dois níveis.
 - Internos, focado nas rede do ASN em que ele está hospeado.
 - Publicos, podendo atender como Fall-Back todos os ASNs que participarem do projeto.
[2] Semi-Aberto porque só estaria aberto para atender IPs dos ASNs que estiverem participando do projeto.
[3] Auto-Hospedado porque todo o recurso de hardware e rede seria provido pelo ASN Hospedeiro.
[4] Para participar do projeto, o ASN tem que se comprometer em estar em Compliance com seguranca(Talvez Manrs?).
[5] Preparar os mecanismos de controle e log de uma forma que inclusive não possibilitem análises de quais DNS-Queries foram feitas por quais IPs.


Como seria o modo operacional desse projeto?

Controle e Comando
------------------
Um head-end que controlará todos os nodos, coletará informações de telemetria(respeitanto privacidade).
(Muitas ideias aqui... Portal do participante, etc...)

Tipos de Nodos de serviço
-------------------------
 - INTERNO
   Preparada para atender como principal toda requisição que vier de IPs do ASN que está hospedando o nodo.
 - DFZ
   Preparada para atender como FallBack toda requisição que vier de IPs de ASNs do projeto.


Escopo da publicação dos serviços
---------------------------------
ExaBGP(ou similar) divulgando os Blocos IPs a partir de triggers internas dos nodo, e triggers do Controle e Comando.
- Se for um nodo da DFZ, divulgará os Blocos /24 v4 e /48 v6
- Se for um nodo INTERNO, divulgará os Blocos /32 v4 e /128 v6

Escopo do serviço de DNS
------------------------
Duas engines diferentes de DNS (Unbound pro primário, e alguma outra pro secundário) fazendo:
- DNS recursivo
- HyperLocal
- Suporte a DNSSEC(de maneira bastante estrita)
- Suporte a DNS Over TLS
- Suporte a DNS Over HTTPS
- DnsDist (load balancer e filtering)


Tipo de máquina hospedeira
--------------------------
 - Baremetal? (Eu acho muito trampo pensar nisso no primeiro momento...)
 - Virtualizada

Arquitetura de deploy dos serviços
----------------------------------
- Ambiente global rodando dentro de uma Virtual Machine
  (Já com o VirtualDisk Criptografado)
- Ambiente de Container rodando em cima da Virtual Machine
  - Cada serviço rodando em um Container
    - CallHome do Comando e Controle
    - Health Check do Virtual hardware
    - BGP
    - DNS(Talvez multiplos, por conta do tipo de resposta.)

Qual seriam o requisitos mínimos da máquina do tipo DFZ?
(ainda para pensar)

Qual seriam o requisitos mínimos da máquina do tipo INTERNO?

Processamento
-------------
- Pelo menos 4 cores, com suporte a SLAT/VT-X/EPT
  (dessa forma, se em algum momento for necessária virtualização aninhada, já estará disponível)

Memória
-------
- Pelo Menos 4GB, para evitar uso de disco

Disco
-----
- 64GB de disco, com um determinado IOPS Mínimo

Rede
----
- Duas interfaces de rede.
  - Uma para ser a dos "responders" dos serviços, respondendo pelos IPs Anycast.
    Essa precisa necessariamente fechar uma sessão BGP com o Core ou Border da rede.
  - Outra para receber pelo menos 1 IPv4 Publico e 1 IPv6 Público
    Essa pode ser apenas com IP estático, e default gateway.
  - NameSpaces diferentes para "Comando e Controle" e "Serviços".

Como seria o processo para o um ASN poder ser participante do projeto?
- Cadastro online.
- Validação de informações.
- Averiguação de compliance com segurança(Outrsourcing disso para MANRS?)
- Sinal de OK para Participante.

Como seria o processo para a ativação de um Nodo da rede de Anycast?
- Participante informa o tipo pretendido de nodo, e informa(concorda) com os requisitos mínimos.
- Geração automatizada de uma Imagem de maquina Virtual para o participante.
  (pensar se as informações de endereçamento serão embedadas na geração da VM)
  (criptografia individualizada do virtual-disk)
- Participante coloca a VM para rodar.
- Sistema se ativa, conecta no CallHome, e começa o primeiro Health-Check.
- Se Health-Check OK, Comando e Controle ordena deploy dos Containers de Serviço(BGP, DNS, etc.)

O que define se os serviços ficam ou não publicados?
- Health Check da máquina, com critérios mínimos de disponibilidade de:
  CPU, Memória(volume e velocidade), Disco (volume e velocidade).
- Comunicação fim-a-fim (v4 e v6) com o Comando e Controle, com "tokens de autorização para funcionamento" com períodos definidos.
- Comunicação plena com todos os DNS-Servers Autoritativos definidos(Ex.: Roots e TLDs)
- Sessão BGP com o Core/Border do ASN hospedeiro.


Ideias complementares

A) Com esse arcabouço técnico, não ficaria difícil entregar outros serviços como NTP.
   Mas isso é uma ideia para o futuro.

B) Solicitar um range de MAC-Address no IEEE-OUI para o projeto, e forçar os MACs de cada Interface.
   (só um a ideia maluca)

C) Aceitar consultas com IPs de Origem de uso privado(Ex.: CGNAT.) nos Nodos do tipo INTERNO

D) Comunicação entre Nodos do tipo INTERNO e Comando e controle pode ser por um IP do /24 v4 e /48 v6 dos Nodos do tipo DFZ. Funcionando como um Brocker.

E) IPs diferentes para tipos de respostas diferentes de DNS (Padrão, Filtro de Malware, Filtro de conteúdo Adulto).
   Como 1.1.1.2 e 1.1.1.3 da Cloudflare.


O que é necesário para o projeto acontecer?
1 - Uma instituição neutra e de nome forte(principalmente no escopo do DNS) no mundo assumir o papel de ser a gestora do projeto.
    Nomes possíveis: 
    - ICANN
    - NRO
    - ISOC (???Talvez vinculado ao MANRS???)
    - Team Cymru
    - RIRs (RIPE e APNIC tem uma quedinha por projetos assim)
2 - Recursos de numeração!
    - 1 x ASN 16 Bits
    - 2 x Blocos IPv4 /24 que permitam um número fácil de lembrar
    - 1 x Bloco IPv6 /32
3 - Criar o conjunto de software
    - Head-End / Comando e Controle
    - Portal central de interação com os Participantes(adesão e suporte)
    - Criador automágico de imagem de Virtual Machine
    - A Virtual Machine em similar
    - Os Containers de serviço
    - 
