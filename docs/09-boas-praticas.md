# ✅ Boas Práticas e Convenções

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Tópicos Avançados](08-topicos-avancados.md) | [Próximo: Erros Comuns ➡️](10-erros-comuns.md)

---

> Estas são as boas práticas e convenções mais usadas na indústria para trabalhar com HTTP. Seguir esses padrões torna seu código mais profissional, seguro e fácil de manter.

---

## 📋 Cola Rápida — Regras de Ouro

| # | Regra | Importância |
|---|---|---|
| 1 | Sempre use HTTPS em produção | ⭐⭐⭐⭐⭐ |
| 2 | Use os métodos HTTP corretamente | ⭐⭐⭐⭐⭐ |
| 3 | Retorne status codes adequados | ⭐⭐⭐⭐⭐ |
| 4 | Valide e sanitize toda entrada | ⭐⭐⭐⭐⭐ |
| 5 | Configure CORS adequadamente | ⭐⭐⭐⭐ |
| 6 | Implemente rate limiting | ⭐⭐⭐⭐ |
| 7 | Use cache quando possível | ⭐⭐⭐⭐ |
| 8 | Versionamento de API | ⭐⭐⭐⭐ |
| 9 | Documente sua API | ⭐⭐⭐⭐ |
| 10 | Padronize respostas de erro | ⭐⭐⭐ |

---

## 1️⃣ Design de URLs

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo | Por quê |
|---|---|---|
| `/getUsuarios` | `/usuarios` | O verbo HTTP já diz a ação |
| `/criarProduto` | `POST /produtos` | Use o método HTTP |
| `/usuario` | `/usuarios` | Sempre **plural** |
| `/Usuarios` | `/usuarios` | Sempre **minúsculo** |
| `/usuarios_ativos` | `/usuarios?status=ativo` | Use query params para filtros |
| `/buscarUsuarioPorEmail` | `/usuarios?email=x@y.com` | Filtros na query string |
| `/usuarios/deletar/5` | `DELETE /usuarios/5` | O método já indica a ação |
| `/usuario/1/pedido/2/item/3` | `/pedidos/2/itens/3` | Evite aninhamento > 2 níveis |

### Boas práticas de URL

```
  ✅ BOM:
  GET    /api/v1/usuarios           → Listar
  GET    /api/v1/usuarios/42        → Buscar um
  POST   /api/v1/usuarios           → Criar
  PUT    /api/v1/usuarios/42        → Atualizar completo
  PATCH  /api/v1/usuarios/42        → Atualizar parcial
  DELETE /api/v1/usuarios/42        → Remover
  GET    /api/v1/usuarios/42/posts  → Posts do usuário 42

  ❌ RUIM:
  GET    /api/v1/getUser/42
  POST   /api/v1/createUser
  GET    /api/v1/deleteUser/42
  POST   /api/v1/user/update
```

### Paginação, filtro e ordenação

```bash
# Paginação
GET /api/produtos?page=2&per_page=20

# Filtros
GET /api/produtos?categoria=eletronicos&preco_min=100

# Ordenação
GET /api/produtos?sort=preco&order=asc

# Busca
GET /api/produtos?q=notebook

# Tudo junto
GET /api/produtos?q=notebook&categoria=eletronicos&sort=preco&order=asc&page=1&per_page=20
```

---

## 2️⃣ Status Codes Corretos

### Mapeamento recomendado

| Ação | Sucesso | Erro comum |
|---|---|---|
| `GET /items` | `200 OK` | `404` se vazio é erro |
| `GET /items/1` | `200 OK` | `404 Not Found` |
| `POST /items` | `201 Created` | `400 Bad Request`, `409 Conflict` |
| `PUT /items/1` | `200 OK` | `404`, `400` |
| `PATCH /items/1` | `200 OK` | `404`, `400` |
| `DELETE /items/1` | `204 No Content` | `404 Not Found` |

### ❌ Errado vs ✅ Certo

