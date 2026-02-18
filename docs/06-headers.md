# 📋 Headers HTTP

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Códigos de Status](05-status-codes.md) | [Próximo: Tópicos Intermediários ➡️](07-topicos-intermediarios.md)

---

> Headers (cabeçalhos) são **metadados** que acompanham toda requisição e resposta HTTP. São como as informações escritas no **envelope** de uma carta — não são o conteúdo em si, mas dizem coisas importantes sobre ele.

---

## 📋 Cola Rápida — Headers Mais Usados

### Headers de Request (Cliente → Servidor)

| Header | Exemplo | Para que serve |
|---|---|---|
| `Host` | `Host: api.exemplo.com` | Qual servidor acessar |
| `Accept` | `Accept: application/json` | Que formato quer receber |
| `Content-Type` | `Content-Type: application/json` | Formato dos dados enviados |
| `Authorization` | `Authorization: Bearer abc123` | Autenticação/identificação |
| `User-Agent` | `User-Agent: Mozilla/5.0...` | Identifica o cliente |
| `Accept-Language` | `Accept-Language: pt-BR` | Idioma preferido |
| `Cookie` | `Cookie: sessao=abc123` | Dados de sessão/preferências |
| `Cache-Control` | `Cache-Control: no-cache` | Instruções de cache |

### Headers de Response (Servidor → Cliente)

| Header | Exemplo | Para que serve |
|---|---|---|
| `Content-Type` | `Content-Type: text/html` | Formato da resposta |
| `Content-Length` | `Content-Length: 1234` | Tamanho em bytes |
| `Set-Cookie` | `Set-Cookie: sessao=abc` | Define cookie no cliente |
| `Cache-Control` | `Cache-Control: max-age=3600` | Quanto tempo cachear |
| `Location` | `Location: /nova-url` | Para onde redirecionar |
| `Access-Control-Allow-Origin` | `Access-Control-Allow-Origin: *` | CORS — quem pode acessar |
| `X-RateLimit-Remaining` | `X-RateLimit-Remaining: 58` | Requisições restantes |

---

## 1️⃣ Como Ver os Headers

### Com curl

```bash
# Ver TODOS os headers (request e response)
curl -v https://jsonplaceholder.typicode.com/posts/1

# Ver apenas os headers da RESPONSE
curl -I https://jsonplaceholder.typicode.com/posts/1

# Resultado:
# HTTP/2 200
# date: Wed, 18 Feb 2026 10:00:00 GMT
# content-type: application/json; charset=utf-8
# content-length: 292
# cache-control: max-age=43200
# etag: "some-hash"
```

### Com JavaScript

```javascript
const resposta = await fetch('https://jsonplaceholder.typicode.com/posts/1');

// Ver um header específico
console.log(resposta.headers.get('Content-Type'));
// application/json; charset=utf-8

// Ver todos os headers
for (const [chave, valor] of resposta.headers) {
  console.log(`${chave}: ${valor}`);
}
```

### Com Python

```python
import requests

resposta = requests.get('https://jsonplaceholder.typicode.com/posts/1')

# Ver todos os headers
print(dict(resposta.headers))

# Ver um header específico
print(resposta.headers['Content-Type'])
# application/json; charset=utf-8
```

---

## 2️⃣ Content-Type — O Header Mais Importante

O `Content-Type` diz **qual formato** os dados estão. Sem ele, o receptor não sabe como interpretar.

### Tipos mais comuns (MIME Types)

| Content-Type | Formato | Quando usar |
|---|---|---|
| `application/json` | JSON | APIs REST (90% dos casos) |
| `text/html` | HTML | Páginas web |
| `text/plain` | Texto puro | Texto sem formatação |
| `text/css` | CSS | Folhas de estilo |
| `application/javascript` | JavaScript | Scripts |
| `application/xml` | XML | APIs mais antigas |
| `multipart/form-data` | Formulário com arquivo | Upload de arquivos |
| `application/x-www-form-urlencoded` | Formulário simples | Formulários HTML tradicionais |
| `image/png` | Imagem PNG | Imagens |
| `image/jpeg` | Imagem JPEG | Fotos |
| `application/pdf` | PDF | Documentos |
| `application/octet-stream` | Binário genérico | Downloads |

### Exemplo prático

```bash
# Enviar JSON — PRECISA do Content-Type
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"nome": "Maria"}'

# Enviar formulário
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "nome=Maria&email=maria@email.com"

# Upload de arquivo
curl -X POST https://httpbin.org/post \
  -H "Content-Type: multipart/form-data" \
  -F "arquivo=@foto.jpg"
```

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Enviar JSON sem `Content-Type` | `Content-Type: application/json` |
| Enviar `text/plain` com JSON | Usar `application/json` |
| Esquecer `;charset=utf-8` com texto | `text/html; charset=utf-8` |

