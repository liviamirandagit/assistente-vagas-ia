# Habilidade: Busca de Vagas (Job Search)

Este documento descreve o procedimento completo que o Scout deve seguir para buscar vagas de emprego, comparar habilidades e retornar resultados estruturados.

## Fluxo de Execução

### Passo 1: Leitura do Perfil do Usuário
1. Usar `read_file` para ler `data/user-profile.md`
2. Extrair os seguintes campos obrigatórios:
   - `Area de interesse`: Ex: "Desenvolvimento de Software"
   - `Localizacao`: Ex: "São Paulo, SP"
   - `Nivel de experiencia`: Ex: "Pleno"
   - `Habilidades`: Lista completa de habilidades técnicas

### Passo 2: Busca Primária com Firecrawl
1. Construir a query de busca: `"vagas [area_de_interesse] [localizacao]"`
2. Executar via `terminal`:
   ```bash
   firecrawl search "vagas [area_de_interesse] [localizacao]" --json
   ```
3. **Tratamento de sucesso**: Parsear o JSON retornado (array de objetos com `url`, `titulo`, `descricao`, `empresa`, `carga`)
4. **Tratamento de falha**: Se o comando retornar erro ou nenhum resultado, prosseguir para o Passo 3 (Fallback)

### Passo 3: Fallback com Ferramenta Web Nativa (fetch)
Se o Firecrawl falhar consistentemente (3 tentativas), usar `fetch` para acessar URLs diretas:
- Infojobs: `https://www.infojobs.com.br/vagas-de-emprego/[area]-[localizacao].html`
- Vagas.com: `https://www.vagas.com.br/vagas/[area]/[localizacao]`
- Indeed: `https://www.indeed.com.br/vagas?q=[area]&l=[localizacao]`

**Limitações do fallback**: Sem renderização JS, possível bloqueio anti-bot, HTML bruto em vez de markdown limpo.

### Passo 4: Extração de Detalhes da Vaga
Para cada vaga nos resultados (máximo 10 para processar):
1. Tentar `firecrawl scrape <url> --format markdown` via `terminal`
2. Se falhar, usar `titulo` e `descricao` do resultado da busca inicial
3. Anotar na resposta se os detalhes completos não puderam ser obtidos

### Passo 5: Correspondência de Habilidades
Para cada vaga processada:
1. Extrair requisitos da vaga (do markdown ou descrição)
2. Comparar com as habilidades do usuário usando **case-insensitive string matching**
3. Contar:
   - `habilidades_correspondentes`: Habilidades que aparecem na vaga
   - `habilidades_faltantes`: Habilidades exigidas na vaga que o usuário não tem
   - `contagem_correspondencia`: "X de Y habilidades correspondem"
4. **Filtragem por nível**: 
   - Priorizar vagas que mencionam o nível do usuário (Júnior, Pleno, Sênior)
   - Se não houver vagas do nível correspondente nos primeiros resultados, expandir busca e incluir vagas de nível adjacente, anotando a discrepância

### Passo 6: Formatação da Resposta
Retornar até 5 vagas no formato de lista numerada com pares chave-valor (sem tabelas markdown):

```
1. titulo: [título da vaga]
   empresa: [nome da empresa]
   localizacao: [cidade ou Remoto]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2]
   habilidades_faltantes: [habilidade3, habilidade4]
   contagem_correspondencia: [X de Y habilidades correspondem]

2. [próxima vaga no mesmo formato]
```

### Passo 7: Tratamento de Erros
- **Firecrawl falha**: Registrar erro no campo `erro` do envelope de resposta, tentar fallback
- **Fallback também falha**: Relatar erro e parar
- **Nenhum resultado**: Informar usuário e sugerir ampliar termos de busca
- **Algumas extrações falham**: Mostrar resultados parciais com detalhes completos onde disponíveis

## Formato do Envelope de Resposta (Scout → Maestro)

```
## ENVELOPE DE RESPOSTA: SCOUT
### estado
[sucesso | erro]

### resumo
[Breve descrição do que foi feito]

### dados
[Lista de até 5 vagas no formato do Passo 6]

### erro
[Mensagem de erro se estado=erro, caso contrário vazio]
```

## Regras Obrigatórias
1. Nunca inventar dados - se `firecrawl search` ou `firecrawl scrape` falhar, reportar erro exato e parar
2. Sempre usar `firecrawl` como método primário, `fetch` apenas como fallback
3. Todos os caminhos de arquivo devem começar com `data/`
4. Máximo de 5 vagas no resultado final
5. Case-insensitive matching para habilidades
