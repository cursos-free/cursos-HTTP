# 🔀 Tópicos Intermediários

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Headers HTTP](06-headers.md) | [Próximo: Tópicos Avançados ➡️](08-topicos-avancados.md)

---

> Agora que você domina os fundamentos, vamos explorar os tópicos que todo desenvolvedor lida no dia a dia: cookies, sessões, cache, CORS e autenticação.

---

## 📋 Cola Rápida — O que Vamos Cobrir

| Tópico | Para que serve | Importância |
|---|---|---|
| **Cookies** | Armazenar dados no navegador | ⭐⭐⭐⭐⭐ |
| **Sessões** | Manter usuário "logado" | ⭐⭐⭐⭐⭐ |
| **Cache HTTP** | Evitar requisições desnecessárias | ⭐⭐⭐⭐ |
| **CORS** | Controlar acesso entre domínios | ⭐⭐⭐⭐⭐ |
| **Autenticação** | Provar identidade do usuário | ⭐⭐⭐⭐⭐ |
| **Query Strings** | Enviar parâmetros na URL | ⭐⭐⭐⭐ |
| **Redirecionamentos** | Encaminhar para outra URL | ⭐⭐⭐ |
| **Content Negotiation** | Negociar formato de dados | ⭐⭐⭐ |

---

## 1️⃣ Cookies

Cookies são **pequenos textos** que o servidor pede ao navegador para guardar. Em cada requisição seguinte, o navegador envia os cookies de volta.

### 💡 Analogia: Pulseira de evento

```
  1. Você chega no evento         → Primeira requisição
  2. Recebe uma pulseira (#ABC)   → Set-Cookie: sessao=ABC
  3. Mostra a pulseira na volta   → Cookie: sessao=ABC
  4. Segurança te reconhece       → Servidor sabe quem é você
```

### Como funciona

```
  Cliente                              Servidor
     │                                    │
     │── GET /login ────────────────────►│
     │   (usuário + senha)                │
     │                                    │
     │◄── 200 OK ─────────────────────── │
     │    Set-Cookie: sessao=abc123;     │
     │    Path=/; HttpOnly; Secure       │
     │                                    │
     │── GET /perfil ───────────────────►│
     │   Cookie: sessao=abc123           │  ← enviado automaticamente!
     │                                    │
     │◄── 200 OK (dados do perfil) ─────│
```

### Exemplos práticos

```bash
# Ver cookies de um site
curl -v https://google.com 2>&1 | grep -i "set-cookie"

# Enviar cookie em uma requisição
curl -b "sessao=abc123" https://httpbin.org/cookies

# Salvar e reusar cookies (como um navegador)
curl -c cookies.txt https://httpbin.org/cookies/set/nome/maria
curl -b cookies.txt https://httpbin.org/cookies
```

### Atributos importantes do cookie

| Atributo | O que faz | Exemplo |
|---|---|---|
| `Path` | Limita a quais caminhos o cookie é enviado | `Path=/api` |
| `Domain` | Define para qual domínio | `Domain=.exemplo.com` |
| `Expires` | Data de expiração | `Expires=Thu, 01 Jan 2027` |
| `Max-Age` | Tempo de vida em segundos | `Max-Age=3600` |
| `HttpOnly` | JavaScript não pode acessar (segurança!) | `HttpOnly` |
| `Secure` | Só envia via HTTPS | `Secure` |
| `SameSite` | Controla envio cross-site | `SameSite=Strict` |

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Cookie sem `HttpOnly` com dados sensíveis | Sempre usar `HttpOnly` para tokens |
| Cookie sem `Secure` com login | Sempre usar `Secure` em produção |
| Cookie sem `SameSite` | Adicionar `SameSite=Lax` ou `Strict` |
| Guardar senha no cookie | Guardar apenas um ID de sessão |

---

## 2️⃣ Sessões

Sessões são uma forma de **manter estado** entre requisições (lembra que HTTP é stateless?).

