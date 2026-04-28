# Casa dos Dados API — Referencia Completa

Base URL: `https://api.casadosdados.com.br`
Auth: header `api-key`

## Endpoints

### Consulta CNPJ
| Metodo | Path | Descricao |
|--------|------|-----------|
| GET | `/v4/cnpj/{cnpj}` | Consultar empresa por CNPJ |

### Pesquisa Avancada (v5)
| Metodo | Path | Descricao |
|--------|------|-----------|
| POST | `/v5/cnpj/pesquisa` | Pesquisa avancada de empresas |
| POST | `/v5/cnpj/pesquisa/arquivo` | Gerar arquivo CSV com resultados |

### Arquivos
| Metodo | Path | Descricao |
|--------|------|-----------|
| GET | `/v4/cnpj/pesquisa/arquivo` | Listar solicitacoes de arquivos |
| GET | `/v4/public/cnpj/pesquisa/arquivo/{uuid}` | Consultar link de download |

### Saldo
| Metodo | Path | Descricao |
|--------|------|-----------|
| GET | `/v5/saldo` | Consultar saldo total e detalhado |

### Dashboard
| Metodo | Path | Descricao |
|--------|------|-----------|
| GET | `/v4/dashboard/cnpj/empresas-abertas/hoje` | Empresas abertas hoje |
| GET | `/v4/dashboard/cnpj/empresas-abertas/{dias}` | Empresas abertas nos ultimos X dias |

---

## Schema: Body da pesquisa (POST /v5/cnpj/pesquisa)

```json
{
  "busca_textual": [
    {
      "texto": ["termo1", "termo2"],
      "tipo_busca": "radical",
      "razao_social": true,
      "nome_fantasia": true,
      "nome_socio": false
    }
  ],
  "cnpj": [],
  "cnpj_raiz": [],
  "codigo_atividade_principal": ["4321500"],
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
    "inicio": "2025-01-01",
    "fim": "2026-04-28",
    "ultimos_dias": 30
  },
  "capital_social": {
    "minimo": 100000,
    "maximo": 5000000
  },
  "mei": {
    "optante": true,
    "excluir_optante": false
  },
  "simples": {
    "optante": true,
    "excluir_optante": false
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

### Notas sobre os campos

- `texto`: array de strings. Pode enviar multiplos termos
- `tipo_busca`: "radical" (default, mais abrangente) ou "exata"
- `municipio`: array de strings em lowercase sem acentos (ex: ["campinas", "sao paulo"])
- `uf`: array de siglas uppercase (ex: ["SP", "RJ"])
- `data_abertura`: usar `ultimos_dias` OU `inicio`/`fim`, nao ambos
- `capital_social`: valores em reais (numerico)
- `situacao_cadastral`: valores possiveis: "ATIVA", "BAIXADA", "INAPTA", "NULA", "SUSPENSA"
- `matriz_filial`: "MATRIZ" ou "FILIAL"
- `codigo_atividade_principal`: codigo CNAE sem pontuacao (ex: "4321500")
- `limite`: max 20 por pagina
- `pagina`: comecar em 1

---

## Schema: Resposta da pesquisa (CNPJPesquisaResposta)

```json
{
  "cnpj": "24728116000100",
  "razao_social": "RIOTELE-REAL INTERNET OPTICA TELECOMUNICACOES LTDA",
  "nome_fantasia": "GLOBAL LIVE",
  "situacao_cadastral": "ATIVA",
  "data_situacao_cadastral": "2016-05-04",
  "data_inicio_atividade": "2016-05-04",
  "cnae_fiscal": 4321500,
  "cnae_fiscal_descricao": "Instalacao e manutencao eletrica",
  "logradouro": "RUA CARLOS ROBERTO DE MELO",
  "numero": "475",
  "complemento": "PAVMTO12 SALA 05",
  "bairro": "PARQUE GABRIEL",
  "cep": "13186604",
  "municipio": "HORTOLANDIA",
  "uf": "SP",
  "ddd_telefone_1": "1934754499",
  "email": "societario2@alcalacontabilidade.com.br",
  "natureza_juridica": "Sociedade Empresaria Limitada",
  "porte": "DEMAIS",
  "capital_social": 4714404,
  "identificador_matriz_filial": "MATRIZ",
  "opcao_pelo_simples": false,
  "opcao_pelo_mei": false
}
```

---

## Schema: Body para gerar arquivo (POST /v5/cnpj/pesquisa/arquivo)

```json
{
  "nome": "nome_do_arquivo",
  "tipo": "csv",
  "total_linhas": 100,
  "enviar_para": ["email@empresa.com.br"],
  "pesquisa": {
    "... mesmos campos da pesquisa avancada ..."
  }
}
```

Resposta:
```json
{
  "mensagem": "Arquivo em processamento",
  "arquivo_uuid": "abc-123-def-456"
}
```

---

## Schema: Resposta do saldo (GET /v5/saldo)

```json
{
  "saldo_total": 1500,
  "saldos": {
    "pesquisa": 500,
    "consulta": 800,
    "arquivo": 200
  }
}
```

---

## Codigos de erro

| HTTP | Significado |
|------|-------------|
| 200 | Sucesso |
| 202 | Arquivo em processamento |
| 400 | Requisicao invalida |
| 401 | API key invalida |
| 403 | Sem saldo ou bloqueado |
| 404 | Nao encontrado |
| 429 | Rate limit excedido |
| 500 | Erro interno |
