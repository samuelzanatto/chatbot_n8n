# Configuração N8n + Evolution API para Chatbot WhatsApp

Este guia apresenta um passo a passo completo para configurar um chatbot WhatsApp usando N8n e Evolution API.

## ⚙️ Requisitos

Antes de começar, certifique-se de ter instalado:
- **Node.js** (versão mais recente)
- **Docker Desktop**

---

## 🚀 Configurando a Evolution API

### 1. Download do Projeto
Execute o comando para baixar o projeto:
```bash
docker pull https://github.com/samuelzanatto/chatbot_n8n.git
```

### 2. Configuração dos Arquivos

Abra o projeto no VSCode e faça as seguintes alterações:

#### Arquivo `docker-compose.yaml`
Na seção `postgres`, altere:
- `POSTGRES_USER`: seu usuário do banco de dados
- `POSTGRES_PASSWORD`: sua senha do banco de dados

#### Arquivo `.env`
- Altere `DATABASE_CONNECTION_URI` com os dados de conexão PostgreSQL (usuário e senha definidos no passo anterior)
- **Recomendação de Segurança**: Altere a chave padrão `AUTHENTICATION_API_KEY` para uma chave personalizada

### 3. Inicialização dos Containers
No terminal do projeto, execute:
```bash
docker-compose up -d
```

### 4. Verificação dos Containers
Acesse o Docker Desktop e verifique se os containers estão rodando corretamente.

### 5. Acesso à Interface
Acesse: `http://localhost:8080/manager`

**Dados de Login:**
- **Server URL**: `http://localhost:8080`
- **API Key Global**: (Chave definida na variável `AUTHENTICATION_API_KEY` no arquivo `.env`)

### 6. Criação de Instância
1. Clique em **"Instance +"** no canto superior direito
2. Digite o nome da sua instância
3. Mantenha os demais campos como estão
4. Clique em **"Save"**

### 7. Conectar WhatsApp
1. Clique no card da instância criada
2. Clique em **"Get QR Code"**
3. Escaneie o QR Code com o WhatsApp que será usado como chatbot
4. Verifique se foi conectado corretamente (nome do usuário, número, contatos, chats e mensagens devem aparecer)

---

## 🔧 Configurando o N8n

### 1. Instalação via Docker
1. Abra o Docker Desktop
2. Na barra de pesquisa, digite **"n8n"**
3. Faça o pull do repositório **"n8nio/n8n"**

### 2. Configuração do Container
Clique em **"Run"** e configure os Optional Settings:

- **Container name**: `n8n`
- **Host port**: `5678`
- **Host path**: Crie uma pasta "n8n" na raiz da máquina e digite o caminho
- **Container path**: `/home/node/.n8n`
- **Variable**: `GENERIC_TIMEZONE`
- **Value**: `America/Sao_Paulo`

### 3. Inicialização
1. Clique em **"Run"**
2. Acesse: `http://localhost:5678/`
3. Cadastre-se na plataforma quando solicitado

---

## 🤖 Configurando o Chatbot

### 1. Configuração da API Groq
1. Acesse: `https://groq.com/`
2. Clique em **"DEV CONSOLE"**
3. Faça login na plataforma
4. Acesse a seção **"API Keys"**
5. Clique em **"Create API Key"**
6. Digite um nome para a chave
7. Clique em **"Submit"**
8. **Importante**: Copie e salve a chave em um local seguro (não será possível visualizá-la novamente)
9. Clique em **"Done"**

### 2. Instalação do Node Evolution API no N8n
1. No N8n, clique nos 3 pontos no canto inferior esquerdo
2. Clique em **"Settings"**
3. Acesse **"Community Nodes"** → **"Install"**
4. Em "npm Package Name", digite: `n8n-nodes-evolution-api`
5. Marque a caixa de seleção e clique em **"Install"**

### 3. Criação do Workflow

#### 3.1 Configuração Inicial
1. Na tela inicial do N8n, clique em **"Create Workflow"**
2. Renomeie seu workflow

#### 3.2 Webhook
1. Clique em **"Add first step..."**
2. Digite "webhook" e selecione **"Webhook"**
3. Altere o HTTP Method para **POST**
4. Copie a **Test URL**

#### 3.3 Configuração do Webhook na Evolution API
1. Acesse a Evolution API
2. Abra a instância criada
3. Na sidebar esquerda: **Events** → **Webhook**
4. Cole a Test URL no campo URL
5. **Importante**: Altere `localhost` para `host.docker.internal`
6. Ative as opções:
   - **Webhook Base64**
   - **MESSAGES_UPSERT**
7. Clique em **"Save"**

#### 3.4 Teste do Webhook
1. No N8n, nas configurações do webhook, clique em **"Listen for test event"**
2. Envie uma mensagem para você mesmo no WhatsApp (mesmo WhatsApp conectado na Evolution API)
3. Verifique se aparece um output na tela do webhook
4. Feche a tela do webhook

