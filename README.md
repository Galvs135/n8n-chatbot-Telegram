# 🌤️ Telegram Weather Chatbot - N8N Workflow

Um chatbot para Telegram que consulta a temperatura de cidades brasileiras usando a API gratuita do OpenWeather.

## 📋 Descrição

Este workflow do N8N recebe mensagens de texto via Telegram contendo o nome de uma cidade brasileira no formato `Cidade,UF`, consulta a API do OpenWeather e retorna uma mensagem amigável com a temperatura atual.

> **💡 Nota:** O sufixo `,BR` é adicionado automaticamente pelo workflow, então você pode enviar apenas `Cidade,UF`.

### Exemplo de uso:
- **Usuário envia:** `Belo Horizonte,MG`
- **Bot responde:** `🌤️ A temperatura em Belo Horizonte é de 25°C.`

### Em caso de erro:
- **Bot responde:** `❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).`

---

## 🔧 Pré-requisitos

Antes de importar o workflow, você precisará:

1. **N8N instalado** (self-hosted ou n8n.cloud)
2. **Bot do Telegram** criado via [@BotFather](https://t.me/BotFather)
3. **Conta no OpenWeather** com API Key gratuita

---

## 📦 Variáveis de Ambiente Necessárias

Configure as seguintes variáveis de ambiente no seu N8N:

| Variável | Descrição | Onde obter |
|----------|-----------|------------|
| `OPENWEATHER_API_KEY` | Chave da API do OpenWeather | [openweathermap.org/api](https://openweathermap.org/api) |
| `TELEGRAM_BOT_TOKEN` | Token do seu bot Telegram | [@BotFather](https://t.me/BotFather) no Telegram |

### Como configurar variáveis de ambiente no N8N:

#### Opção 1: Via arquivo `.env` (self-hosted)
```env
OPENWEATHER_API_KEY=sua_api_key_aqui
TELEGRAM_BOT_TOKEN=seu_token_aqui
```

#### Opção 2: Via interface do N8N
1. Acesse **Settings** > **Variables**
2. Adicione cada variável com seu respectivo valor

---

## 🚀 Instruções de Importação

### Passo 1: Importar o Workflow

1. Abra o N8N no seu navegador
2. Clique em **"Add Workflow"** ou **"Novo Workflow"**
3. Clique nos três pontos (**⋮**) no canto superior direito
4. Selecione **"Import from File"**
5. Selecione o arquivo `workflow-telegram-chatbot.json`
6. O workflow será carregado com todos os nós configurados

### Passo 2: Configurar Credenciais do Telegram

1. Clique no nó **"Telegram Trigger"**
2. Em **"Credential to connect with"**, clique em **"Create New Credential"**
3. Insira:
   - **Access Token:** Cole o token do seu bot obtido do BotFather
4. Clique em **"Save"**
5. Repita o processo para os nós **"Envia Resposta Sucesso"** e **"Envia Resposta Erro"** (ou selecione a credencial já criada)

### Passo 3: Configurar a API Key do OpenWeather

A API Key é lida automaticamente da variável de ambiente `OPENWEATHER_API_KEY`. Certifique-se de que ela está configurada conforme a seção "Variáveis de Ambiente Necessárias".

### Passo 4: Ativar o Workflow

1. Alterne o switch no canto superior direito para **"Active"**
2. O webhook do Telegram será registrado automaticamente
3. O bot está pronto para receber mensagens!

---

## 🏗️ Estrutura do Workflow

```
┌─────────────────────┐
│  Telegram Trigger   │  ← Recebe mensagens do Telegram
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────┐
│  Captura e Formata Entrada  │  ← Normaliza o texto (minúsculas, trim),
│  (Set Node - queue)         │     adiciona ,BR se necessário e salva chat ID
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│    Consulta OpenWeather     │  ← HTTP Request para a API
│    (HTTP Request Node)      │     com parâmetros: q, units, lang, appid
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│      Valida Resposta        │  ← Verifica se cod=200 e temp existe
│      (IF Node)              │
└──────┬──────────────┬───────┘
       │ TRUE         │ FALSE
       ▼              ▼
┌──────────────┐  ┌──────────────┐
│ Formata      │  │ Formata      │
│ Sucesso      │  │ Erro         │
└──────┬───────┘  └──────┬───────┘
       │                 │
       ▼                 ▼
┌──────────────┐  ┌──────────────┐
│ Envia        │  │ Envia        │
│ Resposta     │  │ Resposta     │
│ Sucesso      │  │ Erro         │
└──────────────┘  └──────────────┘
```

---

## 📡 Parâmetros da API OpenWeather

O nó HTTP Request está configurado com os seguintes parâmetros:

| Parâmetro | Valor | Descrição |
|-----------|-------|-----------|
| `q` | `{{ $json.queue }}` | Cidade formatada (ex: são paulo,sp,br) |
| `units` | `metric` | Temperatura em Celsius |
| `lang` | `pt_br` | Descrições em português brasileiro |
| `appid` | `{{ $env.OPENWEATHER_API_KEY }}` | API Key via variável de ambiente |

---

## 🧪 Testando o Bot

1. Abra o Telegram e encontre seu bot pelo nome
2. Inicie uma conversa com `/start`
3. Envie uma mensagem com uma cidade brasileira:
   - `São Paulo,SP`
   - `Rio de Janeiro,RJ`
   - `Belo Horizonte,MG`
   - `Brasília,DF`
   - `Curitiba,PR`

> **💡 Dica:** O formato `Cidade,UF,BR` também funciona (ex: `São Paulo,SP,BR`), mas o `,BR` é opcional.

### Exemplos de respostas esperadas:

✅ **Sucesso:**
```
🌤️ A temperatura em São Paulo é de 25°C.
```

❌ **Erro (cidade inválida):**
```
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).
```

---

## ⚠️ Observações Importantes

1. **Formato da cidade:** Use o formato `Cidade,UF` (ex: `Belo Horizonte,MG`). O `,BR` é adicionado automaticamente.
2. **Acentuação:** A API do OpenWeather aceita caracteres acentuados
3. **Limite de requisições:** A API gratuita permite até 60 requisições por minuto
4. **Credenciais:** O arquivo JSON exportado **não contém** tokens ou API keys - eles devem ser configurados manualmente

---

## 🔍 Solução de Problemas

### Bot não responde
- Verifique se o workflow está **ativo** (switch verde)
- Confirme que as credenciais do Telegram estão corretas
- Verifique os logs de execução no N8N

### Erro "API key invalid"
- Confirme que a variável `OPENWEATHER_API_KEY` está configurada
- Verifique se a API key é válida em [openweathermap.org](https://openweathermap.org)

### Cidade não encontrada
- Use o formato correto: `Cidade,UF` (ex: `São Paulo,SP`)
- Verifique a grafia da cidade
- Algumas cidades menores podem não estar disponíveis na API

---

## 📄 Licença

Este projeto é disponibilizado para fins educacionais.

---

## 🤝 Contribuições

Sugestões e melhorias são bem-vindas! Abra uma issue ou pull request.
