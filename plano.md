# Scout — Agente de Busca de Vagas
Visão Geral
Sistema multiagente que auxilia os usuários em sua jornada de desenvolvimento de carreira, combinando busca de empregos, identificação de lacunas de habilidades, recomendações de cursos e simulação de entrevistas.

Objetivo : Opção A do menu funciona. Agente busca vagas via Firecrawl, mostra resultados com correspondência de habilidades.

Pré-requisitos
Firecrawl instalado e configurado ( FIRECRAWL_API_KEYdefinido)

# Diretrizes para Modelos MoE
Todas as personas e habilidades são projetadas para modelos Mixture of Experts (MoE). Siga estas regras:

Sem instruções ambíguas. Cada passo deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de produção.
Sem tabelas markdown em nenhuma saída. Use listas numeradas com pares de chave-valor para dados estruturados.
Todos os caminhos de arquivo devem ser relacionados à raiz do projeto com prefixo explícito data/. A raiz do projeto é o diretório recoloca-ia/.
Se uma ferramenta falhar, o agente deverá relatar a falha no campo errose não continuar silenciosamente.
Nunca invente dados. Se um firecrawl searchou firecrawl scrapefalhar, reporte o erro exato e pare. Exceção : Em skills/course-analysis.md, se a busca falha para uma habilidade específica, tente o fallback; se o fallback também falhar, pule essa habilidade e continue com os restantes.
O agente NÃO deve escrever scripts Python, scripts de shell ou qualquer código para implementar personas. O agente personifica cada papel diretamente através do seu comportamento e respostas conversacionais. Personas são instruções comportamentais, não código a ser gerado.
# Arquitetura
┌─────────────────────────────────────────────────┐
│                  Usuário                          │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│              MAESTRO (Orquestrador)              │
│  - Interface principal com o usuário             │
│  - Coordena os agentes especializados            │
│  - Consolida resultados e apresenta ao usuário   │
└──┬──────────────┬──────────────┬────────────────┘
   │              │              │
   ▼              ▼              ▼
┌─────────┐  ┌──────────┐  ┌──────────────┐
│ SCOUT   │  │ CURATOR  │  │ COACH        │
│ (Busca  │  │ (Busca   │  │ (Simulação   │
│  de     │  │  de      │  │  de          │
│  Vagas) │  │  Cursos) │  │  Entrevistas)│
└─────────┘  └──────────┘  └──────────────┘
# Escopo : Implementar o bloco Scout e conectá-lo ao Maestro.

Estrutura de Diretórios
recoloca-ia/
├── AGENTS.md                   # (existente)
├── personas/
│   ├── maestro.md          # (existente)
│   └── scout.md            # NOVO: Agente de busca de vagas
├── skills/
│   ├── firecrawl.md        # (existente)
│   ├── dispatch.md         # (existente)
│   └── job-search.md       # NOVO: Capacidades de busca de vagas
└── data/
    ├── personality-quiz.md       # (existente)
    ├── user-profile.md           # (existente)
    └── job-search-results.md     # NOVO: Últimos resultados do Scout
Scout - Agente de Busca de Vagas
Responsabilidade : Buscar vagas de emprego via Firecrawl, que agrega resultados do Even, Catho, LinkedIn, Glassdoor, Infojobs e outras fontes.

Habilidades :

skills/job-search.md— Fluxo completo: comandos firecrawl, leitura de perfil, proteção de dados, correspondência de habilidades, filtragem por nível, formato de resposta e tratamento de erros. OBRIGATÓRIO CARREGAR.
skills/firecrawl.md— Comandos e regras do CLI Firecrawl.
Ferramentas do Zed :

terminal— executar comandos firecrawl searchefirecrawl scrape
Ferramentas de acesso web :

Sempre use firecrawl searchvia terminalcomo seu método primário de acesso à web
Se o firecrawl falhar consistentemente, você PODE usar curlou wgetcomo fallback — observe as limitações (sem renderização JS, possíveis bloqueios anti-bot, HTML bruto em vez de markdown limpo)
Evite Fetch, webfetchou outras ferramentas HTTP — prefira firecrawl primeiro, depois curl/wget apenas como recuperação
Fontes de Dados :

Busca Firecrawl (agrega Even, Catho, LinkedIn, Glassdoor, Infojobs)
Entradas :

Área de interesse, Localização, Nível de experiência, Habilidades atuais (de data/user-profile.md)
saídas :

