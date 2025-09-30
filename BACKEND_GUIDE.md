# Backend Implementation Guide - Completo

Este documento fornece instruções detalhadas para implementar um backend que resolve as limitações de CORS ao trabalhar com a API do Telegram.

## Problema de CORS

A API do Telegram não permite requisições diretas do navegador devido à política de CORS (Cross-Origin Resource Sharing). Isso significa que você precisa de um servidor intermediário (proxy) para fazer as chamadas à API.

## Soluções Implementadas

### 1. Node.js + Express

#### Instalação
```bash
npm init -y
npm install express cors axios dotenv
```

#### Código Completo (server.js)
```javascript
const express = require('express');
const cors = require('cors');
const axios = require('axios');
const path = require('path');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.static('public')); // Servir arquivos estáticos

// Servir o arquivo HTML principal
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'chatbot-manager.html'));
});

// Proxy para API do Telegram - Método genérico
app.all('/api/telegram/:token/:method', async (req, res) => {
    try {
        const { token, method } = req.params;
        const url = `https://api.telegram.org/bot${token}/${method}`;
        
        let response;
        if (req.method === 'GET') {
            response = await axios.get(url, { params: req.query });
        } else {
            response = await axios.post(url, req.body);
        }
        
        res.json(response.data);
    } catch (error) {
        console.error('Erro na API do Telegram:', error.response?.data || error.message);
        res.status(error.response?.status || 500).json({
            error: error.response?.data || { description: error.message }
        });
    }
});

// Endpoints específicos para facilitar o uso
app.post('/api/telegram/:token/sendMessage', async (req, res) => {
    try {
        const { token } = req.params;
        const response = await axios.post(
            `https://api.telegram.org/bot${token}/sendMessage`,
            req.body
        );
        res.json(response.data);
    } catch (error) {
        res.status(error.response?.status || 500).json({
            error: error.response?.data || { description: error.message }
        });
    }
});

app.get('/api/telegram/:token/getUpdates', async (req, res) => {
    try {
        const { token } = req.params;
        const response = await axios.get(
            `https://api.telegram.org/bot${token}/getUpdates`,
            { params: req.query }
        );
        res.json(response.data);
    } catch (error) {
        res.status(error.response?.status || 500).json({
            error: error.response?.data || { description: error.message }
        });
    }
});

app.listen(PORT, () => {
    console.log(`🚀 Servidor rodando na porta ${PORT}`);
    console.log(`📱 Acesse: http://localhost:${PORT}`);
});
```

#### Arquivo .env
```env
PORT=3000
# Adicione outras configurações se necessário
```

#### Package.json
```json
{
  "name": "telegram-chatbot-backend",
  "version": "1.0.0",
  "description": "Backend para Telegram Chatbot Manager",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "axios": "^1.6.0",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

### 2. Python + Flask

#### Instalação
```bash
pip install flask flask-cors requests python-dotenv
```

#### Código Completo (app.py)
```python
from flask import Flask, request, jsonify, send_from_directory
from flask_cors import CORS
import requests
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
CORS(app)

# Configurações
TELEGRAM_API_BASE = "https://api.telegram.org/bot"

@app.route('/')
def index():
    return send_from_directory('.', 'chatbot-manager.html')

@app.route('/api/telegram/<token>/<method>', methods=['GET', 'POST', 'PUT', 'DELETE'])
def telegram_proxy(token, method):
    try:
        url = f"{TELEGRAM_API_BASE}{token}/{method}"
        
        if request.method == 'GET':
            response = requests.get(url, params=request.args)
        else:
            response = requests.post(url, json=request.json)
        
        return response.json(), response.status_code
        
    except requests.exceptions.RequestException as e:
        return {'error': {'description': str(e)}}, 500
    except Exception as e:
        return {'error': {'description': f'Erro interno: {str(e)}'}}, 500

@app.route('/api/telegram/<token>/sendMessage', methods=['POST'])
def send_message(token):
    try:
        url = f"{TELEGRAM_API_BASE}{token}/sendMessage"
        response = requests.post(url, json=request.json)
        return response.json(), response.status_code
    except Exception as e:
        return {'error': {'description': str(e)}}, 500

@app.route('/api/telegram/<token>/getUpdates', methods=['GET'])
def get_updates(token):
    try:
        url = f"{TELEGRAM_API_BASE}{token}/getUpdates"
        response = requests.get(url, params=request.args)
        return response.json(), response.status_code
    except Exception as e:
        return {'error': {'description': str(e)}}, 500

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```

### 3. Cloudflare Workers

#### worker.js
```javascript
addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
    const url = new URL(request.url)
    
    // CORS headers
    const corsHeaders = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type',
    }

    // Handle CORS preflight
    if (request.method === 'OPTIONS') {
        return new Response(null, { headers: corsHeaders })
    }

    // Proxy para API do Telegram
    if (url.pathname.startsWith('/api/telegram/')) {
        const pathParts = url.pathname.split('/')
        const token = pathParts[3]
        const method = pathParts[4]
        
        const telegramUrl = `https://api.telegram.org/bot${token}/${method}`
        
        const modifiedRequest = new Request(telegramUrl, {
            method: request.method,
            headers: request.headers,
            body: request.body
        })

        const response = await fetch(modifiedRequest)
        const data = await response.text()

        return new Response(data, {
            status: response.status,
            headers: {
                ...corsHeaders,
                'Content-Type': 'application/json'
            }
        })
    }

    return new Response('Not Found', { status: 404 })
}
```

## Modificações no Frontend

Para usar o backend, você precisará modificar as URLs das requisições no arquivo HTML:

### Configuração da URL Base
```javascript
// Adicione no início do arquivo JavaScript
const API_BASE_URL = 'http://localhost:3000'; // Para desenvolvimento local
// const API_BASE_URL = 'https://seu-dominio.com'; // Para produção

