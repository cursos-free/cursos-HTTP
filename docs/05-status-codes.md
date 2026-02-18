# 🔢 Códigos de Status HTTP

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Métodos HTTP](04-metodos-http.md) | [Próximo: Headers HTTP ➡️](06-headers.md)

---

> Todo response HTTP vem com um **código de status** — um número de 3 dígitos que diz se deu certo, se deu errado e por quê. Pense neles como os "semáforos" da internet.

---

## 📋 Cola Rápida — Famílias de Status

| Faixa | Categoria | Significado | Cor do semáforo |
|---|---|---|---|
| **1xx** | Informacional | "Recebi, estou processando..." | 🔵 Azul |
| **2xx** | Sucesso | "Deu tudo certo!" | 🟢 Verde |
| **3xx** | Redirecionamento | "O que você quer está em outro lugar" | 🟡 Amarelo |
| **4xx** | Erro do Cliente | "Você fez algo errado" | 🔴 Vermelho |
| **5xx** | Erro do Servidor | "Eu (servidor) fiz algo errado" | 💥 Crítico |

### 💡 Analogia: Pedido no Restaurante

| Código | No restaurante |
|---|---|
| **200** | "Aqui está seu prato!" |
| **201** | "Novo prato criado no cardápio!" |
| **301** | "Mudamos de endereço, vá ao novo local" |
| **400** | "Não entendi seu pedido" |
| **401** | "Precisa se identificar primeiro" |
| **403** | "Sei quem você é, mas não pode entrar aqui" |
| **404** | "Esse prato não existe" |
| **500** | "A cozinha pegou fogo" |

---

## 🟢 2xx — Sucesso

Tudo funcionou como esperado!

| Código | Nome | Quando usar | Exemplo |
|---|---|---|---|
| **200** | OK | Requisição bem-sucedida (geral) | `GET /posts` → lista de posts |
| **201** | Created | Recurso criado com sucesso | `POST /posts` → novo post |
| **204** | No Content | Sucesso, mas sem dados para retornar | `DELETE /posts/1` → deletado |
| **202** | Accepted | Requisição aceita, processando depois | Upload de vídeo grande |
| **206** | Partial Content | Retornando parte do recurso | Download parcial de arquivo |

### Exemplos práticos

```bash
# 200 OK — Buscar dados
curl -s -o /dev/null -w "%{http_code}" \
  https://jsonplaceholder.typicode.com/posts/1
# 200

# 201 Created — Criar recurso
curl -s -o /dev/null -w "%{http_code}" \
  -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "Novo", "body": "Post", "userId": 1}'
# 201

# 204 No Content — Deletar recurso
# (Algumas APIs retornam 200 ao invés de 204)
```

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Retornar 200 ao criar um recurso | Retornar **201 Created** |
| Retornar 200 com body vazio ao deletar | Retornar **204 No Content** |
| Retornar 200 com `{ "erro": "não encontrado" }` | Retornar **404 Not Found** |

---

## 🟡 3xx — Redirecionamento

O recurso que você pediu está em outro lugar.

| Código | Nome | O que acontece | Permanente? |
|---|---|---|---|
| **301** | Moved Permanently | URL mudou para sempre | ✅ Sim |
| **302** | Found | URL temporariamente diferente | ❌ Não |
| **304** | Not Modified | "Nada mudou, use o cache" | — |
| **307** | Temporary Redirect | Igual ao 302, mantém o método | ❌ Não |
| **308** | Permanent Redirect | Igual ao 301, mantém o método | ✅ Sim |

### Como funciona o redirecionamento

```
  Cliente                          Servidor
     │                                │
     │── GET /pagina-antiga ────────►│
     │                                │
     │◄── 301 Moved Permanently ─────│
     │     Location: /pagina-nova     │
     │                                │
     │── GET /pagina-nova ──────────►│  ← navegador segue automaticamente
     │                                │
     │◄── 200 OK ────────────────────│
     │     <html>conteúdo</html>     │
```

