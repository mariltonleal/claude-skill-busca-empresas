---
name: busca_empresas_cnpj
description: "Central completa de busca e consulta de empresas brasileiras via API Casa dos Dados. Use SEMPRE que o usuario pedir para buscar empresa por nome, razao social, nome fantasia, socio, CNAE, cidade, UF, CEP, capital social, Simples, MEI, empresas novas, gerar CSV de leads, consultar saldo da API, ou qualquer pesquisa de empresas/CNPJs. Tambem use quando o usuario disser 'buscar empresa', 'procurar cnpj', 'encontrar empresa', 'listar empresas', 'prospectar', 'gerar leads', 'empresas abertas', 'pesquisa avancada cnpj', 'exportar empresas'. Esta skill complementa a busca_cnpj (consulta detalhada de um CNPJ especifico via cnpj.ws)."
---

# Busca Empresas CNPJ — Central de pesquisa de empresas brasileiras

Skill para buscar, filtrar e exportar dados de empresas brasileiras usando a API Casa dos Dados. Permite localizar empresas por nome, atividade, localizacao, porte, regime tributario e muito mais.

## IMPORTANTE: API paga com creditos

A API Casa dos Dados e **paga por creditos**. Cada consulta consome creditos do saldo. Por isso, esta skill implementa controle de saldo obrigatorio para evitar uso excessivo.

### Custo por operacao

