# 🚀 Tópicos Avançados

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Tópicos Intermediários](07-topicos-intermediarios.md) | [Próximo: Boas Práticas ➡️](09-boas-praticas.md)

---

> Estes são os tópicos que separam o iniciante do profissional. Aqui cobrimos HTTPS/TLS, HTTP/2 e 3, WebSockets, APIs REST e performance — tudo que o mercado pede.

---

## 📋 Cola Rápida — Tópicos Avançados

| Tópico | Relevância no mercado | Dificuldade |
|---|---|---|
| **HTTPS / TLS** | ⭐⭐⭐⭐⭐ Obrigatório | 🟡 Médio |
| **HTTP/2** | ⭐⭐⭐⭐⭐ Padrão atual | 🟡 Médio |
| **HTTP/3 / QUIC** | ⭐⭐⭐⭐ Crescendo | 🔴 Avançado |
| **WebSockets** | ⭐⭐⭐⭐ Tempo real | 🟡 Médio |
| **APIs REST** | ⭐⭐⭐⭐⭐ Essencial | 🟢 Fácil-Médio |
| **Compressão** | ⭐⭐⭐⭐ Performance | 🟢 Fácil |
| **Rate Limiting** | ⭐⭐⭐⭐ Proteção | 🟡 Médio |
| **Webhooks** | ⭐⭐⭐⭐ Integração | 🟡 Médio |

---

## 1️⃣ HTTPS e TLS

### O que é HTTPS?

HTTPS = HTTP + **TLS** (Transport Layer Security). O TLS **criptografa** toda a comunicação entre cliente e servidor.

```
  HTTP:
  Cliente ──── "senha: 123456" ────────────► Servidor
               👀 Hackers podem ler!

  HTTPS:
  Cliente ──── "a8K$2mX!p9..." ────────────► Servidor
               🔒 Impossível de ler!        (descriptografa)
```

### Como funciona o TLS Handshake

```
  Cliente                                    Servidor
     │                                          │
     │── Client Hello ─────────────────────────►│
     │   (versões TLS suportadas,               │
     │    cifras suportadas)                    │
     │                                          │
     │◄── Server Hello ────────────────────────│
     │   (versão escolhida, cifra,              │
     │    certificado digital 📜)               │
     │                                          │
     │   Verifica certificado ✅                │
     │   (É realmente o Google?)                │
     │                                          │
     │── Troca de chaves ──────────────────────►│
     │   (geram chave secreta compartilhada 🔑) │
     │                                          │
     │◄── Confirmação ─────────────────────────│
     │                                          │
     │══ 🔒 CONEXÃO CRIPTOGRAFADA 🔒 ═════════│
     │   Agora sim: HTTP dentro do TLS          │
```

### Certificados SSL/TLS