### Exemplo prático

```bash
# Seguir redirecionamentos automaticamente (-L)
curl -L http://google.com
# Redireciona http → https → www.google.com

# Ver o redirecionamento sem seguir
curl -I http://google.com
# HTTP/1.1 301 Moved Permanently
# Location: http://www.google.com/
```

### 304 Not Modified — Cache

```bash
# Primeira requisição: servidor retorna dados + ETag
curl -I https://jsonplaceholder.typicode.com/posts/1
# ETag: "some-hash-value"

# Segunda requisição: "ainda é o mesmo?"
curl -H "If-None-Match: \"some-hash-value\"" \
  https://jsonplaceholder.typicode.com/posts/1
# Se não mudou: 304 Not Modified (sem body = mais rápido!)
```

---

## 🔴 4xx — Erro do Cliente

O problema está no **seu pedido**. Algo que você enviou está errado.

| Código | Nome | Significado | Causa comum |
|---|---|---|---|
| **400** | Bad Request | Requisição mal formada | JSON inválido, campo obrigatório faltando |
| **401** | Unauthorized | Não autenticado | Falta token/login |
| **403** | Forbidden | Sem permissão | Logado, mas sem acesso |
| **404** | Not Found | Recurso não existe | URL errada |
| **405** | Method Not Allowed | Método não permitido | POST em rota que só aceita GET |
| **408** | Request Timeout | Demorou demais | Conexão lenta |
| **409** | Conflict | Conflito de dados | Email já cadastrado |
| **415** | Unsupported Media Type | Formato não suportado | Enviar XML quando só aceita JSON |
| **422** | Unprocessable Entity | Dados inválidos | Email com formato errado |
| **429** | Too Many Requests | Muitas requisições | Rate limiting |

### 400 Bad Request

```bash
# Enviar JSON inválido
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{titulo: sem aspas}'  # JSON inválido!
# 400 Bad Request
```

### 401 vs 403 — Qual a diferença?

```
  ┌─────────────────────────────────────────────────┐
  │  401 Unauthorized = "Quem é você?"              │
  │  ─────────────────────────────────              │
  │  O servidor não sabe quem você é.               │
  │  Solução: fazer login / enviar token.           │
  │                                                  │
  │  403 Forbidden = "Sei quem é, mas não pode!"    │
  │  ─────────────────────────────────              │
  │  O servidor sabe quem você é,                   │
  │  mas você não tem permissão.                    │
  │  Solução: pedir acesso ao admin.                │
  └─────────────────────────────────────────────────┘
```

Exemplo prático:

```bash
# 401 — Sem autenticação
curl https://api.github.com/user
# 401 Unauthorized — "Quem é você?"

# 403 — Autenticado, mas sem permissão
curl -H "Authorization: Bearer meu-token" \
  https://api.exemplo.com/admin/painel
# 403 Forbidden — "Você não é admin!"
```

### 404 Not Found

```bash
# Recurso que não existe
curl -s -o /dev/null -w "%{http_code}" \
  https://jsonplaceholder.typicode.com/posts/99999
# 404
```

### 429 Too Many Requests

```
  Requisição 1: ✅ 200 OK
  Requisição 2: ✅ 200 OK
  Requisição 3: ✅ 200 OK
  ...
  Requisição 100: ❌ 429 Too Many Requests
  Header: Retry-After: 60  ← "Tente novamente em 60 segundos"
```

---

## 💥 5xx — Erro do Servidor

O problema está no **servidor**, não em você.

| Código | Nome | Significado | Causa comum |
|---|---|---|---|
| **500** | Internal Server Error | Erro genérico no servidor | Bug no código do servidor |
| **501** | Not Implemented | Funcionalidade não implementada | Método não suportado |
| **502** | Bad Gateway | Servidor intermediário recebeu resposta inválida | Proxy/load balancer com problema |
| **503** | Service Unavailable | Servidor temporariamente indisponível | Manutenção, sobrecarga |
| **504** | Gateway Timeout | Servidor intermediário não recebeu resposta a tempo | Backend lento |

