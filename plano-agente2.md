# Plano de Implementação do Agente Scout (Buscador de Vagas) — plano-agente2.md

Baseado no [plano.md](file:///home/livia/zed.IA/plano.md) original, este plano foca exclusivamente na criação do agente **Scout**, responsável por buscar vagas de emprego, utilizando o Firecrawl CLI como método primário e a ferramenta de acesso web nativa do Zed (`fetch`) como fallback.

## Visão Geral
O Scout é o agente especializado em busca de vagas do sistema multiagente de recolocação profissional. Ele deve priorizar o uso do Firecrawl CLI (que agrega resultados de Infojobs, Vagas.com, Indeed, LinkedIn, Glassdoor, Catho, entre outros), e recorrer à ferramenta de acesso web nativa apenas se o Firecrawl falhar consistentemente.

## Pré-requisitos
- Firecrawl CLI instalado e configurado (variável `FIRECRAWL_API_KEY` definida)
- Ferramenta de acesso web nativa do Zed (`fetch`) disponível
- Arquivos de perfil do usuário existentes em `data/user-profile.md`

## Estrutura de Diretórios (Atualizada)
```
zed.IA/
├── plano.md                   # Plano original
├── plano-agente2.md           # NOVO: Este plano (Scout)
├── AGENTS.md                  # Regras do projeto
├── personas/
│   ├── maestro.md             # Orquestrador (existente)
│   └── scout.md               # NOVO: Persona do Scout
├── skills/
│   ├── firecrawl.md           # Comandos Firecrawl (existente)
│   ├── dispatch.md             # Protocolo de handoff (existente)
│   └── job-search.md          # NOVO: Procedimento de busca de vagas (contém o fluxo completo)
└── data/
    ├── personality-quiz.md     # (existente)
    ├── user-profile.md         # Perfil do usuário (existente)
    └── job-search-results.md   # NOVO: Resultados das buscas do Scout
```

## Especificações do Agente Scout
### Responsabilidade
Buscar vagas de emprego em sites como **Infojobs, Vagas.com, Indeed** (além de outras fontes agregadas pelo Firecrawl), comparar requisitos com as habilidades do usuário e retornar até 5 resultados estruturados.

### Ferramentas Utilizadas
1. **Primária**: `terminal` do Zed para executar comandos Firecrawl CLI (`firecrawl search`, `firecrawl scrape`)
2. **Fallback**: Ferramenta de acesso web nativa (`fetch`) para buscar diretamente nos sites alvo se o Firecrawl falhar
3. **Leitura de Dados**: `read_file` para acessar `data/user-profile.md`

### Fontes de Dados
- Firecrawl (agrega Infojobs, Vagas.com, Indeed, LinkedIn, Glassdoor, Catho)
- Fallback direto: URLs específicas de Infojobs, Vagas.com, Indeed (via `fetch`)

## Procedimento de Busca (Armazenado em `skills/job-search.md`)
O procedimento completo de busca deve ser documentado em `skills/job-search.md` e incluir:
1. **Passo 1: Leitura do Perfil do Usuário**
   - Ler `data/user-profile.md` para obter: área de interesse, localização, nível de experiência, lista de habilidades atuais.
2. **Passo 2: Busca Primária com Firecrawl**
   - Executar: `firecrawl search "vagas [area] [localizacao]" --json`
   - Se o comando retornar erro ou nenhum resultado, prosseguir para o fallback.
3. **Passo 3: Fallback com Ferramenta Web Nativa**
   - Se Firecrawl falhar, usar `fetch` para acessar URLs de busca direta:
     - Infojobs: `https://www.infojobs.com.br/vagas-de-emprego/[area]-[localizacao].html`
     - Vagas.com: `https://www.vagas.com.br/vagas/[area]/[localizacao]`
     - Indeed: `https://www.indeed.com.br/vagas?q=[area]&l=[localizacao]`
   - Extrair títulos, empresas, localizações e links das páginas retornadas.
4. **Passo 4: Extração de Detalhes da Vaga**
   - Para cada vaga encontrada, usar `firecrawl scrape <url> --format markdown` (ou `fetch` no fallback) para obter descrição completa e requisitos.
5. **Passo 5: Correspondência de Habilidades**
   - Comparar requisitos da vaga com as habilidades do usuário (case-insensitive string matching).
   - Contar habilidades correspondentes e faltantes.
   - Filtrar por nível de experiência (priorizar o nível do usuário, incluir níveis adjacentes se necessário).
6. **Passo 6: Formatação da Resposta**
   - Retornar até 5 vagas no formato de lista numerada com pares chave-valor (sem tabelas markdown):
     ```
     1. titulo: [título]
        empresa: [empresa]
        localizacao: [localização]
        link: [url]
        habilidades_correspondentes: [lista]
        habilidades_faltantes: [lista]
        contagem_correspondencia: [X de Y]
     ```
7. **Passo 7: Tratamento de Erros**
   - Se Firecrawl falhar: registrar erro no campo `erro` do envelope de resposta, tentar fallback.
   - Se fallback também falhar: relatar erro e parar.
   - Se nenhum resultado for encontrado: informar usuário e sugerir ampliar termos de busca.

## Persona do Scout (`personas/scout.md`)
Conter:
- Papel e responsabilidades
- Referência obrigatória a `skills/job-search.md` e `skills/firecrawl.md`
- Regras de tratamento de erros
- Formato do envelope de resposta

## Fluxo de Integração com o Maestro
1. Usuário seleciona opção "A" no menu
2. Maestro constrói envelope de despacho com contexto do `user-profile.md`
3. Maestro usa `spawn_agent` para disparar o Scout com a persona e contexto
4. Scout executa o procedimento em `skills/job-search.md`
5. Scout retorna envelope de resposta com resultados ou erros
6. Maestro salva resultados em `data/job-search-results.md`, exibe ao usuário e retorna ao menu

## Tarefas de Implementação
1. Criar `skills/job-search.md` com o procedimento completo de busca (incluindo Firecrawl primário e fallback nativo)
2. Criar `personas/scout.md` com a persona do agente
3. Atualizar o Maestro (`personas/maestro.md`) para integrar o disparo do Scout via `spawn_agent`
4. Testar busca funcional com Firecrawl
5. Testar fallback com ferramenta web nativa
6. Testar tratamento de erros (falha no Firecrawl, falha no fallback, nenhum resultado)

## Critérios de Aceite
- Scout busca vagas prioritariamente via Firecrawl CLI
- Scout usa fallback nativo (`fetch`) se Firecrawl falhar
- Resultados incluem vagas de Infojobs, Vagas.com, Indeed
- Até 5 vagas retornadas com correspondência de habilidades
- Erros são relatados corretamente sem dados inventados
