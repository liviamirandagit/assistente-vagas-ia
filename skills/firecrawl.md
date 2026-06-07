# Habilidade: Firecrawl CLI

Esta habilidade descreve como o Scout utiliza a ferramenta de linha de comando `firecrawl` para buscar e extrair dados de vagas de emprego.

## Ferramenta Principal

*   `terminal`: Para executar comandos CLI do `firecrawl`.

## Comandos Disponíveis

### 1. Busca de Vagas (`firecrawl search`)
*   **Descrição:** Realiza uma busca agregada em múltiplas fontes (Indeed, Catho, LinkedIn, Glassdoor, Infojobs, etc.).
*   **Sintaxe:** `firecrawl search "<termos_de_busca>" --json`
*   **Exemplo:** `firecrawl search "Engenheiro de Software Remoto" --json`
*   **Saída Esperada (JSON):**
    ```json
    [
      {
        "url": "https://exemplo.com/vaga-123",
        "titulo": "Vaga de Engenheiro de Software",
        "descricao": "Breve descrição da vaga...",
        "empresa": "Nome da Empresa",
        "carga": "Full-time"
      }
    ]
    ```
*   **Tratamento de Erros:** Se o comando falhar, o terminal retornará um erro. O Scout deve capturar essa saída e retornar no `response_envelope` com `state: "error"`.

### 2. Extração de Detalhes (`firecrawl scrape`)
*   **Descrição:** Acessa uma URL específica de uma vaga e extrai o conteúdo em formato Markdown limpo.
*   **Sintaxe:** `firecrawl scrape <url> --format markdown`
*   **Exemplo:** `firecrawl scrape "https://exemplo.com/vaga-123" --format markdown`
*   **Saída Esperada:** Texto em Markdown contendo a descrição completa da vaga, requisitos, benefícios, etc.
*   **Tratamento de Erros:**
    *   Se `firecrawl scrape` falhar (timeout, 404, etc.), o Scout deve usar as informações da busca inicial (`titulo` e `descricao` de `firecrawl search`) como fallback.
    *   Anotar na resposta que os detalhes completos não puderam ser obtidos.

## Regras de Uso (Obrigatórias)

1.  **Prioridade:** O `firecrawl` é a ferramenta **primária** de acesso à web. Nunca use `fetch`, `curl` ou `wget` como primeira opção.
2.  **Fallback:** Apenas se o `firecrawl` falhar consistentemente (ex: 3 tentativas), o Scout pode usar `curl` ou `wget` como recuperação, anotando as limitações (sem JS, possível bloqueio).
3.  **Formato JSON:** Sempre use a flag `--json` no `firecrawl search` para facilitar o processamento programático.
4.  **Formato Markdown:** Use `--format markdown` no `firecrawl scrape` para obter texto limpo e legível.
5.  **Proteção de Dados:** Nunca armazene dados sensíveis do usuário em logs ou saídas intermediárias. Use apenas os dados do `data/user-profile.md` para correspondência.

## Integração com o Scout

O Scout deve:
1.  Ler o perfil do usuário (`data/user-profile.md`) para obter área, localização e habilidades.
2.  Executar `firecrawl search` com os termos apropriados.
3.  Para cada resultado relevante, executar `firecrawl scrape` na URL para obter detalhes completos.
4.  Processar os dados conforme `skills/job-search.md`.
5.  Retornar os resultados via `response_envelope` para o Maestro.
