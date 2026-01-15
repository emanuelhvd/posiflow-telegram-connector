# Posiflow Telegram Connector

Conector customizado do Telegram para a plataforma Tiledesk, desenvolvido especialmente para o Posiflow com suporte a **Reply Keyboard** (botões de resposta rápida).

## Sobre o Projeto

Este projeto é um fork customizado do `@tiledesk/tiledesk-telegram-connector` que integra o Telegram com a plataforma Tiledesk, permitindo que conversas do Telegram sejam gerenciadas através do sistema de atendimento Tiledesk/Posiflow.

### Principais Características

- **Integração bidirecional**: Mensagens fluem do Telegram para o Tiledesk e vice-versa
- **Reply Keyboard**: Botões de resposta rápida que aparecem no histórico do chat (customização Posiflow)
- **Suporte a múltiplos tipos de mídia**: Imagens, vídeos, documentos e textos
- **Configuração via interface web**: Interface administrativa para configurar tokens e webhooks
- **Persistência de configurações**: Armazenamento em MongoDB
- **Gerenciamento de departamentos**: Direcionamento de conversas para departamentos específicos

## Tecnologias Utilizadas

- **Node.js** 16.20.1
- **Express.js** - Framework web
- **MongoDB** - Banco de dados para configurações
- **Axios** - Cliente HTTP
- **Winston** - Sistema de logs
- **Handlebars** - Templates HTML

## Estrutura do Projeto

```
posiflow-telegram-connector/
├── index.js                              # Ponto de entrada da aplicação
├── telegramRoute/
│   ├── index.js                         # Rotas e lógica principal
│   ├── tiledesk/
│   │   ├── TiledeskTelegramTranslator.js   # Tradutor de mensagens Telegram ↔ Tiledesk
│   │   ├── TiledeskTelegram.js             # Cliente da API do Telegram
│   │   ├── TiledeskChannel.js              # Cliente de canais Tiledesk
│   │   ├── TiledeskSubscriptionClient.js   # Gerenciamento de assinaturas
│   │   ├── TiledeskAppsClient.js           # Cliente de apps Tiledesk
│   │   ├── MessageHandler.js               # Processador de mensagens
│   │   └── KVBaseMongo.js                  # Gerenciador de armazenamento MongoDB
│   ├── models/
│   │   └── Setting.js                       # Model de configurações
│   └── winston.js                           # Configuração de logs
└── package.json
```

## Como Funciona

### 1. Fluxo Telegram → Tiledesk

1. Usuário envia mensagem no Telegram
2. Telegram envia webhook para `/telegram?project_id=xxx`
3. `TiledeskTelegramTranslator` converte mensagem do formato Telegram para Tiledesk
4. `TiledeskChannel` envia mensagem para a API do Tiledesk
5. Atendente visualiza mensagem no painel Tiledesk/Posiflow

### 2. Fluxo Tiledesk → Telegram

1. Atendente envia mensagem no painel Tiledesk
2. Webhook recebido em `/tiledesk`
3. `TiledeskTelegramTranslator` converte mensagem do formato Tiledesk para Telegram
4. `TiledeskTelegram` envia mensagem via API do Telegram
5. Usuário recebe mensagem no Telegram

### 3. Reply Keyboard (Customização Posiflow)

Quando mensagens com botões são enviadas do Tiledesk:

- Botões são convertidos para **Reply Keyboard** do Telegram
- Layout: **2 botões por linha** para melhor visualização
- Teclado desaparece automaticamente após seleção (`one_time_keyboard`)
- Cliques nos botões aparecem no histórico do chat
- Suporta botões de tipo `text` e `action`

## Configuração

Este conector pode ser usado de duas formas:

### Opção 1: Standalone (Pacote NPM independente)

Se você vai usar o conector como aplicação independente, configure via arquivo `.env`:

#### Variáveis de Ambiente (.env)

```env
# URL base do conector
BASE_URL=https://seu-dominio.com

# URL da API do Tiledesk
API_URL=https://api.tiledesk.com/v3

# URL da API do Telegram
TELEGRAM_API_URL=https://api.telegram.org/bot

# URL de arquivos do Telegram
TELEGRAM_FILE_URL=https://api.telegram.org/file/bot

# MongoDB
MONGODB_URL=mongodb://localhost:27017/telegram-connector

# URL da API de Apps do Tiledesk
APPS_API_URL=https://api.tiledesk.com/v3/apps

# Nome da marca (opcional)
BRAND_NAME=Posiflow

# Porta do servidor
PORT=3000
```

#### Instalação Standalone

1. Clone o repositório:
```bash
git clone <url-do-repositorio>
cd posiflow-telegram-connector
```

2. Instale as dependências:
```bash
npm install
```

3. Configure as variáveis de ambiente:
```bash
cp .env.example .env
# Edite o arquivo .env com suas configurações
```

4. Inicie o servidor:
```bash
npm start
```

### Opção 2: Integrado com Tiledesk via Docker

Se você usa o Tiledesk em Docker Compose, as configurações são definidas no `docker-compose.yml` do projeto principal.

#### Instalação via Docker

1. Publique o pacote customizado no NPM:
```bash
npm publish
```

2. No `package.json` do servidor Tiledesk, substitua o pacote oficial pelo customizado:

**Antes (pacote oficial):**
```json
"@tiledesk/tiledesk-telegram-connector": "^1.0.0"
```

**Depois (pacote customizado):**
```json
"posiflow-telegram-connector": "^1.0.5"
```

