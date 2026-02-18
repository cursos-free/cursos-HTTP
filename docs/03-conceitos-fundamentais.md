# 📖 Conceitos Fundamentais

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Configuração Inicial](02-configuracao-inicial.md) | [Próximo: Métodos HTTP ➡️](04-metodos-http.md)

---

> Antes de mergulhar nos detalhes do HTTP, é essencial entender os conceitos que formam a base da comunicação na internet. Vamos construir esse conhecimento peça por peça.

---

## 📋 Cola Rápida — Conceitos Principais

| Conceito | Explicação em 1 frase |
|---|---|
| **HTTP** | Protocolo (conjunto de regras) para transferir dados na web |
| **Cliente** | Quem pede a informação (navegador, app, curl) |
| **Servidor** | Quem recebe o pedido e devolve a resposta |
| **URL** | O "endereço" do recurso que você quer acessar |
| **DNS** | Traduz nomes de site (google.com) para números IP |
| **TCP/IP** | A "estrada" por onde os dados viajam |
| **Request** | A mensagem que o cliente envia ao servidor |
| **Response** | A mensagem que o servidor devolve ao cliente |
| **Porta** | O "número do apartamento" no servidor |
| **Protocolo** | Conjunto de regras para comunicação |

---

## 1️⃣ O Modelo Cliente-Servidor

Toda comunicação HTTP segue o modelo **cliente-servidor**:

```
  ┌──────────┐          PEDIDO (Request)          ┌──────────┐
  │          │  ─────────────────────────────►    │          │
  │ CLIENTE  │                                    │ SERVIDOR │
  │          │  ◄─────────────────────────────    │          │
  └──────────┘         RESPOSTA (Response)        └──────────┘
```

### 💡 Analogia: Correios

| Correios | HTTP |
|---|---|
| Você escreve uma carta | Cliente cria uma Request |
| Coloca no envelope com endereço | Adiciona URL e Headers |
| Correio entrega ao destinatário | Internet encaminha ao servidor |
| Destinatário lê e responde | Servidor processa e cria Response |
| Você recebe a resposta | Cliente recebe a Response |

### Características importantes

- **O cliente sempre inicia** — o servidor nunca "liga" para o cliente primeiro
- **Uma request, uma response** — cada pedido gera exatamente uma resposta
- **Sem memória (stateless)** — o servidor não lembra de pedidos anteriores por padrão

---

## 2️⃣ O que é uma URL?

**URL** (Uniform Resource Locator) é o endereço completo de um recurso na internet.

### Anatomia de uma URL

```
  https://www.exemplo.com:443/produtos/lista?categoria=livros&pagina=2#resultados
  └─┬──┘ └──────┬───────┘└┬┘ └──────┬─────┘ └──────────┬──────────┘ └────┬────┘
 Protocolo     Host     Porta    Caminho          Query String         Fragmento
```

| Parte | Exemplo | Para que serve |
|---|---|---|
| **Protocolo** | `https://` | Define as regras de comunicação |
| **Host** | `www.exemplo.com` | Nome/endereço do servidor |
| **Porta** | `:443` | "Porta" de entrada no servidor |
| **Caminho (Path)** | `/produtos/lista` | Localização do recurso no servidor |
| **Query String** | `?categoria=livros&pagina=2` | Parâmetros extras (filtros, busca) |
| **Fragmento** | `#resultados` | Posição na página (não vai ao servidor) |

### Portas padrão

| Protocolo | Porta | Precisa digitar? |
|---|---|---|
| HTTP | 80 | Não (é o padrão) |
| HTTPS | 443 | Não (é o padrão) |
| Outras | 3000, 8080, etc. | Sim |

### Exemplos práticos

```
# URL simples
https://google.com

# URL com caminho
https://github.com/usuario/repositorio

# URL com query string (busca no Google)
https://www.google.com/search?q=http+tutorial

# URL com múltiplos parâmetros
https://loja.com/produtos?categoria=eletronicos&preco_max=500&ordenar=preco

# URL com porta customizada (servidor local)
http://localhost:3000/api/usuarios
```

---

## 3️⃣ Como funciona o DNS

O **DNS** (Domain Name System) é como a **agenda de contatos** da internet. Ele traduz nomes que humanos entendem para números que computadores entendem.

