# 📬 Métodos HTTP

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Conceitos Fundamentais](03-conceitos-fundamentais.md) | [Próximo: Códigos de Status ➡️](05-status-codes.md)

---

> Os métodos HTTP (também chamados de "verbos") indicam **o que você quer fazer** com um recurso. Pense neles como as ações que você pode realizar em um sistema.

---

## 📋 Cola Rápida — Todos os Métodos

| Método | Ação | Tem Body? | Seguro? | Idempotente? |
|---|---|---|---|---|
| **GET** | Buscar/ler dados | ❌ Não | ✅ Sim | ✅ Sim |
| **POST** | Criar/enviar dados | ✅ Sim | ❌ Não | ❌ Não |
| **PUT** | Atualizar (substituir tudo) | ✅ Sim | ❌ Não | ✅ Sim |
| **PATCH** | Atualizar (parcialmente) | ✅ Sim | ❌ Não | ❌ Não |
| **DELETE** | Remover/apagar | ❌ Não* | ❌ Não | ✅ Sim |
| **HEAD** | Igual GET, mas só headers | ❌ Não | ✅ Sim | ✅ Sim |
| **OPTIONS** | Verificar o que é permitido | ❌ Não | ✅ Sim | ✅ Sim |

> **Seguro** = não altera dados no servidor | **Idempotente** = repetir várias vezes dá o mesmo resultado

### 💡 Analogia: Sistema de Biblioteca

| Método | Na biblioteca |
|---|---|
| **GET** | Consultar um livro no catálogo |
| **POST** | Cadastrar um livro novo |
| **PUT** | Reescrever toda a ficha do livro |
| **PATCH** | Corrigir só o autor na ficha |
| **DELETE** | Remover um livro do acervo |

---

## 1️⃣ GET — Buscar Dados

O método mais usado da internet. **Toda vez que você abre um site**, é um GET.

### Características
- 🔍 Apenas **lê** dados — não modifica nada
- 📭 **Não tem body** — os parâmetros vão na URL
- 🔄 **Idempotente** — pode repetir sem problema
- 💾 **Pode ser cacheado** pelo navegador

### Exemplos com curl

```bash
# GET simples — buscar um post
curl https://jsonplaceholder.typicode.com/posts/1

# GET com parâmetros na URL (query string)
curl "https://jsonplaceholder.typicode.com/posts?userId=1"

# GET mostrando os headers
curl -v https://jsonplaceholder.typicode.com/posts/1

# GET retornando apenas o status code
curl -o /dev/null -s -w "%{http_code}" https://jsonplaceholder.typicode.com/posts/1
```

### Exemplo com JavaScript

```javascript
// Buscar todos os posts
const resposta = await fetch('https://jsonplaceholder.typicode.com/posts');
const posts = await resposta.json();
console.log(posts);

// Buscar com filtro
const resposta2 = await fetch('https://jsonplaceholder.typicode.com/posts?userId=1');
const postsDoUsuario = await resposta2.json();
```

### Exemplo com Python

```python
import requests

# Buscar um post
resposta = requests.get('https://jsonplaceholder.typicode.com/posts/1')
print(resposta.json())

# Buscar com parâmetros
resposta = requests.get(
    'https://jsonplaceholder.typicode.com/posts',
    params={'userId': 1}
)
print(resposta.json())
```

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| `GET /deletar-usuario?id=5` | `DELETE /usuarios/5` |
| `GET /criar-post?titulo=Oi` | `POST /posts` com body |
| Enviar dados sensíveis na URL via GET | Usar POST com body para dados sensíveis |
| `GET /usuarios` e modificar algo | GET é só para **leitura** |

---

## 2️⃣ POST — Criar/Enviar Dados

Usado para **enviar dados ao servidor**, geralmente para criar algo novo.

### Características
- ✏️ **Cria** novos recursos
- 📦 **Tem body** — os dados vão no corpo da requisição
- ❌ **Não idempotente** — enviar 2 vezes pode criar 2 registros
- 🚫 **Não é cacheado** por padrão

### Exemplos com curl

```bash
# POST simples com JSON
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Meu Primeiro Post",
    "body": "Aprendendo HTTP!",
    "userId": 1
  }'

# POST com form data
curl -X POST https://httpbin.org/post \
  -d "nome=Maria&email=maria@email.com"

# POST com arquivo
curl -X POST https://httpbin.org/post \
  -F "arquivo=@foto.jpg"
```