### 500 vs 502 vs 503 vs 504

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Cliente  │ ──► │  Nginx   │ ──► │   App    │
  │          │     │ (proxy)  │     │ (backend)│
  └──────────┘     └──────────┘     └──────────┘

  500 → A App deu erro (bug no código)
  502 → O Nginx recebeu resposta estranha da App
  503 → A App está fora do ar (manutenção)
  504 → A App demorou demais para responder ao Nginx
```

---

## 🔵 1xx — Informacional (Raro)

| Código | Nome | Significado |
|---|---|---|
| **100** | Continue | "Pode continuar enviando o body" |
| **101** | Switching Protocols | "Vamos trocar de protocolo" (ex: para WebSocket) |
| **103** | Early Hints | "Vai precisar desses recursos, já pode baixar" |

Esses são raramente vistos diretamente — geralmente são tratados automaticamente.

---

## 📊 Tabela Completa de Referência

```
  ┌─────────────────────────────────────────────────────┐
  │           MAPA VISUAL DOS STATUS CODES               │
  ├──────────────┬──────────────────────────────────────┤
  │   1xx 🔵     │  100 Continue                        │
  │  Informação  │  101 Switching Protocols              │
  ├──────────────┼──────────────────────────────────────┤
  │   2xx 🟢     │  200 OK                              │
  │   Sucesso    │  201 Created                          │
  │              │  204 No Content                       │
  ├──────────────┼──────────────────────────────────────┤
  │   3xx 🟡     │  301 Moved Permanently                │
  │  Redirecione │  302 Found                            │
  │              │  304 Not Modified                     │
  ├──────────────┼──────────────────────────────────────┤
  │   4xx 🔴     │  400 Bad Request                      │
  │  Erro Client │  401 Unauthorized                     │
  │              │  403 Forbidden                        │
  │              │  404 Not Found                        │
  │              │  409 Conflict                         │
  │              │  422 Unprocessable Entity              │
  │              │  429 Too Many Requests                 │
  ├──────────────┼──────────────────────────────────────┤
  │   5xx 💥     │  500 Internal Server Error             │
  │  Erro Server │  502 Bad Gateway                       │
  │              │  503 Service Unavailable                │
  │              │  504 Gateway Timeout                    │
  └──────────────┴──────────────────────────────────────┘
```

---

## 🎯 Como Verificar Status Codes

```bash
# Só o status code
curl -s -o /dev/null -w "%{http_code}\n" https://google.com

# Headers completos (inclui status)
curl -I https://google.com

# Verbose (tudo)
curl -v https://google.com 2>&1 | grep "< HTTP"
```

---

## ❌ Mitos sobre Status Codes

| ❌ Mito | ✅ Verdade |
|---|---|
| "404 = o site não existe" | 404 = aquela **página** não existe (o site pode estar funcionando) |
| "500 = o site caiu" | 500 = erro no **processamento** (pode ser só uma rota específica) |
| "200 = tudo certo" | 200 só significa que o servidor **respondeu** — o conteúdo pode ter um erro lógico |
| "Toda API retorna os mesmos codes" | Cada API define seus próprios padrões de resposta |

---

## 🎯 Exercícios

1. Use `curl -I` para verificar o status code de 5 sites diferentes
2. Tente acessar `https://httpstat.us/503` — qual status code retorna?
3. Acesse `https://httpstat.us/301` com e sem a flag `-L` do curl — qual a diferença?
4. Faça um POST com JSON inválido e observe o status code
5. Acesse `https://httpstat.us/418` — o que é esse código? (sim, ele existe! 🫖)

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Métodos HTTP](04-metodos-http.md) | [Próximo: Headers HTTP ➡️](06-headers.md)
