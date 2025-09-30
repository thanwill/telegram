# API do Telegram - Implementação Backend

Este arquivo contém exemplos de como implementar um backend para contornar as restrições CORS ao acessar a API do Telegram.

## 🚨 Problema CORS

O navegador bloqueia requisições diretas para a API do Telegram devido às políticas CORS. Para resolver isso, você tem 3 opções:

### 1. 🔧 Extensão CORS (Desenvolvimento)
- Instale uma extensão como "CORS Unblock" no Chrome
- **⚠️ Use apenas para desenvolvimento!**

### 2. 🖥️ Backend/Proxy (Recomendado para Produção)

#### Node.js + Express
```javascript
const express = require('express');
const cors = require('cors');
const axios = require('axios');

const app = express();
app.use(cors());
app.use(express.json());

// Endpoint para buscar usuários
app.post('/api/telegram-users', async (req, res) => {
    try {
        const { botToken } = req.body;
        
        // Buscar updates do Telegram
        const response = await axios.get(`https://api.telegram.org/bot${botToken}/getUpdates?limit=100`);
        
        if (!response.data.ok) {
            return res.status(400).json({ error: response.data.description });
        }
        
        const updates = response.data.result;
        const usersMap = new Map();
        
        // Processar updates para extrair usuários
        updates.forEach(update => {
            let user = null;
            let chatId = null;
            let messageDate = null;

            if (update.message) {
                user = update.message.from;
                chatId = update.message.chat.id;
                messageDate = new Date(update.message.date * 1000);
            } else if (update.callback_query) {
                user = update.callback_query.from;
                chatId = update.callback_query.message.chat.id;
                messageDate = new Date(update.callback_query.message.date * 1000);
            }

            if (user && chatId) {
                const userId = user.id;
                
                if (!usersMap.has(userId) || usersMap.get(userId).last_interaction < messageDate) {
                    usersMap.set(userId, {
                        id: userId,
                        chat_id: chatId,
                        first_name: user.first_name || 'Unknown',
                        last_name: user.last_name || '',
                        username: user.username || null,
                        status: 'active',
                        last_interaction: messageDate.toISOString(),
                        message_count: (usersMap.get(userId)?.message_count || 0) + 1,
                        joined_date: usersMap.get(userId)?.joined_date || messageDate.toISOString(),
                        language_code: user.language_code || null,
                        is_bot: user.is_bot || false
                    });
                } else {
                    const existingUser = usersMap.get(userId);
                    existingUser.message_count++;
                }
            }
        });
        
        res.json(Array.from(usersMap.values()));
        
    } catch (error) {
        console.error('Erro:', error);
        res.status(500).json({ error: error.message });
    }
});

app.listen(3001, () => {
    console.log('Servidor rodando na porta 3001');
});
```

#### Python + Flask
```python
from flask import Flask, request, jsonify
from flask_cors import CORS
import requests

app = Flask(__name__)
CORS(app)

@app.route('/api/telegram-users', methods=['POST'])
def get_telegram_users():
    try:
        data = request.json
        bot_token = data['botToken']
        
        # Buscar updates do Telegram
        response = requests.get(f'https://api.telegram.org/bot{bot_token}/getUpdates?limit=100')
        
        if not response.json()['ok']:
            return jsonify({'error': response.json()['description']}), 400
        
        updates = response.json()['result']
        users_map = {}
        
        # Processar updates para extrair usuários
        for update in updates:
            user = None
            chat_id = None
            message_date = None
            
            if 'message' in update:
                user = update['message']['from']
                chat_id = update['message']['chat']['id']
                message_date = update['message']['date']
            elif 'callback_query' in update:
                user = update['callback_query']['from']
                chat_id = update['callback_query']['message']['chat']['id']
                message_date = update['callback_query']['message']['date']
            
            if user and chat_id:
                user_id = user['id']
                
                if user_id not in users_map:
                    users_map[user_id] = {
                        'id': user_id,
                        'chat_id': chat_id,
                        'first_name': user.get('first_name', 'Unknown'),
                        'last_name': user.get('last_name', ''),
                        'username': user.get('username'),
                        'status': 'active',
                        'last_interaction': message_date,
                        'message_count': 1,
                        'joined_date': message_date,
                        'language_code': user.get('language_code'),
                        'is_bot': user.get('is_bot', False)
                    }
                else:
                    users_map[user_id]['message_count'] += 1
        
        return jsonify(list(users_map.values()))
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True, port=3001)
```

### 3. 📝 Atualizar o Frontend para usar Backend

No arquivo `chatbot-manager.html`, substitua a função `fetchUsersFromTelegram`:

```javascript
async function fetchUsersFromTelegram(botToken, chatbotId) {
    try {
        // Usar o backend local em vez da API direta
        const response = await fetch('http://localhost:3001/api/telegram-users', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({ botToken })
        });
        
        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.error || 'Erro no servidor');
        }
        
        const users = await response.json();
        
        // Adicionar chatbotId aos usuários
        return users.map(user => ({
            ...user,
            id: Date.now() + Math.random(),
            chatbotId: parseInt(chatbotId),
            last_interaction: new Date(user.last_interaction * 1000).toISOString(),
            joined_date: new Date(user.joined_date * 1000).toISOString()
        }));
        
    } catch (error) {
        throw error;
    }
}
```

## 🚀 Como Usar

1. **Escolha uma implementação** (Node.js ou Python)
2. **Instale as dependências**:
   - Node.js: `npm install express cors axios`
   - Python: `pip install flask flask-cors requests`
3. **Execute o servidor**:
   - Node.js: `node server.js`
   - Python: `python app.py`
4. **Atualize o frontend** para usar o endpoint local
5. **Teste a funcionalidade** no navegador

## 📚 Recursos Adicionais

- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [Webhook vs Long Polling](https://core.telegram.org/bots/webhooks)
- [Bot Security Best Practices](https://core.telegram.org/bots/faq#security)

## ⚠️ Considerações de Segurança

- Nunca exponha tokens de bot no frontend
- Use variáveis de ambiente para tokens
- Implemente autenticação no backend
- Use HTTPS em produção
- Limite rate das requisições