### Exemplo com JavaScript

```javascript
const resposta = await fetch('https://jsonplaceholder.typicode.com/posts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    title: 'Meu Post',
    body: 'Conteúdo do post',
    userId: 1,
  }),
});

const novoPost = await resposta.json();
console.log(novoPost); // { id: 101, title: 'Meu Post', ... }
```

### Exemplo com Python

```python
import requests

novo_post = {
    'title': 'Meu Post',
    'body': 'Conteúdo do post',
    'userId': 1
}

resposta = requests.post(
    'https://jsonplaceholder.typicode.com/posts',
    json=novo_post
)

print(resposta.status_code)  # 201 Created
print(resposta.json())       # { id: 101, ... }
```

### ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| `POST /posts/1` (atualizar) | `PUT /posts/1` ou `PATCH /posts/1` |
| `POST /buscar-usuarios` | `GET /usuarios?filtro=valor` |
| Não enviar `Content-Type` | Sempre incluir o header `Content-Type` |
| Enviar JSON sem `application/json` | `Content-Type: application/json` |

---

## 3️⃣ PUT — Atualizar (Substituir Tudo)

O PUT **substitui completamente** o recurso no servidor. Se você omitir um campo, ele some.

### Características
- 🔄 **Substitui** o recurso inteiro
- 📦 **Tem body** — envia o recurso completo
- ✅ **Idempotente** — enviar 5 vezes dá o mesmo resultado
- ⚠️ Campos não enviados são **removidos**

### Exemplo prático

```bash
# O recurso original:
# { "id": 1, "title": "Post Original", "body": "Conteúdo", "userId": 1 }

# PUT substitui TUDO
curl -X PUT https://jsonplaceholder.typicode.com/posts/1 \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "title": "Post Atualizado",
    "body": "Novo conteúdo completo",
    "userId": 1
  }'
```

### ⚠️ Cuidado com PUT!

```
  ANTES do PUT:
  { "id": 1, "nome": "Maria", "email": "maria@email.com", "idade": 25 }

  PUT com body:
  { "id": 1, "nome": "Maria Silva" }

  DEPOIS do PUT:
  { "id": 1, "nome": "Maria Silva" }
  ❌ email e idade SUMIRAM! PUT substituiu tudo!
```

### JavaScript

```javascript
const resposta = await fetch('https://jsonplaceholder.typicode.com/posts/1', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    id: 1,
    title: 'Título Atualizado',
    body: 'Conteúdo completo atualizado',
    userId: 1,
  }),
});
```

---

## 4️⃣ PATCH — Atualizar Parcialmente

O PATCH atualiza **apenas os campos** que você enviar, sem mexer no resto.

### Diferença entre PUT e PATCH

```
  ┌──────────────────────────────────────────────────┐
  │  Recurso original:                                │
  │  { "id": 1, "nome": "Maria", "idade": 25,        │
  │    "email": "maria@email.com" }                   │
  ├──────────────────────────────────────────────────┤
  │  PUT { "nome": "Maria Silva" }                    │
  │  Resultado: { "nome": "Maria Silva" }             │
  │  ❌ Os outros campos somem!                       │
  ├──────────────────────────────────────────────────┤
  │  PATCH { "nome": "Maria Silva" }                  │
  │  Resultado: { "id": 1, "nome": "Maria Silva",    │
  │    "idade": 25, "email": "maria@email.com" }      │
  │  ✅ Só o nome mudou, o resto continua!            │
  └──────────────────────────────────────────────────┘
```

### Exemplos

```bash
# PATCH — atualizar apenas o título
curl -X PATCH https://jsonplaceholder.typicode.com/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Só o título mudou"}'
```

```python
import requests

# Atualizar apenas o email
resposta = requests.patch(
    'https://jsonplaceholder.typicode.com/posts/1',
    json={'title': 'Novo Título'}
)
print(resposta.json())  # Todos os campos mantidos, só title mudou
```

---

## 5️⃣ DELETE — Remover

Remove um recurso do servidor.

### Características
- 🗑️ **Remove** o recurso
- ✅ **Idempotente** — deletar algo que já não existe não causa erro (idealmente retorna 404)
- 📭 Geralmente **não tem body**

### Exemplos

```bash
# Deletar um post
curl -X DELETE https://jsonplaceholder.typicode.com/posts/1

# Verificar o status code
curl -X DELETE -o /dev/null -s -w "%{http_code}" \
  https://jsonplaceholder.typicode.com/posts/1
# 200
```