---

## 3️⃣ Accept — O que Eu Quero Receber

O header `Accept` diz ao servidor **em qual formato** você quer a resposta.

```bash
# "Quero receber JSON"
curl -H "Accept: application/json" https://httpbin.org/get

# "Quero receber HTML"
curl -H "Accept: text/html" https://httpbin.org/get

# "Quero receber XML"
curl -H "Accept: application/xml" https://httpbin.org/get

# "Aceito qualquer coisa" (padrão)
curl -H "Accept: */*" https://httpbin.org/get
```

### Content-Type vs Accept

```
  ┌─────────────────────────────────────────────────┐
  │  Accept = "O que eu QUERO receber" (request)    │
  │  Content-Type = "O que eu ESTOU enviando"       │
  │                                                  │
  │  Request:                                        │
  │    Accept: application/json  ← quero JSON        │
  │    Content-Type: application/json ← envio JSON   │
  │                                                  │
  │  Response:                                       │
  │    Content-Type: application/json ← recebi JSON  │
  └─────────────────────────────────────────────────┘
```

---

## 4️⃣ Authorization — Autenticação

O header `Authorization` envia credenciais para provar quem você é.

### Tipos mais comuns

| Tipo | Formato | Uso |
|---|---|---|
| **Bearer Token** | `Authorization: Bearer eyJhbG...` | APIs modernas (JWT) |
| **Basic** | `Authorization: Basic dXN1YXJpbzpzZW5oYQ==` | Usuário + senha (base64) |
| **API Key** | `X-API-Key: abc123` | Chave de API (header customizado) |

### Exemplos

```bash
# Bearer Token (mais comum em APIs)
curl -H "Authorization: Bearer meu-token-jwt-aqui" \
  https://api.exemplo.com/perfil

# Basic Auth (usuário:senha em base64)
curl -u usuario:senha https://api.exemplo.com/dados
# Equivale a: Authorization: Basic dXN1YXJpbzpzZW5oYQ==

# API Key via header customizado
curl -H "X-API-Key: minha-chave-aqui" \
  https://api.exemplo.com/dados
```

### ⚠️ Cuidados de segurança

```
  ❌ NUNCA faça isso:
  GET /api/dados?token=abc123    ← Token na URL (fica no log!)
  
  ✅ SEMPRE faça isso:
  Authorization: Bearer abc123   ← Token no header (seguro)
```

---

## 5️⃣ User-Agent — Quem Está Pedindo

O `User-Agent` identifica **qual aplicação** está fazendo a requisição.

```bash
# Ver seu User-Agent
curl https://httpbin.org/user-agent

# Simular um navegador
curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0" \
  https://httpbin.org/user-agent

# Identificar seu script
curl -H "User-Agent: MeuApp/1.0" https://httpbin.org/user-agent
```

### User-Agents comuns

| Aplicação | User-Agent (resumido) |
|---|---|
| Chrome | `Mozilla/5.0 ... Chrome/120.0` |
| Firefox | `Mozilla/5.0 ... Firefox/120.0` |
| curl | `curl/8.5.0` |
| Python requests | `python-requests/2.31` |
| Bot do Google | `Googlebot/2.1` |

---

## 6️⃣ Cache-Control — Controle de Cache

Diz ao navegador se pode guardar uma cópia da resposta e por quanto tempo.

### Valores mais comuns

| Valor | Significado |
|---|---|
| `max-age=3600` | Cache válido por 1 hora (3600 seg) |
| `no-cache` | Pode cachear, mas valide antes de usar |
| `no-store` | NÃO cache de jeito nenhum |
| `public` | Qualquer cache pode guardar |
| `private` | Só o navegador do usuário pode cachear |
| `must-revalidate` | Quando expirar, deve revalidar |

### Exemplos

```bash
# Ver header de cache
curl -I https://jsonplaceholder.typicode.com/posts/1 | grep -i cache

# Requisição sem cache
curl -H "Cache-Control: no-cache" https://httpbin.org/cache

# Verificar idade do cache
curl -I https://jsonplaceholder.typicode.com/posts/1 | grep -i age
```

### Fluxo de cache

```
  Requisição 1:
  Cliente ──► Servidor
  Cliente ◄── 200 OK + Cache-Control: max-age=3600
              (Salva no cache local)

  Requisição 2 (dentro de 1 hora):
  Cliente ──► Cache local ✅ (nem vai ao servidor!)

  Requisição 3 (depois de 1 hora):
  Cliente ──► Servidor (cache expirou)
  Cliente ◄── 200 OK + novos dados
```