### Como funciona

```
  ┌────────────────────────────────────────────────────────┐
  │                 FLUXO DE SESSÃO                         │
  └────────────────────────────────────────────────────────┘

  1. Login:
     Cliente ──► POST /login {email, senha}
     Servidor ──► Cria sessão (ID: abc123)
                  Salva: { abc123: { user: "Maria", role: "admin" } }
     Cliente ◄── Set-Cookie: sessao=abc123

  2. Próximas requisições:
     Cliente ──► GET /dados (Cookie: sessao=abc123)
     Servidor ──► Busca sessão abc123 → "Ah, é a Maria!"
     Cliente ◄── 200 OK (dados da Maria)

  3. Logout:
     Cliente ──► POST /logout (Cookie: sessao=abc123)
     Servidor ──► Deleta sessão abc123
     Cliente ◄── Set-Cookie: sessao=; Max-Age=0
```

### Sessão vs Token (JWT)

| Característica | Sessão (Cookie) | Token (JWT) |
|---|---|---|
| Onde fica o dado? | No **servidor** | No **token** (cliente) |
| Escalabilidade | Difícil (servidor guarda estado) | Fácil (stateless) |
| Revogar acesso | Fácil (deletar sessão) | Difícil (token já emitido) |
| Tamanho | Cookie pequeno | Token pode ser grande |
| Uso principal | Sites tradicionais | APIs, SPAs, Mobile |

---

## 3️⃣ Cache HTTP

O cache evita **fazer a mesma requisição** quando os dados não mudaram. Faz tudo mais rápido!

### 💡 Analogia: Geladeira

```
  Sem cache: Toda vez que quer leite, VAI AO MERCADO
  Com cache: Compra uma vez, PEGA NA GELADEIRA (mais rápido!
             Só volta ao mercado quando acabar)
```

### Headers de cache

```
  ┌──────────────────────────────────────────────────────┐
  │  PRIMEIRA REQUISIÇÃO                                  │
  │                                                       │
  │  Request:  GET /api/dados                             │
  │  Response: 200 OK                                     │
  │            Cache-Control: max-age=3600                │
  │            ETag: "abc123"                             │
  │            Last-Modified: Wed, 18 Feb 2026 10:00      │
  │            { ...dados... }                            │
  ├──────────────────────────────────────────────────────┤
  │  DEPOIS (cache ainda válido - dentro de 1h):          │
  │                                                       │
  │  ✅ Usa do cache local! (nem faz requisição)          │
  ├──────────────────────────────────────────────────────┤
  │  DEPOIS (cache expirou - após 1h):                    │
  │                                                       │
  │  Request:  GET /api/dados                             │
  │            If-None-Match: "abc123"                    │
  │                                                       │
  │  Se não mudou:                                        │
  │  Response: 304 Not Modified (sem body = rápido!)      │
  │                                                       │
  │  Se mudou:                                            │
  │  Response: 200 OK + novos dados + novo ETag           │
  └──────────────────────────────────────────────────────┘
```

### Estratégias de cache

| Estratégia | Cache-Control | Quando usar |
|---|---|---|
| Sem cache | `no-store` | Dados sensíveis (bancário, médico) |
| Revalidar sempre | `no-cache` | Dados que mudam frequentemente |
| Cache curto | `max-age=300` (5 min) | APIs de dados |
| Cache médio | `max-age=3600` (1 hora) | Conteúdo que muda pouco |
| Cache longo | `max-age=31536000` (1 ano) | CSS, JS, imagens com hash |

```bash
# Verificar cache de um recurso
curl -I https://cdn.example.com/style.css | grep -i cache

# Forçar recarregamento (ignorar cache)
curl -H "Cache-Control: no-cache" https://httpbin.org/cache
```

---

## 4️⃣ CORS — Cross-Origin Resource Sharing