// Função helper para construir URLs da API
function buildApiUrl(token, method) {
    return `${API_BASE_URL}/api/telegram/${token}/${method}`;
}
```

### Atualizar Função sendSingleMessage
```javascript
async function sendSingleMessage(botToken, chatId, messageType, content, mediaUrl) {
    try {
        let apiUrl;
        let payload;

        switch (messageType) {
            case 'text':
                apiUrl = buildApiUrl(botToken, 'sendMessage');
                payload = {
                    chat_id: chatId,
                    text: content,
                    parse_mode: 'HTML'
                };
                break;
            // ... outros casos
        }

        const response = await fetch(apiUrl, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(payload)
        });

        const data = await response.json();
        // ... resto da função
    } catch (error) {
        // ... tratamento de erro
    }
}
```

### Atualizar fetchUsersFromTelegram
```javascript
async function fetchUsersFromTelegram(botToken, chatbotId) {
    try {
        const updatesUrl = buildApiUrl(botToken, 'getUpdates') + '?limit=100';
        const updatesResponse = await fetch(updatesUrl);
        
        // ... resto da função
    } catch (error) {
        // ... tratamento de erro
    }
}
```

## Deploy em Produção

### Heroku (Node.js)
1. Crie uma conta no Heroku
2. Instale o Heroku CLI
3. Execute os comandos:
```bash
heroku create seu-app-name
git add .
git commit -m "Deploy inicial"
git push heroku main
```

### Vercel (Node.js)
1. Instale o Vercel CLI: `npm i -g vercel`
2. Execute: `vercel`
3. Siga as instruções

### PythonAnywhere (Python)
1. Faça upload dos arquivos
2. Configure o WSGI file
3. Reinicie a aplicação

### Railway (Node.js/Python)
1. Conecte seu repositório GitHub
2. Railway detecta automaticamente o tipo de projeto
3. Deploy automático

## Estrutura de Arquivos Recomendada

```
projeto/
├── server.js (ou app.py)           # Servidor backend
├── package.json (ou requirements.txt)
├── .env                            # Variáveis de ambiente
├── .gitignore                      # Ignorar node_modules, .env, etc.
├── public/                         # Arquivos estáticos
│   ├── chatbot-manager.html        # Frontend
│   └── assets/                     # CSS, JS, imagens
├── README.md
└── backend-implementation.md       # Este arquivo
```

## Segurança

### Validação de Tokens
```javascript
// Adicione validação de token no backend
function validateBotToken(token) {
    const tokenRegex = /^\d+:[A-Za-z0-9_-]+$/;
    return tokenRegex.test(token);
}

app.use('/api/telegram/:token/*', (req, res, next) => {
    if (!validateBotToken(req.params.token)) {
        return res.status(400).json({
            error: { description: 'Token inválido' }
        });
    }
    next();
});
```

### Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 1000, // 1 segundo
    max: 30, // máximo 30 requisições por segundo
    message: {
        error: { description: 'Muitas requisições, tente novamente' }
    }
});

app.use('/api/telegram', limiter);
```

### Headers de Segurança
```javascript
const helmet = require('helmet');
app.use(helmet());
```

## Logs e Monitoramento

```javascript
// Middleware de log
app.use((req, res, next) => {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
    next();
});

// Log de erros
app.use((err, req, res, next) => {
    console.error('Erro:', err);
    res.status(500).json({
        error: { description: 'Erro interno do servidor' }
    });
});
```

Este backend resolve completamente as limitações de CORS e permite que sua aplicação funcione em produção de forma segura e eficiente.