```javascript
// ❌ ERRADO: Status 200 com erro no body
app.get('/usuarios/:id', (req, res) => {
  const usuario = buscarUsuario(req.params.id);
  if (!usuario) {
    res.status(200).json({ erro: "Não encontrado" });  // ❌
  }
});

// ✅ CERTO: Status code correto
app.get('/usuarios/:id', (req, res) => {
  const usuario = buscarUsuario(req.params.id);
  if (!usuario) {
    res.status(404).json({                              // ✅
      erro: "Usuário não encontrado",
      codigo: "USUARIO_NAO_ENCONTRADO"
    });
  }
});
```

---

## 3️⃣ Formato de Resposta Padronizado

### Resposta de sucesso

```json
// Lista (com paginação)
{
  "dados": [
    { "id": 1, "nome": "Maria" },
    { "id": 2, "nome": "João" }
  ],
  "paginacao": {
    "pagina_atual": 1,
    "por_pagina": 20,
    "total_itens": 45,
    "total_paginas": 3
  }
}

// Item único
{
  "dados": {
    "id": 1,
    "nome": "Maria",
    "email": "maria@email.com"
  }
}
```

### Resposta de erro

```json
// Erro padronizado
{
  "erro": {
    "codigo": "VALIDACAO_FALHOU",
    "mensagem": "Os dados enviados são inválidos",
    "detalhes": [
      {
        "campo": "email",
        "mensagem": "Formato de email inválido"
      },
      {
        "campo": "nome",
        "mensagem": "Nome é obrigatório"
      }
    ]
  }
}
```

### ❌ Errado vs ✅ Certo

```json
// ❌ ERRADO: Formato inconsistente
{"error": "something went wrong"}
{"message": "not found"}
{"success": false, "data": null}

// ✅ CERTO: Formato consistente sempre
{
  "erro": {
    "codigo": "RECURSO_NAO_ENCONTRADO",
    "mensagem": "O usuário com ID 99 não foi encontrado"
  }
}
```

---

## 4️⃣ Versionamento de API

### Estratégias

| Estratégia | Exemplo | Mais usado? |
|---|---|---|
| **Na URL** | `/api/v1/usuarios` | ✅ Sim, mais comum |
| **No header** | `Accept: application/vnd.api.v1+json` | Menos comum |
| **Query param** | `/api/usuarios?version=1` | Raro |

### ❌ Errado vs ✅ Certo

```
  ❌ ERRADO:
  /api/usuarios              ← sem versão — vai quebrar clientes ao mudar

  ✅ CERTO:
  /api/v1/usuarios           ← versionado — pode ter v2 sem quebrar v1
  /api/v2/usuarios           ← nova versão com mudanças
```

### Quando criar nova versão?

| Mudança | Nova versão? |
|---|---|
| Adicionar campo novo na resposta | ❌ Não (retrocompatível) |
| Adicionar novo endpoint | ❌ Não |
| Remover um campo da resposta | ✅ Sim (breaking change) |
| Mudar tipo de um campo | ✅ Sim |
| Mudar formato da resposta | ✅ Sim |
| Remover um endpoint | ✅ Sim |

---

## 5️⃣ Segurança HTTP

### Headers de segurança obrigatórios

```
# Aplicar em toda resposta:
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### Checklist de segurança

- [ ] ✅ HTTPS em todas as rotas
- [ ] ✅ Headers de segurança configurados
- [ ] ✅ CORS restrito (não usar `*` em produção)
- [ ] ✅ Rate limiting implementado
- [ ] ✅ Validação de entrada (sanitizar dados)
- [ ] ✅ Tokens no header `Authorization` (não na URL!)
- [ ] ✅ Cookies com `HttpOnly`, `Secure`, `SameSite`
- [ ] ✅ Não expor informações do servidor (`X-Powered-By`)
- [ ] ✅ Logs de acesso sem dados sensíveis
- [ ] ✅ Timeout configurado em todas as requisições

### ❌ Errado vs ✅ Certo

```bash
# ❌ ERRADO: Token na URL (aparece em logs, histórico, referer)
GET /api/dados?token=eyJhbGci...