---

## 7️⃣ CORS Headers — Controle de Acesso

Headers que controlam quais sites podem acessar uma API. Veremos CORS em detalhes no guia intermediário.

```
  Access-Control-Allow-Origin: *              ← Qualquer site pode acessar
  Access-Control-Allow-Origin: https://meusite.com  ← Só esse site
  Access-Control-Allow-Methods: GET, POST     ← Métodos permitidos
  Access-Control-Allow-Headers: Content-Type  ← Headers permitidos
```

---

## 8️⃣ Headers Customizados (X-Headers)

Você pode criar seus próprios headers! Por convenção, headers customizados usavam o prefixo `X-`:

```bash
# Header customizado
curl -H "X-Request-ID: abc-123" \
     -H "X-Client-Version: 2.0" \
     https://httpbin.org/headers
```

> **Nota:** O prefixo `X-` foi oficialmente descontinuado (RFC 6648), mas muitas APIs ainda usam por costume.

### Headers customizados comuns no mercado

| Header | Para que serve |
|---|---|
| `X-Request-ID` | Identificador único da requisição (rastreio) |
| `X-RateLimit-Limit` | Limite de requisições por período |
| `X-RateLimit-Remaining` | Quantas requisições você ainda pode fazer |
| `X-Forwarded-For` | IP original quando passa por proxy |
| `X-Powered-By` | Tecnologia do servidor (muitos removem por segurança) |

---

## 9️⃣ Headers de Segurança

Headers que protegem seu site contra ataques:

| Header | O que faz | Exemplo |
|---|---|---|
| `Strict-Transport-Security` | Força HTTPS | `max-age=31536000; includeSubDomains` |
| `X-Content-Type-Options` | Previne MIME sniffing | `nosniff` |
| `X-Frame-Options` | Previne clickjacking | `DENY` |
| `Content-Security-Policy` | Controla recursos permitidos | `default-src 'self'` |
| `X-XSS-Protection` | Proteção contra XSS (legado) | `1; mode=block` |

```bash
# Verificar headers de segurança de um site
curl -I https://github.com | grep -i "strict\|x-content\|x-frame\|content-security"
```

---

## 📊 Anatomia Completa de uma Requisição

```
  ┌────── REQUEST ──────────────────────────────────────┐
  │  POST /api/usuarios HTTP/1.1                        │  ← Linha de requisição
  │                                                      │
  │  Host: api.exemplo.com                               │  ← Headers
  │  Content-Type: application/json                      │
  │  Accept: application/json                            │
  │  Authorization: Bearer eyJhbGciOiJ...                │
  │  User-Agent: MeuApp/1.0                              │
  │  Accept-Language: pt-BR                              │
  │  Cache-Control: no-cache                             │
  │                                                      │  ← Linha em branco
  │  {                                                   │  ← Body
  │    "nome": "Maria",                                  │
  │    "email": "maria@email.com"                        │
  │  }                                                   │
  └──────────────────────────────────────────────────────┘

  ┌────── RESPONSE ─────────────────────────────────────┐
  │  HTTP/1.1 201 Created                                │  ← Linha de status
  │                                                      │
  │  Content-Type: application/json                      │  ← Headers
  │  Content-Length: 98                                   │
  │  Location: /api/usuarios/42                          │
  │  Cache-Control: no-store                             │
  │  X-Request-ID: req-abc-123                           │
  │                                                      │  ← Linha em branco
  │  {                                                   │  ← Body
  │    "id": 42,                                         │
  │    "nome": "Maria",                                  │
  │    "email": "maria@email.com"                        │
  │  }                                                   │
  └──────────────────────────────────────────────────────┘
```

---

## ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Enviar JSON sem `Content-Type` | Sempre enviar `Content-Type: application/json` |
| Colocar tokens na URL | Usar `Authorization` header |
| Ignorar o `Accept` | Definir o formato que espera receber |
| Expor `X-Powered-By: Express` | Remover headers que vazam informação do servidor |
| Não configurar CORS | Definir `Access-Control-Allow-Origin` adequado |
| `Cache-Control: no-cache` achando que não cacheia | Usar `no-store` se não quer cache algum |

---

## 🎯 Exercícios

1. Use `curl -v` em 3 sites e compare os headers de resposta
2. Faça um POST no `https://httpbin.org/post` com 3 headers customizados
3. Verifique quais headers de segurança o `github.com` usa
4. Descubra o `Content-Type` de uma imagem usando `curl -I`
5. Use o header `Accept-Language: en-US` e `pt-BR` no mesmo site — muda algo?

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Códigos de Status](05-status-codes.md) | [Próximo: Tópicos Intermediários ➡️](07-topicos-intermediarios.md)
