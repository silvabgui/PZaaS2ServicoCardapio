# PZaaS - Pizza as a Service

## Serviço 02 - Gerenciamento e Consulta de Cardápio

**Propósito:** Disponibilizar o catálogo de pizzas e promoções, aplicando regras de negócio (ordenação de combos) e validação de disponibilidade em tempo real via integração com o microsserviço de Estoque.

Url em produção: `https://pzaas.online/webhook/v1/lista`

## Recursos Utilizados

| Recurso | Descrição |
| :--- | :--- |
| n8n | Usado na estruturação de toda a Arquitetura |
| Redis | Banco de dados utilizado para o registro das Pizzas |
| Postman | Utilizado para testes HTTP's |

## Endpoints Disponíveis

| Método | Endpoint | Descrição |
| :--- | :--- | :--- |
| `GET` | `v1/lista/health` | Verifica a disponibilidade da API. |
| `GET` | `/v1/lista` | Endpoint principal de consulta consumido por clientes ou outras equipes para exibir a vitrine de pizzas filtradas e ordenadas. |
| `POST` | `v1/cardapio` | Responsável por persistir os dados dos produtos na base em memória. |

### Cabeçalhos (Headers)

| Header | Valor | Obrigatório |
| :--- | :--- | :--- |
| `x-api-key` | `turma2026` | Sim |
| `Content-Type` | `application/json` | Sim |
| `x-pedido-id` | | Não |

### Contratos

| Variável | Retorno | Obrigatório | Tipo |
| :--- | :--- | :--- | :--- |
| `nome` | Nome do sabor da pizza | Sim | String |
| `preco_original` | Preço original (é alterada em caso de promoção) | Sim | Number |
| `preco` | Preço atualizado (somente em caso de promoção) | Sim | Number |
| `ingredientes` | Ingredientes da pizza buscado do setor de *Estoque* | Sim | String |

### Payload de Cadastro - POST /v1/cardapio
Formato esperado no corpo da requisição para cadastrar uma nova pizza no banco:

```json
    {
      "nome": "Calabresa",
      "preco_original": 45,
      "preco": 45,
      "ingredientes": ["massa", "molho de tomate", "mussarela", "calabresa", "cebola"]
    }
```


### Teste em Produção - Get /v1/lista

```json
[
    {
        "nome": "Portuguesa",
        "preco": 55,
        "preco_original": 55,
        "ingredientes": [
            "massa",
            "molho de tomate",
            "mussarela",
            "presunto",
            "ovo",
            "cebola"
        ]
    },
    {
        "nome": "Calabresa",
        "preco": 45,
        "preco_original": 45,
        "ingredientes": [
            "massa",
            "molho de tomate",
            "mussarela",
            "calabresa",
            "cebola"
        ]
    },
    {
        "nome": "Napolitana",
        "preco": 50,
        "preco_original": 50,
        "ingredientes": [
            "massa",
            "molho de tomate",
            "mussarela",
            "tomate"
        ]
    },
    {
        "nome": "Mussarela",
        "preco": 40,
        "preco_original": 40,
        "ingredientes": [
            "massa",
            "molho de tomate",
            "mussarela"
        ]
    },
    {
        "nome": "Promoção Casal",
        "preco": 75,
        "preco_original": 90,
        "ingredientes": [
            "massa",
            "molho de tomate",
            "mussarela",
            "calabresa"
        ]
    },
    {
        "nome": "Combo Família",
        "preco": 99,
        "preco_original": 140,
        "ingredientes": [
            "massa",
            "molho de tomate",
            "mussarela",
            "presunto",
            "ovo"
        ]
    }
]
```

### Requisições HTTP

| Código HTTP | Status | Descrição |
| :--- | :--- | :--- | 
| 200 | Ok |Acesso Negado | Sucesso na consulta. Retorna o cardápio ou status do Healthcheck. |
| 201 | Created |Sucesso na criação/persistência de um novo item no Redis. |
| 401 | Unauthorized | Acesso negado. Ocorre quando a x-api-key está ausente ou inválida. |
| 404 | Not Found | Recurso ou rota não encontrada. |

### Como Consumir a API

Para que outra dupla obtenha o cardápio, basta realizar uma requisição `GET` na rota `/v1/lista`. Não é necessário enviar *body*, apenas o header de autenticação.

**Exemplo prático de chamada via cURL:**
```bash
curl -X GET "[https://pzaas.online/webhook/v1/lista](https://pzaas.online/webhook/v1/lista)" \
     -H "x-api-key: turma2026"