Retornar resultados ao Maestro via Response Envelope (Maestro salva em data/job-search-results.md)
Exibir lista de até 5 vagas com título, empresa, localização, link, habilidades correspondentes/em falta e contagem de correspondência
Habilidades
busca-de-emprego.md
Ferramenta Zed : terminal— executa comandos CLI dofirecrawl
Descoberta de vagas :firecrawl search "vagas [area_de_interesse] [localizacao]" --json
Retorna JSON com: url, título, descrição, carga para cada resultado
Detalhes completos da vaga :firecrawl scrape <url> --format markdown
Usar em URLs individuais de vagas dos resultados da busca para obter descrição e requisitos completos
Se a deficiência ou expirar, use o título e a descrição do resultado da busca
Se a busca não retornar resultados, informe ao usuário e sugira ampliar os termos de busca
Para cada vaga encontrada, extraia: título, empresa (da URL ou título), localização, link, habilidades exigidas
Comparar habilidades exigidas com as habilidades atuais do usuário usando correspondência de strings sem distinção de conhecimentos/minúsculas
Se a vaga menciona um nível de experiência, prefira vagas que correspondam ao nível do usuário (Júnior, Pleno, Sênior). Se não houver vagas do nível correspondente nos primeiros resultados, expanda a busca e inclua vagas de nível adjacente anotando a discrepância.
Contar correspondências e listar as habilidades que faltam
colocarr até 5 anos
Protocolo de Handoff do Scout
Contexto passado: área de interesse e localização do usuário do user-profile.md

Envelope de Despacho (construído pelo Maestro):

## DESPACHO: SCOUT
### referencia_persona
[Conteúdo completo de personas/scout.md]

### tarefa
Buscar vagas de emprego para [area] em [localizacao]

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
Area: [area_de_interesse]
Localizacao: [localizacao]
Nivel: [nivel_de_experiencia]
Habilidades: [lista_de_habilidades]

### saida_esperada
Envelope de resposta com estado, resumo, dados (lista de vagas) e erros se houver
Formato de dados da resposta:

1. titulo: [título da vaga]
   empresa: [nome da empresa]
   localizacao: [cidade ou Remoto]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2]
   habilidades_faltantes: [habilidade3, habilidade4]
   contagem_correspondencia: [X de Y habilidades correspondem]

2. [próxima vaga no mesmo formato]
Esquemas dos Arquivos de Dados
dados/resultados-da-busca-de-emprego.md
Armazene os resultados mais recentes do Scout para uso pelo Curator e Coach:

Data da Busca: [AAAA-MM-DD HH:MM]
Parâmetros de Busca:
  Área: [valor]
  Localização: [valor]

Resultados:
1. titulo: [título da vaga]
   empresa: [nome da empresa]
   localizacao: [cidade ou Remoto]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2]
   habilidades_faltantes: [habilidade3, habilidade4]
   contagem_correspondencia: [X de Y habilidades correspondem]

2. titulo: [próximo título de vaga]
   ...
Tarefas
1. Escreverskills/job-search.md
Contendo:

Comandos firecrawl para busca de vagas
Como analisar resultados JSON
Como habilidades específicas do usuário com requisitos da vaga
Regras de filtragem por nível de experiência
Formato de resposta esperado
Tratamento de erros
2. Escreverpersonas/scout.md
Contendo:

Papel e responsabilidade do Scout
Ferramentas disponíveis (terminal com firecrawl)
Referência obrigatória à habilidadeskills/job-search.md
Referência à habilidade skills/firecrawl.md(existente)
Formato do Envelope de Resposta
regras de tratamento de erros
3. Conectar spawn_agentno Maestro para despachar o Scout
Quando o usuário digita "A", o Maestro deve construir o envelope de despacho
Chamar spawn_agentcom o prompt de despacho
Analisar a resposta do Scout
Salvar resultados emdata/job-search-results.md
Exibir ao usuário
menu
4. Testar: busca de vagas funcionais
Usuário A
Scout busca via Firecrawl
Resultados sorteados com habilidades correspondentes/em falta
5. Testar: tratamento de erro
Se a busca falhar, o erro é relatado ao usuário
Agente retorna ao menu
Fluxo
Usuário seleciona A no menu
        │
        ▼
Maestro constrói envelope de despacho para Scout
        │
        ▼
spawn_agent dispara Scout com persona + contexto
        │
        ▼
Scout executa firecrawl search → firecrawl scrape nas URLs
        │
        ▼
Scout corresponde habilidades com user-profile.md
        │
        ▼
Scout retorna envelope de resposta com até 5 vagas
        │
        ▼
Maestro salva em data/job-search-results.md → exibe ao usuário → retorna ao menu
de Erro
Se firecrawl searchfalhar, reportar o erro ao usuário e retornar ao menu.
Se firecrawl scrapefalhar em uma URL específica, use o título/descrição do resultado da busca e anote que a remoção falhou.
Se spawn_agentfalhar, diga ao usuário o que deu errado e retorne ao menu.
Se a busca retornar resultados mas algumas extrações falharam, mostrar resultados parciais com detalhes completos onde disponíveis.
Entregc
Scout totalmente funcional, busca de vagas end-to-end a partir do menu.