```javascript
const resposta = await fetch('https://jsonplaceholder.typicode.com/posts/1', {
  method: 'DELETE',
});
console.log(resposta.status); // 200
```

```python
import requests

resposta = requests.delete('https://jsonplaceholder.typicode.com/posts/1')
print(resposta.status_code)  # 200
```

---

## 6️⃣ HEAD — Só os Cabeçalhos

Igual ao GET, mas **retorna só os headers** (sem o body). Útil para verificar se um recurso existe ou checar metadados sem baixar o conteúdo.

```bash
# HEAD — só retorna os headers
curl -I https://jsonplaceholder.typicode.com/posts/1
```

Resultado:
```
HTTP/2 200
content-type: application/json; charset=utf-8
content-length: 292
```

### Quando usar?
- Verificar se um arquivo existe antes de baixar
- Checar o tamanho de um arquivo (`Content-Length`)
- Verificar se um recurso foi modificado (`Last-Modified`)

---

## 7️⃣ OPTIONS — O que é Permitido?

Pergunta ao servidor **quais métodos são aceitos** para um determinado recurso.

```bash
curl -X OPTIONS https://jsonplaceholder.typicode.com/posts -v
```

O servidor responde com o header `Allow`:
```
Allow: GET, POST, PUT, PATCH, DELETE, OPTIONS
```

### Quando é usado automaticamente?

O navegador envia OPTIONS automaticamente em **requisições cross-origin (CORS)** como um "pedido de permissão" antes da requisição real. Isso é chamado de **preflight request**.

---

## 📊 Comparativo Visual

```
  ┌─────────┬──────────────────┬───────────────────────────────┐
  │ Método  │     Ação         │         Exemplo               │
  ├─────────┼──────────────────┼───────────────────────────────┤
  │  GET    │ 📖 Ler           │ GET /usuarios/1               │
  │  POST   │ ➕ Criar          │ POST /usuarios    + body      │
  │  PUT    │ 🔄 Substituir    │ PUT /usuarios/1   + body      │
  │  PATCH  │ ✏️ Editar campo   │ PATCH /usuarios/1 + body      │
  │  DELETE │ 🗑️ Remover        │ DELETE /usuarios/1            │
  │  HEAD   │ 🔍 Checar        │ HEAD /usuarios/1              │
  │ OPTIONS │ ❓ Perguntar      │ OPTIONS /usuarios             │
  └─────────┴──────────────────┴───────────────────────────────┘
```

---

## 🏪 CRUD e Métodos HTTP

Na prática, usamos os métodos HTTP para operações **CRUD** (Create, Read, Update, Delete):

| CRUD | Método HTTP | Endpoint típico | Descrição |
|---|---|---|---|
| **C**reate | POST | `POST /usuarios` | Criar novo usuário |
| **R**ead | GET | `GET /usuarios` | Listar todos |
| **R**ead | GET | `GET /usuarios/1` | Buscar um específico |
| **U**pdate | PUT/PATCH | `PUT /usuarios/1` | Atualizar usuário |
| **D**elete | DELETE | `DELETE /usuarios/1` | Remover usuário |

### Exemplo completo (CRUD de Usuários)

```bash
# CREATE — Criar usuário
curl -X POST https://jsonplaceholder.typicode.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Ana", "email": "ana@email.com"}'

# READ — Listar todos os usuários
curl https://jsonplaceholder.typicode.com/users

# READ — Buscar usuário específico
curl https://jsonplaceholder.typicode.com/users/1

# UPDATE — Atualizar nome do usuário
curl -X PATCH https://jsonplaceholder.typicode.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Ana Silva"}'

# DELETE — Remover usuário
curl -X DELETE https://jsonplaceholder.typicode.com/users/1
```

---

## 🎯 Exercícios Práticos

1. Faça um `GET` para `https://jsonplaceholder.typicode.com/users` e conte quantos usuários existem
2. Crie um novo post com `POST` incluindo título e corpo personalizados
3. Atualize apenas o título de um post usando `PATCH`
4. Delete o post de id 5 e verifique o status code retornado
5. Use `HEAD` para descobrir o `Content-Type` sem baixar o body

---

[⬅️ Voltar ao Índice](../README.md) | [⬅️ Anterior: Conceitos Fundamentais](03-conceitos-fundamentais.md) | [Próximo: Códigos de Status ➡️](05-status-codes.md)
