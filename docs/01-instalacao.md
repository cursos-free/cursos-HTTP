# 🛠️ Instalação de Ferramentas

[⬅️ Voltar ao Índice](../README.md) | [Próximo: Configuração Inicial ➡️](02-configuracao-inicial.md)

---

> Para estudar e praticar HTTP, você vai precisar de algumas ferramentas. Aqui vamos instalar tudo que você precisa — **não importa qual sistema operacional você usa**.

---

## 📋 Cola Rápida — Ferramentas Essenciais

| Ferramenta | Para quê serve | Obrigatória? |
|---|---|---|
| **curl** | Fazer requisições HTTP pelo terminal | ✅ Sim |
| **Navegador moderno** | Inspecionar tráfego HTTP (DevTools) | ✅ Sim |
| **Postman** ou **Insomnia** | Interface gráfica para testar APIs | 🟡 Recomendado |
| **VS Code** | Editor de código com extensões úteis | 🟡 Recomendado |
| **Node.js** | Executar JavaScript e criar servidores | 🟡 Opcional |
| **Python** | Criar servidores simples e scripts | 🟡 Opcional |

---

## 1️⃣ Instalando o curl

O **curl** é uma ferramenta de linha de comando para fazer requisições HTTP. É a ferramenta mais usada do mundo para esse fim.

### 🪟 Windows

O curl já vem **instalado no Windows 10/11**. Para verificar:

```bash
curl --version
```

Se não estiver instalado:

1. Baixe em [curl.se/windows](https://curl.se/windows/)
2. Extraia o arquivo
3. Adicione a pasta ao PATH do sistema

Ou instale via **winget**:
```bash
winget install cURL.cURL
```

### 🍎 macOS

O curl já vem **instalado por padrão**. Para verificar:

```bash
curl --version
```

Para atualizar para a versão mais recente:
```bash
brew install curl
```

### 🐧 Linux

A maioria das distribuições já inclui o curl. Para instalar ou atualizar:

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install curl -y

# Fedora
sudo dnf install curl

# Arch Linux
sudo pacman -S curl
```

### ✅ Verificando a instalação

```bash
curl --version
# Deve mostrar algo como: curl 8.x.x (x86_64-...)
```

Teste rápido:
```bash
curl https://httpbin.org/get
```

Se retornou um JSON, está funcionando! 🎉

---

## 2️⃣ Navegador com DevTools

Qualquer navegador moderno serve. Recomendações:

| Navegador | DevTools | Download |
|---|---|---|
| **Google Chrome** | Excelentes | [chrome.google.com](https://www.google.com/chrome/) |
| **Firefox** | Excelentes | [firefox.com](https://www.mozilla.org/firefox/) |
| **Microsoft Edge** | Muito boas | Já vem no Windows 10/11 |

### Como abrir o DevTools

| Atalho | Sistema |
|---|---|
| `F12` | Todos |
| `Ctrl + Shift + I` | Windows / Linux |
| `Cmd + Option + I` | macOS |

A aba **"Network" (Rede)** é onde você vai ver todas as requisições HTTP.

---

## 3️⃣ Instalando o Postman

O Postman é uma ferramenta visual para construir, testar e documentar requisições HTTP.

### Todas as plataformas

1. Acesse [postman.com/downloads](https://www.postman.com/downloads/)
2. Baixe a versão para seu sistema
3. Instale normalmente

### 🪟 Windows (via winget)
```bash
winget install Postman.Postman
```

### 🍎 macOS (via Homebrew)
```bash
brew install --cask postman
```

### 🐧 Linux (via Snap)
```bash
sudo snap install postman
```

### 💡 Alternativa: Insomnia

Se preferir algo mais leve:

```bash
# macOS
brew install --cask insomnia

# Linux (Snap)
sudo snap install insomnia

# Windows
winget install Insomnia.Insomnia
```

---

## 4️⃣ Instalando o VS Code (Opcional)

O VS Code é um editor de código gratuito com extensões muito úteis para HTTP.

### Todas as plataformas

1. Acesse [code.visualstudio.com](https://code.visualstudio.com/)
2. Baixe e instale

### Extensões recomendadas para HTTP

| Extensão | Para quê |
|---|---|
| **REST Client** | Fazer requisições HTTP direto do editor |
| **Thunder Client** | Postman dentro do VS Code |
| **HTTP/s and relative link checker** | Verificar links |

Para instalar extensões:
1. Abra o VS Code
2. Pressione `Ctrl + Shift + X` (ou `Cmd + Shift + X` no Mac)
3. Pesquise pelo nome da extensão
4. Clique em **Install**

### Usando a extensão REST Client

Crie um arquivo `teste.http` e escreva:

```http
### Minha primeira requisição
GET https://jsonplaceholder.typicode.com/posts/1 HTTP/1.1
```

Clique em **"Send Request"** que aparece acima da linha. Pronto! 🚀

---

## 5️⃣ Instalando Node.js (Opcional)

Útil para criar servidores HTTP locais e testar APIs.

### 🪟 Windows
```bash
winget install OpenJS.NodeJS.LTS
```

### 🍎 macOS
```bash
brew install node
```

### 🐧 Linux
```bash
# Ubuntu / Debian
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Ou via nvm (recomendado)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
```

### Verificação
```bash
node --version    # v20.x.x ou superior
npm --version     # 10.x.x ou superior
```

---

## 6️⃣ Instalando Python (Opcional)

Python é ótimo para criar servidores HTTP rápidos e fazer scripts de teste.

### 🪟 Windows
```bash
winget install Python.Python.3.12
```

### 🍎 macOS
```bash
brew install python
```

### 🐧 Linux
```bash
# Geralmente já vem instalado
python3 --version

# Se não tiver:
sudo apt install python3 python3-pip -y
```

### Instale a biblioteca requests
```bash
pip install requests
# ou
pip3 install requests
```

---

## ✅ Checklist Final

Verifique se tudo está instalado:

```bash
# Deve funcionar:
curl --version
node --version       # se instalou
python3 --version    # se instalou
```

- [ ] curl instalado e funcionando
- [ ] Navegador com DevTools acessível (F12)
- [ ] Postman ou Insomnia instalado
- [ ] VS Code com REST Client (opcional)
- [ ] Node.js instalado (opcional)
- [ ] Python com requests (opcional)

---

## ❌ Errado vs ✅ Certo

| ❌ Errado | ✅ Certo |
|---|---|
| Instalar tudo de uma vez sem testar | Instalar uma ferramenta, testar, depois ir para a próxima |
| Usar versões muito antigas | Sempre usar a versão LTS ou estável mais recente |
| Não verificar se curl funciona | Rodar `curl --version` logo após instalar |
| Ignorar as DevTools do navegador | Aprender a usar a aba Network — é sua melhor amiga |

---

[⬅️ Voltar ao Índice](../README.md) | [Próximo: Configuração Inicial ➡️](02-configuracao-inicial.md)