O servidor Tiledesk irá carregar automaticamente o pacote customizado ao invés do oficial.

3. Configure as variáveis no `docker-compose.yml`:
```yaml
server:
  environment:
    - API_URL=${EXTERNAL_BASE_URL}/api
    - TELEGRAM_API_URL=https://api.telegram.org/bot
    - TELEGRAM_FILE_URL=https://api.telegram.org/file/bot
    - MONGODB_URI=${MONGODB_URI}
    - BASE_FILE_URL=${EXTERNAL_BASE_URL}/api/
```

**Variáveis importantes:**
- `TELEGRAM_API_URL` e `TELEGRAM_FILE_URL`: APIs públicas do Telegram
- `MONGODB_URI`: Conexão com MongoDB (pode ser Atlas ou local)
- `API_URL`: URL base da API do Tiledesk
- `BASE_FILE_URL`: URL base para upload de arquivos

Essas variáveis são lidas automaticamente pelo servidor Tiledesk que já integra o connector.

## Rotas da API

### Rotas Administrativas

- **GET /** - Página inicial de boas-vindas
- **GET /detail** - Detalhes da instalação
- **POST /install** - Instalar o app no projeto Tiledesk
- **POST /uninstall** - Desinstalar o app
- **GET /configure** - Interface de configuração
- **POST /update** - Atualizar configurações básicas
- **POST /update_advanced** - Atualizar configurações avançadas
- **POST /disconnect** - Desconectar Telegram do projeto

### Rotas de Mensagens

- **POST /tiledesk** - Webhook para mensagens do Tiledesk
- **POST /telegram** - Webhook para mensagens do Telegram

## Uso

### 1. Configuração Inicial

Acesse a interface de configuração:
```
https://seu-dominio.com/configure?project_id=SEU_PROJECT_ID&token=SEU_TOKEN
```

Preencha:
- **Bot Name**: Nome do bot Telegram
- **Telegram Token**: Token do bot criado no @BotFather
- **Department**: Departamento que receberá as conversas

### 2. Webhook do Telegram

O conector configura automaticamente o webhook do Telegram para:
```
https://seu-dominio.com/telegram?project_id=SEU_PROJECT_ID
```

### 3. Enviar Mensagens com Botões

No Tiledesk, envie mensagens com o formato:

```json
{
  "text": "Escolha uma opção:",
  "attributes": {
    "attachment": {
      "buttons": [
        { "type": "text", "value": "Opção 1" },
        { "type": "text", "value": "Opção 2" },
        { "type": "action", "value": "Confirmar" }
      ]
    }
  }
}
```

Os botões aparecerão no Telegram como Reply Keyboard com 2 botões por linha.

## Customizações Posiflow

### TiledeskTelegramTranslator.js

**Modificação principal**: Implementação de Reply Keyboard

```javascript
const BUTTONS_PER_ROW = 2;  // 2 botões por linha

// Botões são organizados em linhas
let keyboard_buttons = [];
let keyboard_row = [];

// Loop pelos botões e organiza em linhas de 2
for (let btn of buttons) {
  keyboard_row.push({ text: btn.value });

  if (keyboard_row.length === BUTTONS_PER_ROW) {
    keyboard_buttons.push(keyboard_row);
    keyboard_row = [];
  }
}

// Configuração do teclado
reply_markup: {
  keyboard: keyboard_buttons,
  resize_keyboard: true,
  one_time_keyboard: true
}
```

### Benefícios do Reply Keyboard

✅ Cliques aparecem no histórico do chat
✅ Melhor visibilidade em dispositivos móveis
✅ Interface mais intuitiva para usuários
✅ Teclado some após uso (não polui a interface)
✅ Suporta múltiplas linhas de botões

## Tipos de Mensagens Suportadas

### Do Telegram para Tiledesk

- ✅ Mensagens de texto
- ✅ Imagens/Fotos
- ✅ Vídeos
- ✅ Documentos/Arquivos
- ❌ Mensagens editadas (ignoradas)

### Do Tiledesk para Telegram

- ✅ Mensagens de texto
- ✅ Imagens
- ✅ Vídeos
- ✅ Documentos
- ✅ Botões (Reply Keyboard)
- ✅ Frames (links)
- ✅ Comandos encadeados

## Logs e Monitoramento

O sistema usa Winston para logging com níveis:

- **error**: Erros críticos
- **verbose**: Informações de fluxo
- **debug**: Detalhes de mensagens e requisições
- **info**: Inicialização e eventos importantes

## Limitações

- Botões de URL não são suportados no Reply Keyboard (apenas Inline Keyboard)
- Máximo de 36 caracteres por botão (texto é truncado automaticamente)
- Reply Keyboard funciona melhor com até 10 botões
- Mensagens editadas no Telegram são ignoradas

## Contribuição

Este é um projeto customizado para Posiflow. Para contribuir:

1. Faça fork do projeto
2. Crie uma branch para sua feature
3. Commit suas mudanças
4. Push para a branch
5. Abra um Pull Request

## Licença

ISC License

## Autor

**Posiflow** - Fork customizado com Reply Keyboard support

Baseado no projeto original: [@tiledesk/tiledesk-telegram-connector](https://www.npmjs.com/package/@tiledesk/tiledesk-telegram-connector)

## Suporte

Para questões e suporte, entre em contato com a equipe Posiflow.

---

**Versão**: 1.0.2
**Node.js**: 16.20.1
**Última atualização**: 2025