# ⚙️ Configuração Inicial

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Instalação](01-instalacao.md) | [Próximo: Conceitos Fundamentais ➡️](03-conceitos-fundamentais.md)

---

> Agora que as ferramentas estão instaladas, vamos configurar tudo para você começar a praticar HTTP de verdade.

---

## 📋 Cola Rápida — O que vamos configurar

| Item | Tempo estimado | Dificuldade |
|---|---|---|
| Testar curl com diferentes APIs | 2 min | 🟢 Fácil |
| Configurar Postman/Insomnia | 5 min | 🟢 Fácil |
| Criar um servidor HTTP local | 3 min | 🟢 Fácil |
| Configurar VS Code REST Client | 3 min | 🟢 Fácil |
| Entender as DevTools do navegador | 5 min | 🟢 Fácil |

---

## 1️⃣ Testando o curl com APIs públicas

Vamos fazer algumas requisições para garantir que tudo funciona:

### Requisição GET básica

```bash
# Buscar um post de uma API pública
curl https://jsonplaceholder.typicode.com/posts/1
```

Resultado esperado:
```json
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit..."
}
```

### Ver os detalhes completos da requisição

```bash
# O -v (verbose) mostra TUDO: headers, status, conexão
curl -v https://jsonplaceholder.typicode.com/posts/1
```

### Apenas os headers da resposta

```bash
# O -I mostra só os cabeçalhos (HEAD request)
curl -I https://jsonplaceholder.typicode.com/posts/1
```

### Enviar dados (POST)

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "Meu Post", "body": "Conteúdo", "userId": 1}'
```

### 🎯 APIs públicas para praticar

| API | URL | O que retorna |
|---|---|---|
| JSONPlaceholder | `jsonplaceholder.typicode.com` | Posts, comentários, usuários fake |
| HTTPBin | `httpbin.org` | Ecoa sua requisição de volta |
| PokeAPI | `pokeapi.co/api/v2` | Dados de Pokémon |
| ViaCEP | `viacep.com.br/ws/{cep}/json` | Endereços brasileiros |
| Dog API | `dog.ceo/api/breeds/image/random` | Fotos aleatórias de cachorros 🐕 |

Teste todas:

```bash
# Buscar endereço pelo CEP
curl https://viacep.com.br/ws/01001000/json/

# Buscar dados de um Pokémon
curl https://pokeapi.co/api/v2/pokemon/pikachu

# Foto aleatória de cachorro
curl https://dog.ceo/api/breeds/image/random

# Testar seus headers
curl https://httpbin.org/headers
```

---

## 2️⃣ Configurando o Postman

### Primeiros passos

1. Abra o Postman
2. Você pode usar **sem criar conta** (clique em "Skip" ou "Lightweight API Client")
3. Na barra superior, você verá o campo de URL

### Sua primeira requisição no Postman

1. Selecione o método **GET**
2. Digite a URL: `https://jsonplaceholder.typicode.com/posts/1`
3. Clique em **Send**
4. Veja a resposta na parte inferior da tela

```
┌─────────────────────────────────────────────────────────┐
│  GET  ▼  │ https://jsonplaceholder.typicode.com/posts/1 │  Send  │
├─────────────────────────────────────────────────────────┤
│  Params  │  Headers  │  Body  │  Auth  │                │
├─────────────────────────────────────────────────────────┤
│  Body    │  Headers (9)  │  Status: 200 OK  │  287ms    │
│  ┌──────────────────────────────────────────────┐       │
│  │ {                                            │       │
│  │   "userId": 1,                               │       │
│  │   "id": 1,                                   │       │
│  │   "title": "sunt aut facere..."              │       │
│  │ }                                            │       │
│  └──────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────┘
```

### Organizando com Collections

1. Clique em **"New Collection"**
2. Nomeie como **"Estudos HTTP"**
3. Salve suas requisições dentro dessa collection

---

## 3️⃣ Criando um Servidor HTTP Local

### Com Python (1 linha!)

```bash
# Crie uma pasta para testes
mkdir ~/http-testes && cd ~/http-testes

# Crie um arquivo HTML simples
echo '<h1>Olá, HTTP!</h1><p>Meu primeiro servidor</p>' > index.html

# Inicie o servidor
python3 -m http.server 8080
```

Agora abra no navegador: `http://localhost:8080` 🎉

### Com Node.js

Crie um arquivo `servidor.js`:

```javascript
const http = require('http');

const servidor = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);  // Log da requisição
  
  res.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
  res.end('<h1>Olá, HTTP!</h1><p>Servidor Node.js funcionando!</p>');
});

servidor.listen(3000, () => {
  console.log('🚀 Servidor rodando em http://localhost:3000');
});
```

Execute:
```bash
node servidor.js
```

### Testando seu servidor local

```bash
# Em outro terminal:
curl http://localhost:8080    # Python
curl http://localhost:3000    # Node.js

# Ver headers
curl -v http://localhost:8080
```

---

## 4️⃣ Configurando o VS Code REST Client

### Criando seu arquivo de requisições

1. No VS Code, crie um arquivo chamado `requisicoes.http`
2. Adicione o seguinte conteúdo:

```http
### ===========================
### Estudos HTTP - Requisições
### ===========================

### 1. GET simples
GET https://jsonplaceholder.typicode.com/posts/1 HTTP/1.1

### 2. GET com query params
GET https://jsonplaceholder.typicode.com/posts?userId=1 HTTP/1.1

### 3. POST com JSON
POST https://jsonplaceholder.typicode.com/posts HTTP/1.1
Content-Type: application/json

{
  "title": "Meu Post HTTP",
  "body": "Aprendendo HTTP na prática",
  "userId": 1
}

### 4. PUT (atualizar)
PUT https://jsonplaceholder.typicode.com/posts/1 HTTP/1.1
Content-Type: application/json

{
  "id": 1,
  "title": "Post Atualizado",
  "body": "Conteúdo modificado",
  "userId": 1
}

### 5. DELETE
DELETE https://jsonplaceholder.typicode.com/posts/1 HTTP/1.1

### 6. Buscar CEP brasileiro
GET https://viacep.com.br/ws/01001000/json/ HTTP/1.1
```

3. Clique em **"Send Request"** acima de cada linha `###`

> **💡 Dica:** Cada `###` separa uma requisição diferente no mesmo arquivo.

---

## 5️⃣ Conhecendo as DevTools do Navegador

### Passo a passo

1. Abra o navegador (Chrome, Firefox ou Edge)
2. Pressione `F12` para abrir as DevTools
3. Clique na aba **"Network"** (ou "Rede")
4. Acesse qualquer site

### O que observar

```
┌─────────────────────────────────────────────────────────┐
│  Network                                                │
├─────┬──────────┬────────┬──────┬─────────┬─────────────┤
│ Name│  Status  │  Type  │ Size │  Time   │  Waterfall  │
├─────┼──────────┼────────┼──────┼─────────┼─────────────┤
│ /   │  200     │  doc   │ 45KB │  120ms  │  ████       │
│style│  200     │  css   │ 12KB │   45ms  │   ███       │
│app  │  200     │  js    │ 89KB │  200ms  │    ██████   │
│logo │  200     │  png   │  5KB │   30ms  │     ██      │
│api  │  200     │  xhr   │  2KB │   80ms  │      ████   │
└─────┴──────────┴────────┴──────┴─────────┴─────────────┘
```

### Filtros importantes

| Filtro | O que mostra |
|---|---|
| **All** | Todas as requisições |
| **XHR/Fetch** | Requisições AJAX (APIs) |
| **Doc** | Documentos HTML |
| **CSS** | Folhas de estilo |
| **JS** | Scripts JavaScript |
| **Img** | Imagens |

### Clicando em uma requisição

Ao clicar em qualquer requisição, você verá:

- **Headers** — Cabeçalhos enviados e recebidos
- **Preview** — Pré-visualização da resposta
- **Response** — Resposta completa
- **Timing** — Quanto tempo cada etapa levou

---

## 6️⃣ Configurações Extras Úteis

### Alias para curl (Linux/macOS)

Adicione ao seu `~/.bashrc` ou `~/.zshrc`:

```bash
# Atalhos úteis para HTTP
alias httpget='curl -s -o /dev/null -w "%{http_code}"'
alias httpheaders='curl -s -D - -o /dev/null'
alias httpjson='curl -s -H "Accept: application/json"'
alias httppost='curl -X POST -H "Content-Type: application/json" -d'
```

Uso:
```bash
# Ver apenas o status code
httpget https://google.com
# 200

# Ver apenas os headers
httpheaders https://google.com

# Receber JSON formatado
httpjson https://jsonplaceholder.typicode.com/posts/1 | python3 -m json.tool
```

### Formatando JSON no terminal

```bash
# Com Python (já instalado na maioria dos sistemas)
curl -s https://jsonplaceholder.typicode.com/posts/1 | python3 -m json.tool

# Com jq (precisa instalar)
sudo apt install jq        # Linux
brew install jq             # macOS
curl -s https://jsonplaceholder.typicode.com/posts/1 | jq .
```

---

## ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Começar sem testar se o curl funciona | Sempre verificar `curl --version` primeiro |
| Decorar URLs de APIs | Salvar requisições no Postman ou arquivo `.http` |
| Ignorar as DevTools do navegador | Manter a aba Network aberta enquanto navega |
| Não formatar a saída JSON | Usar `jq` ou `python3 -m json.tool` |
| Testar apenas com um método (GET) | Praticar GET, POST, PUT, DELETE desde o início |

---

## 🎯 Exercícios Práticos

1. Faça um `GET` para `https://viacep.com.br/ws/20040020/json/` e descubra qual endereço é esse
2. Use o Postman para fazer um `POST` em `https://jsonplaceholder.typicode.com/posts` com seus próprios dados
3. Abra a aba Network do navegador e conte quantas requisições são feitas ao abrir o Google
4. Crie um servidor local com Python e acesse pelo curl

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Instalação](01-instalacao.md) | [Próximo: Conceitos Fundamentais ➡️](03-conceitos-fundamentais.md)