```
  Você digita:          DNS resolve:           Servidor real:
  ┌─────────────┐      ┌──────────────┐       ┌──────────────┐
  │ google.com  │ ──►  │ 142.250.79.  │ ──►   │ Servidor do  │
  │ (nome)      │      │ 46 (IP)      │       │ Google       │
  └─────────────┘      └──────────────┘       └──────────────┘
```

### 💡 Analogia: Lista telefônica

- Você não liga para "João" — liga para o **número** dele
- O DNS é a lista que diz: "João = (11) 99999-9999"
- Da mesma forma: "google.com = 142.250.79.46"

### Testando o DNS

```bash
# Descobrir o IP de um site
nslookup google.com

# Ou com dig (mais detalhado)
dig google.com

# Acessar um site pelo IP direto
curl http://142.250.79.46
```

---

## 4️⃣ TCP/IP — A Estrada dos Dados

Os dados na internet viajam por camadas. Pense em uma **encomenda sendo enviada**:

```
  ┌─────────────────────────────────────────────┐
  │              CAMADAS DA INTERNET             │
  ├─────────────────────────────────────────────┤
  │  4. APLICAÇÃO (HTTP, HTTPS)                 │ ← Você está aqui!
  │     "O conteúdo da carta"                   │
  ├─────────────────────────────────────────────┤
  │  3. TRANSPORTE (TCP, UDP)                   │
  │     "O envelope com número de rastreio"     │
  ├─────────────────────────────────────────────┤
  │  2. INTERNET (IP)                           │
  │     "O endereço no envelope"                │
  ├─────────────────────────────────────────────┤
  │  1. REDE/LINK (Ethernet, Wi-Fi)             │
  │     "O caminhão dos correios"               │
  └─────────────────────────────────────────────┘
```

### TCP — Entrega Garantida

O **TCP** (Transmission Control Protocol) garante que os dados cheguem **completos e na ordem certa**.

Antes de trocar dados, cliente e servidor fazem um **"aperto de mão" (handshake)**:

```
  Cliente                     Servidor
     │                           │
     │──── SYN (Oi!) ──────────►│
     │                           │
     │◄─── SYN-ACK (Oi, ok!) ──│
     │                           │
     │──── ACK (Beleza!) ──────►│
     │                           │
     │  ✅ Conexão estabelecida  │
     │   Agora pode trocar HTTP  │
```

---

## 5️⃣ Anatomia de uma Requisição HTTP (Request)

Uma requisição HTTP tem 3 partes:

```
┌──────────────────────────────────────────────┐
│  LINHA DE REQUISIÇÃO (Request Line)          │
│  GET /api/usuarios HTTP/1.1                  │
├──────────────────────────────────────────────┤
│  CABEÇALHOS (Headers)                        │
│  Host: api.exemplo.com                       │
│  Accept: application/json                    │
│  Authorization: Bearer abc123                │
│  User-Agent: Mozilla/5.0                     │
├──────────────────────────────────────────────┤
│  CORPO (Body) — opcional                     │
│  {                                           │
│    "nome": "Maria",                          │
│    "email": "maria@email.com"                │
│  }                                           │
└──────────────────────────────────────────────┘
```

| Parte | O que é | Exemplo |
|---|---|---|
| **Método** | O que você quer fazer | GET, POST, PUT, DELETE |
| **Caminho** | Onde está o recurso | /api/usuarios |
| **Versão** | Versão do HTTP | HTTP/1.1 |
| **Headers** | Informações extras | Content-Type, Accept |
| **Body** | Dados enviados | JSON, formulário, arquivo |

---

## 6️⃣ Anatomia de uma Resposta HTTP (Response)

```
┌──────────────────────────────────────────────┐
│  LINHA DE STATUS (Status Line)               │
│  HTTP/1.1 200 OK                             │
├──────────────────────────────────────────────┤
│  CABEÇALHOS (Headers)                        │
│  Content-Type: application/json              │
│  Content-Length: 245                          │
│  Date: Wed, 18 Feb 2026 10:30:00 GMT         │
│  Cache-Control: max-age=3600                 │
├──────────────────────────────────────────────┤
│  CORPO (Body)                                │
│  {                                           │
│    "id": 1,                                  │
│    "nome": "Maria",                          │
│    "email": "maria@email.com"                │
│  }                                           │
└──────────────────────────────────────────────┘
```

| Parte | O que é | Exemplo |
|---|---|---|
| **Versão** | Versão do HTTP | HTTP/1.1 |
| **Status Code** | Número indicando resultado | 200, 404, 500 |
| **Status Text** | Descrição do código | OK, Not Found, Error |
| **Headers** | Metadados da resposta | Content-Type, Date |
| **Body** | Os dados retornados | HTML, JSON, imagem |