CORS é o mecanismo que **controla quais sites podem acessar uma API**. É a dor de cabeça número 1 de todo desenvolvedor frontend.

### 💡 Analogia: Portaria de condomínio

```
  Seu site (meusite.com) quer acessar a API (api.exemplo.com)

  Sem CORS:
  "Qualquer um pode entrar no condomínio" ← inseguro

  Com CORS:
  "Só moradores (e visitantes autorizados) podem entrar"
```

### Como funciona

```
  meusite.com                              api.exemplo.com
       │                                         │
       │── GET /dados ─────────────────────────►│
       │   Origin: https://meusite.com           │
       │                                         │
       │◄── 200 OK ──────────────────────────── │
       │    Access-Control-Allow-Origin:         │
       │    https://meusite.com                  │
       │                                         │
       │    ✅ Navegador permite o acesso!        │
```

### O erro de CORS mais famoso

```
Access to fetch at 'https://api.exemplo.com/dados'
from origin 'http://localhost:3000' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

### Preflight Request (OPTIONS)

Para requisições "complexas" (POST com JSON, headers customizados), o navegador faz uma **verificação prévia**:

```
  Navegador                                  Servidor
     │                                          │
     │── OPTIONS /api/dados ──────────────────►│  ← Preflight
     │   Origin: https://meusite.com            │
     │   Access-Control-Request-Method: POST    │
     │   Access-Control-Request-Headers:        │
     │   Content-Type                           │
     │                                          │
     │◄── 204 No Content ────────────────────  │
     │    Access-Control-Allow-Origin: *        │
     │    Access-Control-Allow-Methods:         │
     │    GET, POST, PUT, DELETE                │
     │    Access-Control-Allow-Headers:         │
     │    Content-Type, Authorization           │
     │                                          │
     │── POST /api/dados ─────────────────────►│  ← Requisição real
     │   Content-Type: application/json         │
     │   {"nome": "Maria"}                      │
     │                                          │
     │◄── 201 Created ────────────────────────│
```

### Headers CORS

| Header | Quem envia | O que faz |
|---|---|---|
| `Origin` | Cliente | "Estou vindo de tal site" |
| `Access-Control-Allow-Origin` | Servidor | "Aceito requisições de tal site" |
| `Access-Control-Allow-Methods` | Servidor | "Aceito esses métodos" |
| `Access-Control-Allow-Headers` | Servidor | "Aceito esses headers" |
| `Access-Control-Max-Age` | Servidor | "Cache o preflight por X segundos" |

### Como resolver erros de CORS

| Cenário | Solução |
|---|---|
| Sua API, seu servidor | Adicione os headers CORS no servidor |
| API de terceiros | Use um proxy no seu backend |
| Desenvolvimento local | Configure CORS no servidor ou use proxy |
| Não controla o servidor | CORS não tem solução no frontend |

### Exemplo — Configurando CORS no servidor

```javascript
// Node.js com Express
const express = require('express');
const cors = require('cors');
const app = express();

// Permitir qualquer origem (⚠️ só para desenvolvimento!)
app.use(cors());

