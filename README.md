# 🤖 Telegram Chatbot Manager

Um sistema completo de gerenciamento de chatbots do Telegram com interface web moderna, construído com HTML5, Bootstrap 5 e JavaScript vanilla. Permite gerenciar múltiplos bots, usuários, enviar mensagens em massa e acompanhar histórico de conversas em tempo real.

## ✨ Funcionalidades Principais

### 🔧 Gerenciamento de Chatbots
- ✅ Adicionar/remover chatbots com tokens da API do Telegram
- ✅ Ativar/desativar bots individualmente
- ✅ Validação automática de tokens
- ✅ Monitoramento de status (online/offline)

### 👥 Gerenciamento de Usuários
- ✅ Importação automática de usuários via API do Telegram
- ✅ Visualização em tabela paginada com filtros
- ✅ Busca por nome, username ou chat ID
- ✅ Sistema de bloqueio/desbloqueio de usuários
- ✅ Estatísticas detalhadas por usuário

### 💬 Sistema de Mensagens
- ✅ Envio de mensagens em tempo real
- ✅ Suporte a texto, fotos e documentos
- ✅ Mensagens em massa para múltiplos usuários
- ✅ Sistema de agendamento de mensagens
- ✅ Preview de mensagens antes do envio

### 📊 Histórico e Relatórios
- ✅ Histórico completo de conversas
- ✅ Visualização em formato de chat
- ✅ Exportação de dados em JSON
- ✅ Estatísticas de mensagens enviadas/recebidas
- ✅ Atualização em tempo real via API

## 🖼️ Capturas de Tela

> **Nota**: Adicione aqui as capturas de tela do seu projeto

### Dashboard Principal
<!-- Adicionar captura da tela inicial com navegação -->
*Tela inicial mostrando a navegação principal e cards de estatísticas*

### Gerenciamento de Chatbots
<!-- Adicionar captura da seção de chatbots -->
*Interface para adicionar/gerenciar chatbots com tabela de status*

### Lista de Usuários
<!-- Adicionar captura da tabela de usuários -->
*Tabela paginada com filtros, busca e ações por usuário*

### Detalhes do Usuário
<!-- Adicionar captura do modal de detalhes -->
*Modal com informações completas e histórico de mensagens*

### Envio de Mensagens
<!-- Adicionar captura da seção de mensagens -->
*Interface para composição e envio de mensagens com preview*

### Mensagens Agendadas
<!-- Adicionar captura do modal de agendamento -->
*Sistema de agendamento com tabela de mensagens pendentes*

## 🚀 Instalação e Configuração

### Pré-requisitos
- Navegador web moderno (Chrome, Firefox, Safari, Edge)
- Token de bot do Telegram (obtido via [@BotFather](https://t.me/BotFather))
- Servidor web local ou hosting (para contornar limitações de CORS)

### Instalação Simples

1. **Clone ou baixe o projeto**
   ```bash
   git clone [seu-repositorio]
   cd telegram-chatbot-manager
   ```

2. **Abra o arquivo HTML**
   ```bash
   # Opção 1: Servir via servidor local (recomendado)
   python -m http.server 8000
   # Acesse: http://localhost:8000/chatbot-manager.html
   
   # Opção 2: Abrir diretamente no navegador
   # Abra chatbot-manager.html no seu navegador
   ```

3. **Configure seu primeiro chatbot**
   - Vá para [@BotFather](https://t.me/BotFather) no Telegram
   - Crie um novo bot com `/newbot`
   - Copie o token fornecido
   - Adicione o bot na interface web

### Configuração para Produção

Para uso em produção, implemente um backend para contornar limitações de CORS:

#### Backend Node.js (Recomendado)
```javascript
// server.js
const express = require('express');
const cors = require('cors');
const axios = require('axios');

const app = express();
app.use(cors());
app.use(express.json());

// Proxy para API do Telegram
app.post('/api/telegram/:token/:method', async (req, res) => {
  try {
    const { token, method } = req.params;
    const response = await axios.post(
      `https://api.telegram.org/bot${token}/${method}`,
      req.body
    );
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log('Servidor rodando na porta 3000');
});
```

#### Backend Python (Flask)
```python
# app.py
from flask import Flask, request, jsonify
from flask_cors import CORS
import requests

app = Flask(__name__)
CORS(app)

@app.route('/api/telegram/<token>/<method>', methods=['POST'])
def telegram_proxy(token, method):
    try:
        response = requests.post(
            f'https://api.telegram.org/bot{token}/{method}',
            json=request.json
        )
        return response.json()
    except Exception as e:
        return {'error': str(e)}, 500

if __name__ == '__main__':
    app.run(debug=True, port=3000)