# ✅ CERTO: Token no header
GET /api/dados
Authorization: Bearer eyJhbGci...
```

```javascript
// ❌ ERRADO: CORS aberto para todo mundo
app.use(cors({ origin: '*' }));

// ✅ CERTO: CORS restrito
app.use(cors({
  origin: ['https://meusite.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,
}));
```

```javascript
// ❌ ERRADO: Usar dado do request direto no banco
app.get('/usuarios', (req, res) => {
  db.query(`SELECT * FROM users WHERE name = '${req.query.nome}'`); // SQL Injection!
});

// ✅ CERTO: Usar parâmetros preparados
app.get('/usuarios', (req, res) => {
  db.query('SELECT * FROM users WHERE name = ?', [req.query.nome]);
});
```

---

## 6️⃣ Performance HTTP

### Cache eficiente

```
  Recurso estático (CSS, JS, imagens):
  ✅ Cache-Control: public, max-age=31536000, immutable
        (1 ano + immutable = nunca revalida)
     Use hash no nome: style.abc123.css

  API com dados que mudam pouco:
  ✅ Cache-Control: max-age=300
     ETag: "versao-hash"

  Dados sensíveis/pessoais:
  ✅ Cache-Control: private, no-store

  Página HTML principal:
  ✅ Cache-Control: no-cache
     (sempre revalida, mas pode cachear)
```

### Compressão

```javascript
// Node.js/Express — habilitar compressão
const compression = require('compression');
app.use(compression());  // gzip automático!
```

### Outras otimizações

| Prática | Impacto | Dificuldade |
|---|---|---|
| Habilitar HTTP/2 | ⭐⭐⭐⭐⭐ | 🟢 Fácil |
| Compressão (gzip/brotli) | ⭐⭐⭐⭐ | 🟢 Fácil |
| Cache com ETags | ⭐⭐⭐⭐ | 🟡 Médio |
| CDN para assets estáticos | ⭐⭐⭐⭐⭐ | 🟢 Fácil |
| Paginação em listas | ⭐⭐⭐⭐ | 🟢 Fácil |
| Timeout adequado | ⭐⭐⭐ | 🟢 Fácil |
| Keep-Alive | ⭐⭐⭐ | 🟢 Fácil |

### Timeouts recomendados

```javascript
// ❌ ERRADO: Sem timeout (pode travar para sempre)
const resposta = await fetch('/api/dados');

// ✅ CERTO: Com timeout
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 5000); // 5 segundos

const resposta = await fetch('/api/dados', {
  signal: controller.signal,
});
clearTimeout(timeout);
```

---

## 7️⃣ Logging e Monitoramento

### O que logar em requisições HTTP

```
  ✅ LOGAR:
  - Timestamp
  - Método + URL
  - Status code
  - Tempo de resposta
  - IP do cliente (para segurança)
  - Request ID (para rastreio)

  ❌ NÃO LOGAR:
  - Senhas
  - Tokens de autenticação
  - Dados bancários / cartão de crédito
  - Dados pessoais sensíveis (CPF, etc.)
```

### Formato de log recomendado

```
2026-02-18T10:30:00Z [req-abc123] GET /api/usuarios 200 45ms
2026-02-18T10:30:01Z [req-def456] POST /api/usuarios 201 120ms
2026-02-18T10:30:02Z [req-ghi789] GET /api/usuarios/999 404 15ms
2026-02-18T10:30:03Z [req-jkl012] POST /api/login 401 30ms
```

### Request ID para rastreio

```javascript
// Middleware para gerar Request ID
app.use((req, res, next) => {
  const requestId = req.headers['x-request-id'] || crypto.randomUUID();
  req.requestId = requestId;
  res.setHeader('X-Request-ID', requestId);
  
  // Log com request ID
  console.log(`[${requestId}] ${req.method} ${req.url}`);
  
  next();
});
```

---

## 8️⃣ Tratamento de Erros

### Padrão de erro consistente

```javascript
// Middleware de tratamento de erros centralizado
app.use((err, req, res, next) => {
  // Log do erro (para o desenvolvedor)
  console.error(`[${req.requestId}] Erro:`, err.message);

  // Resposta para o cliente
  const statusCode = err.statusCode || 500;
  
  res.status(statusCode).json({
    erro: {
      codigo: err.codigo || 'ERRO_INTERNO',
      mensagem: statusCode === 500
        ? 'Erro interno do servidor'    // Genérico para o cliente
        : err.message,                   // Mensagem real para 4xx
    },
    request_id: req.requestId,
  });
});
```

### ❌ Errado vs ✅ Certo

```javascript
// ❌ ERRADO: Expor detalhes do erro interno
res.status(500).json({
  erro: "ECONNREFUSED 127.0.0.1:5432 - Postgres is down",
  stack: "Error at line 42..."
});