#### 3.5 Nó Edit Fields (Dados)
1. Adicione um novo nó clicando no **+** ao lado do webhook
2. Pesquise "set" e adicione **"Edit Fields (Set)"**
3. Certifique-se de que os dados foram carregados (se não, clique em **"Execute previous nodes"**)
4. Mantenha Mode em **"Manual Mapping"**
5. Adicione os campos clicando em **"Add Field"**:

| Name | Value |
|------|-------|
| Quem mandou | remoteJid |
| Instância | instance |
| Mensagem | conversation |
| Id da mensagem | id |
| Nome da pessoa | pushName |

6. Clique em **"Test step"** para verificar
7. Renomeie o nó para **"Dados"**

#### 3.6 Nó Set Auxiliar
1. Adicione outro nó **"Set"** ao lado do "Edit Fields"
2. Deixe este nó vazio (não altere nada)

#### 3.7 AI Agent
1. Clique em **+** ao lado do nó "Set" vazio
2. Pesquise "ai agent" e adicione **"AI Agent"**
3. Em "Source for Prompt (User Message)", selecione **"Define below"**
4. Clique em **"Execute previous nodes"**
5. Arraste o campo **"Mensagem"** para o input **"Text"** do AI Agent
6. Em "Options", clique em **"Add Option"** → **"System Message"**
7. Digite a personalidade e regras do seu agente de IA

#### 3.8 Groq Chat Model
1. Clique no primeiro **"+"** abaixo do AI Agent (Chat Model)
2. Selecione **"Groq Chat Model"**
3. Em "Credential to connect with", selecione **"Create new credential"**
4. Cole a chave de API do Groq
5. Clique em **"Save"**
6. Selecione o modelo de IA desejado no campo "Model"

#### 3.9 Redis Chat Memory
1. Clique no segundo **"+"** (Memory)
2. Selecione **"Redis Chat Memory"**
3. Em "Credential to connect with", selecione **"Create new credential"**
4. Altere apenas o **Host** para: `host.docker.internal`
5. Clique em **"Save"**
6. Configure os campos:
   - **Session ID**: "Define below"
   - **Key**: Arraste "Quem mandou" dos dados da mensagem
   - **Session Time To Live**: `3600`
   - **Context Window Length**: `10`

#### 3.10 Evolution API - Ler Mensagens
1. Adicione um nó **"Evolution API"** ao lado do AI Agent
2. Em "Credential to connect with", selecione **"Create new credential"**
3. Configure:
   - **Server Url**: `http://host.docker.internal:8080`
   - **ApiKey**: Chave de API da Evolution API (mesma do arquivo .env)
4. Clique em **"Save"**
5. Configure os campos:
   - **Recurso**: "Chat"
   - **Operação**: "Ler Mensagens"
   - **Nome da Instância**: Arraste "Instancia"
   - **Contato**: Arraste "Quem mandou"
   - **Id da mensagem**: Arraste "Id da mensagem"
6. Clique em **"Test step"** (deve retornar sucesso)

#### 3.11 Evolution API - Enviar Mensagem
1. Adicione outro nó **"Evolution API"**
2. Configure:
   - **Recurso**: "Mensagem"
   - **Operação**: "Enviar Texto"
   - **Nome da Instancia**: Mesmo dado anterior
   - **Número do Destinatário**: Mesmo dado anterior
   - **Mensagem**: Arraste o output do "AI Agent"
3. Em "Adicionar Campo" (Opções):
   - Adicione **"Delay"**: `3000`
   - Adicione **"Preview de Link"**

### 4. Teste e Ativação

#### Teste do Workflow
1. Clique em **"Test workflow"** no centro inferior
2. Envie uma mensagem para o número cadastrado
3. O chatbot deve responder

#### Ativação para Produção
1. Substitua a Test URL do webhook pela URL de produção (encontrada no nó webhook do N8n)
2. Atualize a URL na Evolution API
3. Salve as configurações da Evolution API
4. No N8n, salve e ative o fluxo (switch no canto superior direito deve ficar verde)
5. Para monitorar execuções, clique em **"Executions"**

---

## ✅ Verificação Final

Após completar todos os passos:
- O chatbot deve responder automaticamente às mensagens
- As conversas ficam registradas no Redis
- O fluxo aparece ativo no N8n
- As execuções podem ser monitoradas na aba "Executions"

## 🔍 Solução de Problemas

- Verifique se todos os containers estão rodando no Docker Desktop
- Confirme se as URLs estão corretas (use `host.docker.internal` para comunicação entre containers)
- Verifique se as chaves de API estão corretas
- Teste cada nó individualmente usando "Test step"

---

**Pronto!** Seu chatbot WhatsApp com N8n e Evolution API está configurado e funcionando.