| Operacao | Custo estimado | Endpoint |
|----------|---------------|----------|
| Pesquisa de empresas | ~1 credito por pagina | POST /v5/cnpj/pesquisa |
| Consulta CNPJ individual | ~1 credito | GET /v4/cnpj/{cnpj} |
| Geracao de arquivo CSV | ~1 credito por linha | POST /v5/cnpj/pesquisa/arquivo |
| Consulta de saldo | gratuito | GET /v5/saldo |
| Dashboard empresas abertas | gratuito | GET /v4/dashboard/* |

### Controle de saldo obrigatorio

**ANTES de cada chamada paga**, consultar o saldo via `GET /v5/saldo` e aplicar estas regras:

| Saldo | Nivel | Acao |
|-------|-------|------|
| > 1000 | Normal | Executar normalmente |
| 500–1000 | Atencao | Executar + exibir aviso: "⚠️ Saldo em atencao: {saldo} creditos restantes" |
| 100–499 | Baixo | Exibir alerta antes de executar: "🔶 Saldo baixo: {saldo} creditos. Deseja continuar?" — aguardar confirmacao do usuario |
| < 100 | Critico | Bloquear execucao: "🔴 Saldo critico: {saldo} creditos. Operacao bloqueada para evitar ficar sem saldo. Use '/busca_empresas_cnpj saldo' para ver detalhes." |

**Para geracao de CSV** (que consome mais creditos), aplicar regra mais conservadora:
- Sempre mostrar estimativa: "Este CSV vai consumir ~{total_linhas} creditos. Saldo atual: {saldo}. Confirma?"
- Bloquear se creditos < total_linhas solicitado

### Formato da resposta de saldo

```bash
GET /v5/saldo
```

Resposta:
```json
{
  "saldo_total": 9080,
  "saldos": {
    "avulso": {
      "valor": 9080,
      "criado_em": "2025-07-24T14:40:28Z",
      "expira_em": "2027-07-24T14:40:28Z"
    },
    "subscription-2000": {
      "valor": 0,
      "criado_em": "2025-06-11T21:15:53Z",
      "expira_em": "2025-07-13T00:00:00Z"
    }
  }
}
```

### Exibicao de saldo

Sempre que consultar o saldo (automaticamente ou via `/busca_empresas_cnpj saldo`), exibir:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  SALDO API CASA DOS DADOS
��━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💰 Saldo total: {saldo_total} creditos

📦 Detalhamento:
   Avulso: {valor} creditos (expira em {data_formatada})
   Assinatura: {valor} creditos (expira em {data_formatada})

{emoji_nivel} Status: {Normal / Atencao / Baixo / Critico}
```

### Rodape de saldo apos cada busca

Apos cada busca paga, incluir no rodape da resposta:

```
💰 Saldo: {saldo_apos_busca} creditos restantes {emoji se atencao/baixo/critico}
```

## Configuracao

### API Key

A API requer autenticacao via header `api-key`. A configuracao fica no arquivo `config.json` na propria pasta da skill:

**Caminho**: `~/.claude/skills/busca_empresas_cnpj/config.json`

```json
{
  "api_key": "SUA_API_KEY_AQUI",
  "base_url": "https://api.casadosdados.com.br"
}
```

Ao executar, leia este arquivo para obter a `api_key` e `base_url`. Se o arquivo nao existir ou a key for "COLE_SUA_API_KEY_AQUI", avisar o usuario para configurar:
- Copiar `config.example.json` para `config.json`
- Colar a API key real

### Base URL

```
https://api.casadosdados.com.br
```

## Como funciona

O usuario invoca com `/busca_empresas_cnpj` seguido do que quer buscar. Exemplos:

```
/busca_empresas_cnpj riotele
/busca_empresas_cnpj empresas de embalagens em Campinas SP
/busca_empresas_cnpj empresas abertas nos ultimos 30 dias em SP com telefone
/busca_empresas_cnpj CNAE 4321500 em Campinas
/busca_empresas_cnpj capital social acima de 500 mil em SP
/busca_empresas_cnpj gerar CSV de empresas de embalagens em SP
/busca_empresas_cnpj consultar saldo
```

## Roteamento por intencao

Analisar o input do usuario e escolher a funcao correta:

| Intencao do usuario | Funcao | Endpoint |
|---------------------|--------|----------|
| Buscar por nome/razao social/fantasia/socio | `buscar_empresas` | POST /v5/cnpj/pesquisa |
| Pesquisa com multiplos filtros | `pesquisa_avancada` | POST /v5/cnpj/pesquisa |
| Buscar por cidade/UF/CEP/bairro | `buscar_por_localizacao` | POST /v5/cnpj/pesquisa |
| Buscar por CNAE/atividade | `buscar_por_atividade` | POST /v5/cnpj/pesquisa |
| Buscar empresas com email/telefone | `buscar_com_contato` | POST /v5/cnpj/pesquisa |
| Buscar empresas novas/abertas recentemente | `buscar_empresas_novas` | POST /v5/cnpj/pesquisa |
| Buscar por capital social | `buscar_por_capital` | POST /v5/cnpj/pesquisa |
| Buscar Simples/MEI | `buscar_simples_mei` | POST /v5/cnpj/pesquisa |
| Buscar matriz ou filial | `buscar_matriz_filial` | POST /v5/cnpj/pesquisa |
| Consultar CNPJ especifico | `consultar_cnpj` | GET /v4/cnpj/{cnpj} |
| Gerar CSV/arquivo | `gerar_arquivo` | POST /v5/cnpj/pesquisa/arquivo |
| Listar arquivos gerados | `listar_arquivos` | GET /v4/cnpj/pesquisa/arquivo |
| Consultar link de download | `consultar_link_arquivo` | GET /v4/public/cnpj/pesquisa/arquivo/{uuid} |
| Consultar saldo | `consultar_saldo` | GET /v5/saldo |
| Empresas abertas hoje | `dashboard_hoje` | GET /v4/dashboard/cnpj/empresas-abertas/hoje |
| Empresas abertas ultimos X dias | `dashboard_dias` | GET /v4/dashboard/cnpj/empresas-abertas/{dias} |

## Execucao das chamadas

### Formato padrao do curl

```bash
curl -s -w '\n%{http_code}' \
  --max-time 30 \
  -X {METHOD} \
  -H 'accept: application/json' \
  -H 'content-type: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}{PATH}' \
  -d '{BODY}'
```

### Defaults para todas as pesquisas

- `limite`: 10
- `pagina`: 1
- `tipo_busca`: "radical" (busca por radicais, mais abrangente que "exata")
- `situacao_cadastral`: ["ATIVA"]
- Sempre remover campos vazios/nulos do body antes de enviar

## Funcoes detalhadas

### 1. buscar_empresas (busca textual)

A funcao mais usada. Busca por nome, razao social, nome fantasia ou socio.

```bash
curl -s -w '\n%{http_code}' --max-time 30 \
  -X POST \
  -H 'accept: application/json' \
  -H 'content-type: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v5/cnpj/pesquisa' \
  -d '{
    "busca_textual": [
      {
        "texto": ["{termo}"],
        "tipo_busca": "radical",
        "razao_social": true,
        "nome_fantasia": true,
        "nome_socio": false
      }
    ],
    "situacao_cadastral": ["ATIVA"],
    "limite": 10,
    "pagina": 1
  }'
```

Quando o usuario pedir busca por **nome de socio**, mudar `nome_socio: true` e os outros para `false`.

### 2. consultar_cnpj (consulta direta)

```bash
curl -s -w '\n%{http_code}' --max-time 30 \
  -X GET \
  -H 'accept: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v4/cnpj/{cnpj_limpo}'
```

### 3. pesquisa_avancada (todos os filtros)

Body completo com todos os filtros disponiveis:

```json
{
  "busca_textual": [
    {
      "texto": ["{termo}"],
      "tipo_busca": "radical",
      "razao_social": true,
      "nome_fantasia": true,
      "nome_socio": false
    }
  ],
  "cnpj": [],
  "cnpj_raiz": [],
  "codigo_atividade_principal": [],
  "incluir_atividade_secundaria": true,
  "codigo_atividade_secundaria": [],
  "codigo_natureza_juridica": [],
  "situacao_cadastral": ["ATIVA"],
  "matriz_filial": "MATRIZ",
  "cep": [],
  "uf": ["SP"],
  "municipio": ["campinas"],
  "bairro": [],
  "ddd": [],
  "telefone": [],
  "data_abertura": {
    "inicio": "",
    "fim": "",
    "ultimos_dias": null
  },
  "capital_social": {
    "minimo": null,
    "maximo": null
  },
  "mei": {
    "optante": null,
    "excluir_optante": null
  },
  "simples": {
    "optante": null,
    "excluir_optante": null
  },
  "porte_empresa": {
    "codigos": []
  },
  "mais_filtros": {
    "somente_matriz": false,
    "somente_filial": false,
    "com_email": false,
    "com_telefone": false,
    "somente_fixo": false,
    "somente_celular": false,
    "excluir_empresas_visualizadas": false,
    "excluir_email_contab": false
  },
  "excluir": {
    "cnpj": []
  },
  "limite": 10,
  "pagina": 1
}
```

**Regra**: so incluir no body os campos que o usuario realmente informou. Remover tudo que estiver vazio, null ou com array vazio.

### 4. buscar_empresas_novas

Empresas abertas recentemente. Usa o filtro `data_abertura.ultimos_dias`:

```json
{
  "situacao_cadastral": ["ATIVA"],
  "uf": ["{uf}"],
  "municipio": ["{cidade}"],
  "data_abertura": {
    "ultimos_dias": 30
  },
  "mais_filtros": {
    "com_email": true,
    "com_telefone": true
  },
  "limite": 10,
  "pagina": 1
}
```

### 5. gerar_arquivo (exportar CSV)

```bash
curl -s -w '\n%{http_code}' --max-time 30 \
  -X POST \
  -H 'accept: application/json' \
  -H 'content-type: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v5/cnpj/pesquisa/arquivo' \
  -d '{
    "nome": "{nome_arquivo}",
    "tipo": "csv",
    "total_linhas": 100,
    "enviar_para": ["{email}"],
    "pesquisa": { ... filtros da pesquisa ... }
  }'
```

### 6. consultar_saldo

```bash
curl -s -w '\n%{http_code}' --max-time 30 \
  -X GET \
  -H 'accept: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v5/saldo'
```

### 7. listar_arquivos

```bash
curl -s -w '\n%{http_code}' --max-time 30 \
  -X GET \
  -H 'accept: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v4/cnpj/pesquisa/arquivo?pagina=1'
```

### 8. consultar_link_arquivo

```bash
curl -s -w '\n%{http_code}' --max-time 30 \
  -X GET \
  -H 'accept: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v4/public/cnpj/pesquisa/arquivo/{uuid}'
```

- HTTP 200: arquivo pronto, retornar link de download
- HTTP 202: ainda processando, informar o usuario
- HTTP 404: arquivo nao encontrado

### 9. dashboard_hoje e dashboard_dias

```bash
# Empresas abertas hoje
curl -s -w '\n%{http_code}' --max-time 30 \
  -X GET \
  -H 'accept: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v4/dashboard/cnpj/empresas-abertas/hoje'

# Empresas abertas nos ultimos X dias
curl -s -w '\n%{http_code}' --max-time 30 \
  -X GET \
  -H 'accept: application/json' \
  -H 'api-key: {API_KEY}' \
  '{BASE_URL}/v4/dashboard/cnpj/empresas-abertas/{dias}'
```

## Tratamento de erros HTTP

| Codigo | Significado | Mensagem para usuario |
|--------|-------------|----------------------|
| 200 | Sucesso | Processar e exibir resultados |
| 202 | Processando | "Arquivo em processamento. Consulte novamente em alguns minutos." |
| 400 | Requisicao invalida | "Solicitacao invalida. Verifique os filtros enviados." |
| 401 | API key invalida | "API key invalida ou ausente. Verifique a configuracao." |
| 403 | Sem saldo/bloqueado | "Sem saldo para esta operacao ou acesso bloqueado." |
| 404 | Nao encontrado | "Nenhum registro encontrado." |
| 429 | Rate limit | "Limite de requisicoes excedido. Aguarde alguns segundos." |
| 500+ | Erro servidor | "Erro interno da API. Tente novamente mais tarde." |

## Output padrao (JSON)

Todas as pesquisas retornam este formato:

```json
{
  "success": true,
  "total": 15,
  "pagina": 1,
  "limite": 10,
  "empresas": [
    {
      "cnpj": "24728116000100",
      "cnpj_formatado": "24.728.116/0001-00",
      "razao_social": "RIOTELE-REAL INTERNET OPTICA TELECOMUNICACOES LTDA",
      "nome_fantasia": "GLOBAL LIVE",
      "matriz_filial": "Matriz",
      "situacao": "Ativa",
      "porte": "Demais",
      "natureza_juridica": "Sociedade Empresaria Limitada",
      "data_abertura": "04/05/2016",
      "capital_social": "R$ 4.714.404,00",
      "cidade": "Hortolandia",
      "uf": "SP",
      "bairro": "PARQUE GABRIEL",
      "cep": "13.186-604",
      "atividade_principal": "Instalacao e manutencao eletrica"
    }
  ]
}
```

## Resposta humana

Apos o JSON, exibir versao formatada:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  BUSCA DE EMPRESAS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔎 Busca: {termo/filtros usados}
📊 Total encontrado: {total}
📄 Pagina {pagina} de {total_paginas}

1. 🏢 {razao_social}
   🏷️ Fantasia: {nome_fantasia}
   📄 CNPJ: {cnpj_formatado}
   📊 Situação: {situacao}
   🏠 Tipo: {Matriz/Filial}
   📍 {cidade}/{uf}
   💰 Capital: {capital_social_formatado}
   🧩 Atividade: {atividade_principal}

2. 🏢 {razao_social}
   ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 Para ver detalhes completos: /busca_cnpj {cnpj}
💡 Para cadastrar no BaseERP: /postman POST /customers {payload}
💡 Para proxima pagina: /busca_empresas_cnpj {mesmos filtros} pagina 2
```

## Funcoes auxiliares de formatacao

- **formatarCNPJ**: `12345678000190` → `12.345.678/0001-90`
- **formatarCEP**: `13060001` → `13.060-001`
- **formatarMoedaBRL**: `1500000` → `R$ 1.500.000,00`
- **formatarData**: `2016-05-04` → `04/05/2016`
- **limparTexto**: substituir null/vazio por "Nao informado"
- **somenteNumeros**: remover tudo que nao for digito
- **normalizarMunicipio**: lowercase, sem acentos (API espera assim)
- **removerCamposVazios**: limpar body antes de enviar (remover arrays vazios, nulls, objetos vazios)
- **montarBuscaTextual**: montar bloco `busca_textual` do body
- **montarRespostaHumana**: formatar resultado para exibicao

## Integracao com outras skills

Apos exibir resultados, sugerir proximas acoes:

- **Detalhes completos**: "Para dados fiscais completos de uma empresa: `/busca_cnpj {cnpj}`"
- **Cadastro BaseERP**: "Para cadastrar no ERP: `/postman POST /customers`"
- **Prospeccao**: "Para exportar lista: informe `/busca_empresas_cnpj gerar CSV ...`"

## Valores de referencia

### Situacao cadastral
- `ATIVA`, `BAIXADA`, `INAPTA`, `NULA`, `SUSPENSA`

### Matriz/Filial
- `MATRIZ`, `FILIAL`

### Tipo busca
- `radical` (busca por radicais das palavras — padrao, mais abrangente)
- `exata` (busca exata pelo termo completo)

## Seguranca

- NUNCA exibir a API key completa — mascarar como `abc...xyz`
- API key fica apenas no `environments.json`, nunca no historico ou output
- Ao exibir headers, mascarar o `api-key`

## Detalhes da API

Para referencia completa dos schemas de resposta e campos disponiveis, consulte `references/casadosdados-api.md`.