---

## 7️⃣ HTTP é Stateless (Sem Estado)

O HTTP **não lembra** de requisições anteriores. Cada request é independente.

### 💡 Analogia: Atendimento de balcão

```
  ❌ SEM ESTADO (HTTP puro):
  ────────────────────────────
  Visita 1: "Quero comprar um café"     → "Aqui está!"
  Visita 2: "Quero o mesmo de antes"    → "O quê? Não sei quem é você"

  ✅ COM ESTADO (HTTP + Cookies/Sessões):
  ────────────────────────────
  Visita 1: "Quero um café"             → "Aqui está! Guarde este cartão: #123"
  Visita 2: "Sou o #123, quero outro"   → "Ah sim! Mais um café!"
```

**Como resolver?** Usando **cookies**, **tokens** ou **sessões** (veremos nos tópicos intermediários).

---

## 8️⃣ HTTP vs HTTPS

| Característica | HTTP | HTTPS |
|---|---|---|
| **Significado** | HyperText Transfer Protocol | HTTP **Secure** |
| **Porta padrão** | 80 | 443 |
| **Criptografia** | ❌ Não | ✅ Sim (TLS/SSL) |
| **Segurança** | Dados visíveis a qualquer um na rede | Dados criptografados |
| **Quando usar** | Nunca em produção | ✅ Sempre |
| **Cadeado 🔒** | Não aparece | Aparece no navegador |

```
  HTTP (sem criptografia):
  Cliente ──── "senha: 123456" ────► Servidor
                    👀 Qualquer um pode ler!

  HTTPS (com criptografia):
  Cliente ──── "aB$9x!mK2..." ────► Servidor
                    🔒 Só o servidor entende!
```

---

## 9️⃣ Versões do HTTP

| Versão | Ano | Principais mudanças |
|---|---|---|
| **HTTP/0.9** | 1991 | Apenas GET, sem headers |
| **HTTP/1.0** | 1996 | Headers, status codes, POST |
| **HTTP/1.1** | 1997 | Conexões persistentes, Host header |
| **HTTP/2** | 2015 | Multiplexação, binário, compressão |
| **HTTP/3** | 2022 | QUIC (UDP), mais rápido |

### Qual versão você usa?

Provavelmente **HTTP/2** ou **HTTP/3** — os navegadores modernos usam automaticamente.

```bash
# Verificar qual versão um site usa
curl -sI https://google.com | head -1
# HTTP/2 200

curl --http1.1 -sI https://google.com | head -1
# HTTP/1.1 200 OK
```

---

## 🔟 Resumo Visual

```
  ┌─────────────────────────────────────────────────────────────┐
  │                    RESUMO: COMO HTTP FUNCIONA                │
  └─────────────────────────────────────────────────────────────┘

  1. Você digita URL         ──► https://api.com/dados
  2. DNS resolve o nome      ──► 93.184.216.34
  3. TCP faz o handshake     ──► SYN → SYN-ACK → ACK
  4. TLS criptografa (HTTPS) ──► 🔒 Conexão segura
  5. Cliente envia Request   ──► GET /dados HTTP/1.1
  6. Servidor processa       ──► Busca os dados
  7. Servidor envia Response ──► HTTP/1.1 200 OK + dados
  8. Navegador renderiza     ──► 🎨 Mostra na tela
```

---

## ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| "HTTP e HTTPS são a mesma coisa" | HTTPS adiciona criptografia ao HTTP |
| "O servidor envia dados sozinho" | O cliente sempre inicia a comunicação |
| "A URL é só o domínio" | A URL inclui protocolo, host, path, query, etc. |
| "HTTP lembra de mim entre requests" | HTTP é stateless — cada request é independente |
| "DNS é um servidor só" | DNS é um sistema distribuído mundial |

---

## 🎯 Exercícios

1. Desmembre esta URL em partes: `https://loja.com.br:8080/produtos/busca?q=notebook&marca=dell#resultados`
2. Use `nslookup` para descobrir o IP do `github.com`
3. Faça `curl -v https://httpbin.org/get` e identifique: método, headers, status code e body
4. Qual a diferença entre o body do request e o body do response?

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Configuração Inicial](02-configuracao-inicial.md) | [Próximo: Métodos HTTP ➡️](04-metodos-http.md)