// Permitir origens específicas (✅ produção)
app.use(cors({
  origin: ['https://meusite.com', 'https://app.meusite.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| `Access-Control-Allow-Origin: *` em produção | Especificar a origem exata |
| Tentar resolver CORS no frontend | CORS é resolvido no **servidor** |
| Desabilitar CORS no navegador | Configurar corretamente no servidor |
| Ignorar o preflight | Configurar resposta adequada para OPTIONS |

---

## 5️⃣ Autenticação HTTP

### Tipos de autenticação

#### Basic Auth

```bash
# Envia usuário:senha em Base64
curl -u usuario:senha https://api.exemplo.com/dados

# Equivale a:
# Authorization: Basic dXN1YXJpbzpzZW5oYQ==
# (base64 de "usuario:senha")
```

⚠️ **NÃO é seguro** sem HTTPS! Qualquer um pode decodificar Base64.

#### Bearer Token (JWT)

```bash
# Token JWT (mais usado em APIs modernas)
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  https://api.exemplo.com/perfil
```

#### API Key

```bash
# Via header
curl -H "X-API-Key: sua-chave-aqui" https://api.exemplo.com/dados

# Via query string (menos seguro)
curl "https://api.exemplo.com/dados?api_key=sua-chave-aqui"
```

#### OAuth 2.0 (Simplificado)

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Usuário  │     │ Sua App  │     │ Google   │
  └──────────┘     └──────────┘     └──────────┘
       │                │                │
       │── "Login com   │                │
       │    Google" ──►│                │
       │                │── "Posso      │
       │                │    acessar?" ─►│
       │                │                │
       │◄── "Permita   │                │
       │    acesso" ────────────────────│
       │                │                │
       │── "Sim!" ─────────────────────►│
       │                │                │
       │                │◄── token ─────│
       │                │                │
       │◄── "Logado!" ──│                │
```

### Comparativo

| Método | Segurança | Complexidade | Quando usar |
|---|---|---|---|
| Basic Auth | ⭐ | Fácil | Testes internos (com HTTPS) |
| API Key | ⭐⭐ | Fácil | APIs simples, serviço a serviço |
| Bearer/JWT | ⭐⭐⭐⭐ | Médio | APIs modernas, SPAs, Mobile |
| OAuth 2.0 | ⭐⭐⭐⭐⭐ | Complexo | "Login com Google/GitHub" |

---

## 6️⃣ Query Strings Avançadas

### Padrões comuns em APIs

```bash
# Paginação
GET /api/produtos?page=2&limit=20

# Ordenação
GET /api/produtos?sort=preco&order=asc

# Filtros
GET /api/produtos?categoria=eletronicos&preco_min=100&preco_max=500

# Busca
GET /api/produtos?q=notebook+dell

# Combinando tudo
GET /api/produtos?q=notebook&categoria=eletronicos&sort=preco&order=asc&page=1&limit=10
```

### Caracteres especiais na URL

| Caractere | Codificação URL | Exemplo |
|---|---|---|
| Espaço | `%20` ou `+` | `?q=meu+produto` |
| `&` | `%26` | `?nome=Tom%26Jerry` |
| `=` | `%3D` | `?formula=2%2B2%3D4` |
| `?` | `%3F` | Já é o separador |
| `/` | `%2F` | `?path=pasta%2Farquivo` |
| `#` | `%23` | `?cor=%23FF0000` |

---

## 7️⃣ Content Negotiation

O cliente e o servidor **negociam** o melhor formato para a resposta.

```bash
# "Quero JSON, mas aceito XML também"
curl -H "Accept: application/json, application/xml;q=0.9" \
  https://httpbin.org/get

# "Quero em português do Brasil"
curl -H "Accept-Language: pt-BR, pt;q=0.9, en;q=0.5" \
  https://httpbin.org/get

# "Aceito resposta comprimida"
curl -H "Accept-Encoding: gzip, deflate, br" \
  https://httpbin.org/gzip
```

### Fator de qualidade (q)

O `q` indica a **preferência** de 0 a 1:

```
Accept: text/html, application/json;q=0.9, text/plain;q=0.5
         └── q=1 (padrão)  └── 90% preferido    └── 50% preferido
```

---

## 🎯 Exercícios

1. Use `curl -c` e `curl -b` para salvar e reenviar cookies de um site
2. Faça uma requisição com Basic Auth para `https://httpbin.org/basic-auth/usuario/senha`
3. Verifique os headers CORS de 3 APIs diferentes usando `curl -I`
4. Crie uma requisição com query string combinando paginação + filtro + ordenação
5. Envie um request com `Accept-Language: pt-BR` para o Google e veja se muda o idioma

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Headers HTTP](06-headers.md) | [Próximo: Tópicos Avançados ➡️](08-topicos-avancados.md)