| Tipo | Valida | Custo | Exemplo |
|---|---|---|---|
| **DV** (Domain Validation) | Domínio | Grátis (Let's Encrypt) | Blogs, sites pequenos |
| **OV** (Organization Validation) | Domínio + Empresa | $$ | Empresas |
| **EV** (Extended Validation) | Verificação completa | $$$ | Bancos, e-commerce |

### Verificando HTTPS

```bash
# Ver detalhes do certificado de um site
curl -v https://google.com 2>&1 | grep -A5 "Server certificate"

# Verificar se o certificado é válido
openssl s_client -connect google.com:443 -brief

# Testar com SSL Labs (online)
# https://www.ssllabs.com/ssltest/
```

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Usar HTTP em produção | **Sempre HTTPS** em produção |
| Certificado expirado | Renovar automaticamente (Let's Encrypt) |
| TLS 1.0 ou 1.1 | Usar **TLS 1.2** ou **1.3** |
| Ignorar avisos de certificado | Investigar e corrigir o problema |

---

## 2️⃣ HTTP/2 — O Salto de Performance

### Problemas do HTTP/1.1

```
  HTTP/1.1: Uma requisição por vez (por conexão)
  ┌──────────────────────────────────────────┐
  │  Conexão 1: GET /style.css ──────────►   │
  │             ◄──────── resposta           │
  │             GET /app.js ──────────►      │  ← espera anterior terminar!
  │             ◄──────── resposta           │
  │  Conexão 2: GET /logo.png ──────────►    │  ← abre outra conexão
  │             ◄──────── resposta           │
  └──────────────────────────────────────────┘

  HTTP/2: Múltiplas requisições simultâneas (multiplexação)
  ┌──────────────────────────────────────────┐
  │  Conexão 1: GET /style.css ──────────►   │
  │             GET /app.js ──────────►      │  ← ao mesmo tempo!
  │             GET /logo.png ──────────►    │  ← ao mesmo tempo!
  │             ◄──────── style.css          │
  │             ◄──────── logo.png           │
  │             ◄──────── app.js             │
  └──────────────────────────────────────────┘
```

### Melhorias do HTTP/2

| Feature | HTTP/1.1 | HTTP/2 |
|---|---|---|
| **Formato** | Texto | Binário (mais eficiente) |
| **Multiplexação** | ❌ 1 request por conexão | ✅ Múltiplos por conexão |
| **Compressão de Headers** | ❌ Headers completos toda vez | ✅ HPACK (comprime headers) |
| **Server Push** | ❌ Só responde | ✅ Pode enviar antes de pedir |
| **Priorização** | ❌ Não | ✅ Define prioridades |
| **Conexões** | 6-8 por domínio | 1 conexão multiplexada |

### Verificando se um site usa HTTP/2

```bash
# Com curl
curl -sI https://google.com | head -1
# HTTP/2 200

# Forçar HTTP/1.1
curl --http1.1 -sI https://google.com | head -1
# HTTP/1.1 200 OK

# Forçar HTTP/2
curl --http2 -sI https://google.com | head -1
# HTTP/2 200
```

### Server Push (HTTP/2)

```
  Navegador: "Quero a página /index.html"
  Servidor:  "Aqui está! E já estou mandando o style.css e app.js também,
              sei que vai precisar."

  Sem Push:                   Com Push:
  GET index.html              GET index.html
  ← index.html               ← index.html
  GET style.css               ← style.css (push!)
  ← style.css                 ← app.js (push!)
  GET app.js                  Tempo total: ~1 ida
  ← app.js
  Tempo total: ~3 idas
```

---

## 3️⃣ HTTP/3 e QUIC

### O problema do HTTP/2

O HTTP/2 usa **TCP**, que tem um problema: se um pacote se perde, **tudo** trava esperando a retransmissão (Head-of-Line Blocking na camada TCP).

### Solução: QUIC

O HTTP/3 usa **QUIC** (baseado em UDP), que resolve isso:

```
  HTTP/2 (TCP):
  Stream A: ████░████████    ← Stream A travou, B e C esperam
  Stream B:      ░░░░░████
  Stream C:      ░░░░░░░██

  HTTP/3 (QUIC):
  Stream A: ████░████████    ← Stream A travou sozinha
  Stream B: ██████████       ← B e C continuam normal!
  Stream C: ████████
```

| Feature | HTTP/2 (TCP) | HTTP/3 (QUIC) |
|---|---|---|
| **Transporte** | TCP | QUIC (UDP) |
| **Handshake** | TCP + TLS (2-3 RTT) | 1 RTT (ou 0-RTT!) |
| **Head-of-Line Blocking** | Na camada TCP | ❌ Resolvido |
| **Migração de conexão** | ❌ IP mudou = reconectar | ✅ Mantém conexão |
| **Criptografia** | Opcional (TLS) | Obrigatória (built-in) |

### Verificando HTTP/3

```bash
# Verificar se um site suporta HTTP/3
curl --http3 -sI https://google.com 2>&1 | head -1
# HTTP/3 200

# Ou verificar o header Alt-Svc
curl -sI https://google.com | grep -i alt-svc
# alt-svc: h3=":443"
```

---

## 4️⃣ WebSockets

HTTP é **request-response**: o cliente pergunta, o servidor responde. E se o servidor precisar enviar dados **sem o cliente pedir**? (chat, notificações, dados em tempo real)

### HTTP vs WebSocket

```
  HTTP (half-duplex):
  Cliente ──► "Tem mensagem?" ──► Servidor
  Cliente ◄── "Não."          ◄── Servidor
  Cliente ──► "E agora?"      ──► Servidor
  Cliente ◄── "Sim! Uma!"     ◄── Servidor
  (fica perguntando = polling, ineficiente)

  WebSocket (full-duplex):
  Cliente ──► "Quero conexão WS" ──► Servidor
  Cliente ◄── "Aceito! Conexão  ◄── Servidor
               aberta! 🔌"
  Cliente ◄── "Nova mensagem!"  ◄── Servidor  (envia quando quiser!)
  Cliente ──► "Minha resposta"  ──► Servidor  (ao mesmo tempo!)
  Cliente ◄── "Outra mensagem!" ◄── Servidor
```

### Handshake WebSocket

O WebSocket **começa como HTTP** e "atualiza" a conexão:

```
  Request:
  GET /chat HTTP/1.1
  Host: servidor.com
  Upgrade: websocket              ← "Quero atualizar para WebSocket"
  Connection: Upgrade
  Sec-WebSocket-Key: dGhlIHNhb...

  Response:
  HTTP/1.1 101 Switching Protocols
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Accept: s3pPLM...

  ✅ Agora é WebSocket! Comunicação bidirecional.
```

### Exemplo com JavaScript

```javascript
// Conectar ao WebSocket
const ws = new WebSocket('wss://echo.websocket.org');

// Quando a conexão abrir
ws.onopen = () => {
  console.log('Conectado!');
  ws.send('Olá, servidor!');
};

// Quando receber mensagem
ws.onmessage = (evento) => {
  console.log('Recebido:', evento.data);
};

// Quando fechar
ws.onclose = () => {
  console.log('Desconectado');
};

// Quando der erro
ws.onerror = (erro) => {
  console.error('Erro:', erro);
};
```

### Quando usar WebSocket vs HTTP

| Cenário | Usar |
|---|---|
| Chat em tempo real | ✅ WebSocket |
| Buscar dados de vez em quando | HTTP |
| Dashboard com dados ao vivo | ✅ WebSocket |
| Formulário de cadastro | HTTP |
| Jogo multiplayer | ✅ WebSocket |
| Download de arquivo | HTTP |
| Notificações push | ✅ WebSocket / SSE |

### Server-Sent Events (SSE) — Alternativa mais simples

Se só o servidor precisa enviar dados (unidirecional):

```javascript
// Cliente recebe atualizações do servidor
const fonte = new EventSource('/api/atualizacoes');

fonte.onmessage = (evento) => {
  console.log('Nova atualização:', evento.data);
};
```

```
  SSE: Servidor ────► Cliente (só uma direção)
  WebSocket: Servidor ◄────► Cliente (duas direções)
```

---

## 5️⃣ APIs REST

REST (Representational State Transfer) é o **padrão mais usado** para projetar APIs HTTP.

### Princípios REST

| Princípio | Significado |
|---|---|
| **Client-Server** | Separação clara entre cliente e servidor |
| **Stateless** | Cada request é independente |
| **Cacheable** | Respostas podem ser cacheadas |
| **Uniform Interface** | URLs e métodos padronizados |
| **Layered System** | Pode ter camadas intermediárias (proxy, CDN) |

### Estrutura de uma API REST

```
  ┌───────────────────────────────────────────────────┐
  │              API REST: /api/usuarios               │
  ├──────────┬──────────────────┬──────────────────────┤
  │  Método  │  Endpoint        │  Ação                │
  ├──────────┼──────────────────┼──────────────────────┤
  │  GET     │  /usuarios       │  Listar todos        │
  │  GET     │  /usuarios/42    │  Buscar um           │
  │  POST    │  /usuarios       │  Criar novo          │
  │  PUT     │  /usuarios/42    │  Atualizar completo  │
  │  PATCH   │  /usuarios/42    │  Atualizar parcial   │
  │  DELETE  │  /usuarios/42    │  Remover             │
  ├──────────┼──────────────────┼──────────────────────┤
  │  GET     │  /usuarios/42/   │  Listar posts do     │
  │          │  posts           │  usuário 42          │
  └──────────┴──────────────────┴──────────────────────┘
```

### Exemplo completo de API REST

```bash
# --- LISTAR TODOS ---
curl https://jsonplaceholder.typicode.com/users
# 200 OK → [...lista de usuários...]

# --- BUSCAR UM ---
curl https://jsonplaceholder.typicode.com/users/1
# 200 OK → { "id": 1, "name": "Leanne Graham", ... }

# --- CRIAR ---
curl -X POST https://jsonplaceholder.typicode.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Maria", "email": "maria@email.com"}'
# 201 Created → { "id": 11, "name": "Maria", ... }

# --- ATUALIZAR ---
curl -X PATCH https://jsonplaceholder.typicode.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Leanne Updated"}'
# 200 OK → { "id": 1, "name": "Leanne Updated", ... }

# --- DELETAR ---
curl -X DELETE https://jsonplaceholder.typicode.com/users/1
# 200 OK

# --- RECURSO ANINHADO ---
curl https://jsonplaceholder.typicode.com/users/1/posts
# 200 OK → [...posts do usuário 1...]
```

### ❌ Errado vs ✅ Certo (Design de API REST)

| ❌ Errado | ✅ Certo |
|---|---|
| `GET /getUsuarios` | `GET /usuarios` |
| `POST /criarUsuario` | `POST /usuarios` |
| `GET /deletarUsuario?id=1` | `DELETE /usuarios/1` |
| `POST /usuarios/1/atualizar` | `PUT /usuarios/1` |
| `/Usuarios` (maiúscula) | `/usuarios` (minúscula) |
| `/usuario` (singular) | `/usuarios` (plural) |
| Retornar 200 com `{"erro": "não encontrado"}` | Retornar `404 Not Found` |

---

## 6️⃣ Compressão HTTP

A compressão reduz o tamanho dos dados trafegados, tornando tudo mais rápido.

### Como funciona

```
  Cliente                                   Servidor
     │                                         │
     │── GET /dados ─────────────────────────►│
     │   Accept-Encoding: gzip, br             │
     │   ("Aceito dados comprimidos")          │
     │                                         │
     │◄── 200 OK ─────────────────────────────│
     │    Content-Encoding: gzip               │
     │    Content-Length: 1234 (era 5678!)      │
     │    (dados comprimidos)                  │
     │                                         │
     │    Navegador descomprime automaticamente │
```

### Algoritmos de compressão

| Algoritmo | Header | Compressão | Velocidade | Suporte |
|---|---|---|---|---|
| **gzip** | `gzip` | Boa | Rápida | Universal |
| **Brotli** | `br` | Excelente | Médio | Moderno |
| **deflate** | `deflate` | Boa | Rápida | Universal |
| **zstd** | `zstd` | Excelente | Rápida | Novo |

```bash
# Requisição com compressão
curl -H "Accept-Encoding: gzip" https://httpbin.org/gzip

# Ver se a resposta veio comprimida
curl -sI https://google.com | grep -i content-encoding
# content-encoding: gzip
```

---

## 7️⃣ Rate Limiting

Servidores limitam quantas requisições você pode fazer em um período para evitar abusos.

### Como funciona

```
  Requisições:  ████████████████████░░░░░░░░░░
                └── 20 de 100 usadas ──►  80 restantes

  Headers de Rate Limit:
  X-RateLimit-Limit: 100        ← Limite total
  X-RateLimit-Remaining: 80     ← Quanto falta
  X-RateLimit-Reset: 1708200000 ← Quando reseta (timestamp)
```

### Quando estoura o limite

```bash
# Muitas requisições...
curl https://api.github.com/users/octocat
# 200 OK
# X-RateLimit-Remaining: 1

curl https://api.github.com/users/octocat
# 200 OK
# X-RateLimit-Remaining: 0

curl https://api.github.com/users/octocat
# 429 Too Many Requests
# Retry-After: 60   ← "Tente de novo em 60 segundos"
```

### Como lidar

```python
import requests
import time

def requisicao_segura(url):
    resposta = requests.get(url)
    
    if resposta.status_code == 429:
        retry_after = int(resposta.headers.get('Retry-After', 60))
        print(f"Rate limited! Esperando {retry_after}s...")
        time.sleep(retry_after)
        return requisicao_segura(url)  # Tenta de novo
    
    # Mostra quanto resta
    restante = resposta.headers.get('X-RateLimit-Remaining')
    if restante:
        print(f"Requisições restantes: {restante}")
    
    return resposta
```

---

## 8️⃣ Webhooks

Webhook é o **inverso de uma API**: ao invés de você ficar perguntando, o servidor te avisa quando algo acontece.

```
  API (Polling):
  Você  ──► "Tem pedido novo?"  ──► Loja   (a cada 5 segundos)
  Você  ◄── "Não."             ◄── Loja
  Você  ──► "E agora?"         ──► Loja
  Você  ◄── "Sim! Pedido #42!" ◄── Loja   (gastou muitas requisições)

  Webhook:
  Você  ──► "Me avisa quando tiver pedido" ──► Loja (1 vez)
  ...
  Loja  ──► POST /meu-webhook               ──► Você  (quando acontece!)
             {"evento": "pedido_criado", "id": 42}
```

### Exemplo: Recebendo um Webhook

```javascript
// Servidor que recebe webhooks
const express = require('express');
const app = express();
app.use(express.json());

app.post('/webhook', (req, res) => {
  const evento = req.body;
  
  console.log('Webhook recebido:', evento);
  
  // Processar o evento
  switch (evento.tipo) {
    case 'pagamento_aprovado':
      console.log(`Pagamento ${evento.id} aprovado!`);
      break;
    case 'pedido_criado':
      console.log(`Novo pedido: ${evento.pedido_id}`);
      break;
  }
  
  // IMPORTANTE: Responder rápido (200 OK)
  res.status(200).json({ recebido: true });
});

app.listen(3000);
```

---

## 🎯 Exercícios

1. Use `curl -v https://google.com` e identifique a versão do HTTP e TLS usados
2. Compare o tempo de `curl --http1.1` vs `curl --http2` para o mesmo site
3. Verifique se `google.com` suporta HTTP/3 olhando o header `Alt-Svc`
4. Crie um mini servidor WebSocket usando Node.js
5. Projete uma API REST para um sistema de "tarefas" (to-do list) com todos os endpoints

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Tópicos Intermediários](07-topicos-intermediarios.md) | [Próximo: Boas Práticas ➡️](09-boas-praticas.md)
