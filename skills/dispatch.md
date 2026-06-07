# Habilidade: Despacho de Agentes

Esta habilidade descreve como o Maestro utiliza a ferramenta `spawn_agent` para delegar tarefas a outros agentes.

## Ferramenta Principal

*   `spawn_agent`: Para criar e interagir com sub-agentes.

## Protocolo de Despacho

O Maestro constrói uma mensagem detalhada para o `spawn_agent`, incluindo:

1.  **`label`**: Um rótulo descritivo para a tarefa em andamento (ex: "Buscando Vagas de Emprego").
2.  **`message`**: O prompt completo para o sub-agente. Este prompt deve conter:
    *   A persona do agente a ser despachado (referenciando o conteúdo do arquivo `.md` correspondente, ex: `personas/scout.md`).
    *   A tarefa específica a ser realizada.
    *   O contexto necessário, incluindo dados do perfil do usuário (`data/user-profile.md`) e quaisquer outras informações relevantes.
    *   O formato de saída esperado.
    *   Instruções claras sobre o tratamento de erros.

### Exemplo de Construção da Mensagem para o Scout:

Quando o usuário seleciona a opção 'A' (buscar vagas de emprego), o Maestro deve:

1.  Ler `data/user-profile.md` para obter as informações do usuário (`area_interesse`, `localizacao`, `nivel_experiencia`, `habilidades`).
2.  Ler o conteúdo de `personas/scout.md`.
3.  Construir a mensagem para `spawn_agent` da seguinte forma:

    ```
    Você é o Maestro. Seu objetivo é orquestrar a busca de vagas de emprego.
    O usuário selecionou a opção 'A' para buscar vagas.
    Construa o seguinte envelope de despacho para o agente Scout e use a ferramenta `spawn_agent` para chamá-lo:

    ## DESPACHO: SCOUT
    ### referencia_persona
    [Conteúdo completo de personas/scout.md]

    ### tarefa
    Buscar vagas de emprego para [area_interesse] em [localizacao]

    ### perfil_usuario
    [Conteúdo de data/user-profile.md]

    ### contexto
    Area: [area_interesse]
    Localizacao: [localizacao]
    Nivel: [nivel_experiencia]
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
    ```
4.  Chamar `default_api.spawn_agent(label="Buscando Vagas de Emprego", message=mensagem_construida)`.

## Tratamento de Respostas

Após a execução do `spawn_agent`, o Maestro deve analisar a resposta do sub-agente:

*   Se o `state` for "success", processar os `data` retornados, salvar em `data/job-search-results.md` e apresentar ao usuário.
*   Se o `state` for "error", exibir a mensagem de `error` ao usuário.
*   Em ambos os casos, retornar ao `menu_principal` do Maestro.