```

## 📖 Como Usar

### 1. Primeiro Acesso

1. **Adicione um Chatbot**
   - Clique em "Adicionar Chatbot"
   - Insira nome, token e descrição
   - Ative o chatbot (botão de energia)

2. **Importe Usuários**
   - Vá para a seção "Usuários"
   - Clique em "Buscar Usuários"
   - Selecione o chatbot ativo
   - Os usuários que interagiram com o bot serão importados

### 2. Enviando Mensagens

1. **Mensagem Individual**
   - Na lista de usuários, clique no ícone de envelope
   - Ou use "Ver Detalhes" → "Enviar Mensagem"

2. **Mensagem em Massa**
   - Selecione múltiplos usuários na tabela
   - Clique em "Mensagem em Massa"
   - Compose sua mensagem na seção correspondente

3. **Agendar Mensagens**
   - Na seção "Mensagens", marque "Agendar envio"
   - Defina data e horário
   - A mensagem ficará pendente até o horário definido

### 3. Gerenciando Usuários

- **Bloquear/Desbloquear**: Use os botões na tabela de usuários
- **Ver Histórico**: Clique em "Ver Detalhes" para histórico completo
- **Filtrar Usuários**: Use os filtros por chatbot, status ou busca
- **Exportar Dados**: Botão "Exportar" para backup em JSON

## 🔧 Funcionalidades Avançadas

### Sistema de Histórico
- Armazena todas as mensagens enviadas e recebidas
- Visualização em formato de chat com timestamps
- Indicadores de status de entrega
- Suporte a diferentes tipos de mídia

### API Integration
- Integração real com a API do Telegram
- Busca automática de novos usuários
- Atualização de histórico em tempo real
- Tratamento de erros e limitações de rate

### Armazenamento Local
- Todos os dados são salvos no localStorage do navegador
- Backup automático a cada ação
- Possibilidade de exportação para backup externo

## 🛠️ Tecnologias Utilizadas

- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **Framework CSS**: Bootstrap 5.3.0
- **Ícones**: Bootstrap Icons 1.11.0
- **API**: Telegram Bot API
- **Armazenamento**: localStorage (navegador)

## 📋 Estrutura do Projeto

```
telegram-chatbot-manager/
├── chatbot-manager.html      # Arquivo principal da aplicação
├── backend-implementation.md # Guia de implementação do backend
├── README.md                # Este arquivo
└── screenshots/             # Capturas de tela (adicionar)
    ├── dashboard.png
    ├── chatbots.png
    ├── users.png
    ├── user-details.png
    ├── messages.png
    └── scheduled.png
```

## 🔒 Considerações de Segurança

### Tokens de Bot
- ⚠️ **Nunca exponha tokens em código cliente**
- ✅ Use backend proxy para chamadas da API
- ✅ Implemente autenticação adequada
- ✅ Considere rotação regular de tokens

### Dados dos Usuários
- 📊 Todos os dados ficam no navegador (localStorage)
- 🔒 Para produção, considere criptografia local
- 📤 Implemente backup regular dos dados
- 🗑️ Forneça opções de limpeza de dados

## 🚨 Limitações e CORS

### Limitação de CORS
O navegador bloqueia requisições diretas para a API do Telegram devido à política de CORS. Soluções:

1. **Extensão CORS** (desenvolvimento):
   - Chrome: "CORS Unblock" ou "Disable CORS"
   - Firefox: "CORS Everywhere"

2. **Backend Proxy** (produção):
   - Use os exemplos de backend fornecidos
   - Implemente autenticação adequada

3. **Hosting com Proxy**:
   - Vercel, Netlify com Functions
   - Cloudflare Workers

## 🤝 Contribuição

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-funcionalidade`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova funcionalidade'`)
4. Push para a branch (`git push origin feature/nova-funcionalidade`)
5. Abra um Pull Request

### Roadmap de Funcionalidades

- [ ] 🔐 Sistema de autenticação
- [ ] 🗄️ Backend com banco de dados
- [ ] 📱 Responsividade mobile melhorada
- [ ] 🔔 Notificações push
- [ ] 📈 Dashboard com gráficos
- [ ] 🤖 Templates de mensagens
- [ ] 🌐 Suporte a múltiplos idiomas
- [ ] 📊 Relatórios avançados
- [ ] 🔗 Webhooks para automação
- [ ] 💰 Sistema de cobrança/limites

## 📞 Suporte

### Problemas Comuns

**1. Erro de CORS**
```
Erro: Access to fetch at 'https://api.telegram.org' from origin has been blocked by CORS policy
```
**Solução**: Use extensão CORS ou implemente backend proxy

**2. Token inválido**
```
Erro: Unauthorized - Bot token is invalid
```
**Solução**: Verifique o token no @BotFather

**3. Usuários não aparecem**
```
Nenhum usuário encontrado
```
**Solução**: Certifique-se que usuários interagiram com o bot

### Contato

- 📧 Email: [seu-email@exemplo.com]
- 💬 Telegram: [@seu-usuario]
- 🐛 Issues: [link-do-repositorio/issues]

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 🏆 Agradecimentos

- [Telegram Bot API](https://core.telegram.org/bots/api) pela excelente documentação
- [Bootstrap](https://getbootstrap.com/) pelo framework CSS
- [Bootstrap Icons](https://icons.getbootstrap.com/) pelos ícones
- Comunidade de desenvolvedores por feedback e sugestões

---

<div align="center">

**⭐ Se este projeto foi útil, considere dar uma estrela!**

[Demo Online](seu-link-demo) • [Documentação](seu-link-docs) • [Reportar Bug](seu-link-issues)

</div>