// ✅ CERTO: Mensagem genérica para o cliente
res.status(500).json({
  erro: {
    codigo: "ERRO_INTERNO",
    mensagem: "Ocorreu um erro inesperado. Tente novamente."
  }
});
// (detalhes ficam no log do servidor)
```

---

## 9️⃣ Documentação de API

### Ferramentas populares

| Ferramenta | Tipo | Destaque |
|---|---|---|
| **Swagger/OpenAPI** | Especificação + UI | Padrão da indústria |
| **Postman Collections** | Coleção de requests | Fácil de compartilhar |
| **README.md** | Markdown | Simples e no GitHub |
| **API Blueprint** | Markdown estruturado | Legível |

### Exemplo de documentação mínima

```markdown
## POST /api/usuarios

Cria um novo usuário.

**Headers:**
- `Content-Type: application/json`
- `Authorization: Bearer <token>`

**Body:**
| Campo | Tipo   | Obrigatório | Descrição       |
|-------|--------|-------------|-----------------|
| nome  | string | ✅          | Nome completo    |
| email | string | ✅          | Email válido     |
| idade | number | ❌          | Idade (opcional) |

**Exemplo de request:**
```json
{
  "nome": "Maria",
  "email": "maria@email.com",
  "idade": 25
}
```

**Respostas:**
- `201 Created` — Usuário criado
- `400 Bad Request` — Dados inválidos
- `409 Conflict` — Email já cadastrado
```

---

## 🔟 Checklist Final — Produção

```
  ┌────────────────────────────────────────────────────────┐
  │            CHECKLIST ANTES DE IR PARA PRODUÇÃO          │
  ├────────────────────────────────────────────────────────┤
  │  SEGURANÇA                                              │
  │  [ ] HTTPS habilitado                                   │
  │  [ ] Headers de segurança configurados                  │
  │  [ ] CORS restrito                                      │
  │  [ ] Rate limiting ativo                                │
  │  [ ] Validação de entrada em todas as rotas             │
  │  [ ] Tokens seguros (não na URL)                        │
  │                                                         │
  │  PERFORMANCE                                            │
  │  [ ] HTTP/2 habilitado                                  │
  │  [ ] Compressão habilitada (gzip/brotli)                │
  │  [ ] Cache configurado adequadamente                    │
  │  [ ] Timeouts definidos                                 │
  │  [ ] Paginação em endpoints de lista                    │
  │                                                         │
  │  QUALIDADE                                              │
  │  [ ] Status codes corretos                              │
  │  [ ] Formato de resposta padronizado                    │
  │  [ ] Tratamento de erros consistente                    │
  │  [ ] API versionada                                     │
  │  [ ] Documentação atualizada                            │
  │  [ ] Logging configurado (sem dados sensíveis)          │
  └────────────────────────────────────────────────────────┘
```

---

## 🎯 Exercícios

1. Revise uma API que você usa frequentemente — ela segue as boas práticas de URL?
2. Crie uma resposta de erro padronizada para 3 cenários diferentes
3. Configure headers de segurança em um servidor Node.js/Express
4. Escreva a documentação de 3 endpoints de uma API fictícia
5. Implemente um middleware de logging que registra método, URL, status e tempo

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Tópicos Avançados](08-topicos-avancados.md) | [Próximo: Erros Comuns ➡️](10-erros-comuns.md)
