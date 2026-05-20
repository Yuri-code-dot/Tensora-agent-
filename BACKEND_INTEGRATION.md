# TENSORA Backend Integration Guide

Complete guide for connecting your AI backend to the TENSORA frontend.

## Table of Contents

1. [Quick Start](#quick-start)
2. [API Specifications](#api-specifications)
3. [Backend Examples](#backend-examples)
4. [Production Setup](#production-setup)
5. [Debugging](#debugging)
6. [Security](#security)

---

## Quick Start

### Prerequisites

- A running HTTP server/API
- CORS enabled for your frontend domain
- Optional: LLM API key (OpenAI, Anthropic, etc.)

### 1. Implement Required Endpoints

Your backend must have these two endpoints:

```
GET  /api/health
POST /api/chat
```

### 2. Connect Frontend

```javascript
// In browser console
setBackendConfig('http://localhost:8000/api', 'optional-api-key', false);
```

### 3. Test

Send a message in any mode. Check browser console for logs.

---

## API Specifications

### Endpoint: GET /api/health

**Purpose**: Check if backend is online and ready

**Request**:
```http
GET /api/health HTTP/1.1
Host: localhost:8000
```

**Response** (200 OK):
```json
{
  "status": "ok",
  "timestamp": "2024-01-15T10:30:45Z",
  "version": "1.0.0"
}
```

**Example responses**:

```bash
# Successful
curl http://localhost:8000/api/health
{"status":"ok","timestamp":"2024-01-15T10:30:45Z"}

# Backend down - no response or timeout
curl http://localhost:8000/api/health
# Connection refused
```

---

### Endpoint: POST /api/chat

**Purpose**: Process user prompts through selected mode

**Request**:
```http
POST /api/chat HTTP/1.1
Content-Type: application/json
Authorization: Bearer optional-api-key

{
  "prompt": "What is machine learning?",
  "system": "You are a helpful assistant",
  "agent": "RESEARCH",
  "mode": "INSTANT",
  "timestamp": 1705330496000
}
```

**Parameters**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `prompt` | string | ✓ | User's input message |
| `system` | string | ✗ | System prompt/role for agent |
| `agent` | string | ✗ | Agent name (for MAX mode logging) |
| `mode` | string | ✓ | "INSTANT" \| "EXTREME" \| "MAX" |
| `timestamp` | number | ✗ | Unix timestamp in milliseconds |

**Response** (200 OK):
```json
{
  "reply": "Machine learning is a subset of artificial intelligence...",
  "agent": "RESEARCH",
  "tokens_used": 127,
  "mode": "INSTANT"
}
```

**Response fields**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reply` | string | ✓ | LLM response text |
| `agent` | string | ✗ | Agent that processed (for logging) |
| `tokens_used` | number | ✗ | Token count (if tracking) |
| `mode` | string | ✗ | Processing mode used |

**Error Response** (400+):
```json
{
  "error": "Invalid prompt",
  "code": "INVALID_INPUT",
  "details": "Prompt cannot be empty"
}
```

---

## Backend Examples

### Node.js + Express

```javascript
// server.js
const express = require('express');
const cors = require('cors');
const { Configuration, OpenAIApi } = require('openai');

const app = express();
app.use(cors());
app.use(express.json());

// Initialize OpenAI
const openai = new OpenAIApi(
  new Configuration({
    apiKey: process.env.OPENAI_API_KEY,
  })
);

// Health check
app.get('/api/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    version: '1.0.0'
  });
});

// Chat endpoint
app.post('/api/chat', async (req, res) => {
  try {
    const { prompt, system, agent, mode } = req.body;

    if (!prompt) {
      return res.status(400).json({ error: 'Prompt is required' });
    }

    // Call OpenAI
    const response = await openai.createChatCompletion({
      model: 'gpt-4',
      messages: [
        { role: 'system', content: system || 'You are a helpful assistant.' },
        { role: 'user', content: prompt }
      ],
      temperature: 0.7,
      max_tokens: 2000,
    });

    const reply = response.data.choices[0].message.content;

    res.json({
      reply,
      agent: agent || 'ASSISTANT',
      tokens_used: response.data.usage.total_tokens,
      mode
    });
  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({
      error: 'Processing failed',
      details: error.message
    });
  }
});

app.listen(8000, () => console.log('TENSORA backend on :8000'));
```

**Run**:
```bash
npm install express cors openai
OPENAI_API_KEY=sk-... node server.js
```

---

### Python + Flask

```python
# server.py
from flask import Flask, request, jsonify
from flask_cors import CORS
import openai
from datetime import datetime

app = Flask(__name__)
CORS(app)

openai.api_key = "sk-..."

@app.route('/api/health', methods=['GET'])
def health():
    return jsonify({
        'status': 'ok',
        'timestamp': datetime.utcnow().isoformat() + 'Z',
        'version': '1.0.0'
    })

@app.route('/api/chat', methods=['POST'])
def chat():
    try:
        data = request.json
        prompt = data.get('prompt')
        system = data.get('system', 'You are a helpful assistant.')
        agent = data.get('agent', 'ASSISTANT')
        mode = data.get('mode')

        if not prompt:
            return jsonify({'error': 'Prompt is required'}), 400

        # Call OpenAI
        response = openai.ChatCompletion.create(
            model='gpt-4',
            messages=[
                {'role': 'system', 'content': system},
                {'role': 'user', 'content': prompt}
            ],
            temperature=0.7,
            max_tokens=2000
        )

        reply = response['choices'][0]['message']['content']

        return jsonify({
            'reply': reply,
            'agent': agent,
            'tokens_used': response['usage']['total_tokens'],
            'mode': mode
        })

    except Exception as e:
        print(f'Error: {e}')
        return jsonify({
            'error': 'Processing failed',
            'details': str(e)
        }), 500

if __name__ == '__main__':
    app.run(port=8000, debug=True)
```

**Run**:
```bash
pip install flask flask-cors openai
python server.py
```

---

### Go + Gin

```go
// main.go
package main

import (
	"github.com/gin-contrib/cors"
	"github.com/gin-gonic/gin"
	"github.com/sashabaranov/go-openai"
	"os"
	"time"
)

type ChatRequest struct {
	Prompt    string `json:"prompt"`
	System    string `json:"system"`
	Agent     string `json:"agent"`
	Mode      string `json:"mode"`
	Timestamp int64  `json:"timestamp"`
}

type ChatResponse struct {
	Reply      string `json:"reply"`
	Agent      string `json:"agent"`
	TokensUsed int    `json:"tokens_used"`
	Mode       string `json:"mode"`
}

func main() {
	r := gin.Default()
	r.Use(cors.Default())

	r.GET("/api/health", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"status":    "ok",
			"timestamp": time.Now().UTC().Format(time.RFC3339),
			"version":   "1.0.0",
		})
	})

	r.POST("/api/chat", func(c *gin.Context) {
		var req ChatRequest
		if err := c.BindJSON(&req); err != nil {
			c.JSON(400, gin.H{"error": "Invalid request"})
			return
		}

		if req.Prompt == "" {
			c.JSON(400, gin.H{"error": "Prompt is required"})
			return
		}

		client := openai.NewClient(os.Getenv("OPENAI_API_KEY"))

		resp, err := client.CreateChatCompletion(
			c.Request.Context(),
			openai.ChatCompletionRequest{
				Model: openai.GPT4,
				Messages: []openai.ChatCompletionMessage{
					{
						Role:    "system",
						Content: req.System,
					},
					{
						Role:    "user",
						Content: req.Prompt,
					},
				},
				Temperature: 0.7,
				MaxTokens:   2000,
			},
		)

		if err != nil {
			c.JSON(500, gin.H{"error": err.Error()})
			return
		}

		reply := resp.Choices[0].Message.Content

		c.JSON(200, ChatResponse{
			Reply:      reply,
			Agent:      req.Agent,
			TokensUsed: resp.Usage.TotalTokens,
			Mode:       req.Mode,
		})
	})

	r.Run(":8000")
}
```

**Run**:
```bash
go get github.com/gin-gonic/gin github.com/sashabaranov/go-openai
OPENAI_API_KEY=sk-... go run main.go
```

---

## Production Setup

### Docker

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

ENV PORT=8000
ENV NODE_ENV=production

EXPOSE 8000
CMD ["node", "server.js"]
```

```bash
docker build -t tensora-backend .
docker run -p 8000:8000 \
  -e OPENAI_API_KEY=sk-... \
  tensora-backend
```

### Environment Variables

```bash
# .env
OPENAI_API_KEY=sk-...
PORT=8000
NODE_ENV=production
CORS_ORIGIN=https://yourdomain.com
LOG_LEVEL=info
```

### CORS Configuration

```javascript
// For production
const cors = require('cors');

const corsOptions = {
  origin: process.env.CORS_ORIGIN || 'http://localhost:3000',
  credentials: true,
  optionsSuccessStatus: 200,
  methods: ['GET', 'POST', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
};

app.use(cors(corsOptions));
```

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use('/api/', limiter);
```

---

## Debugging

### Browser Console

```javascript
// Check current config
console.log(CONFIG);

// View API calls
// Open DevTools → Network tab → filter by "chat" or "health"

// Test backend manually
fetch('http://localhost:8000/api/health')
  .then(r => r.json())
  .then(d => console.log(d))

// Test chat endpoint
fetch('http://localhost:8000/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    prompt: 'Hello',
    system: 'Be helpful',
    mode: 'INSTANT'
  })
})
  .then(r => r.json())
  .then(d => console.log(d))
```

### Backend Logs

```javascript
// Add detailed logging
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  console.log('Body:', req.body);
  next();
});

app.post('/api/chat', (req, res) => {
  console.log('Chat request received:', req.body);
  // ... process
  console.log('Chat response sending:', response);
  res.json(response);
});
```

### Network Inspection

```bash
# Test with curl
curl http://localhost:8000/api/health

curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Hello",
    "system": "Be helpful",
    "mode": "INSTANT"
  }'
```

---

## Security

### API Keys

```javascript
// Never expose API keys in frontend
// Store in environment variables on backend

// Good ✓
const apiKey = process.env.OPENAI_API_KEY;

// Bad ✗
const apiKey = 'sk-...'; // Hardcoded
```

### CORS

```javascript
// Restrict to specific origins
const corsOptions = {
  origin: 'https://yourdomain.com',
  credentials: true
};
```

### Input Validation

```javascript
// Validate request
if (!prompt || prompt.length > 10000) {
  return res.status(400).json({ error: 'Invalid prompt' });
}

if (!['INSTANT', 'EXTREME', 'MAX'].includes(mode)) {
  return res.status(400).json({ error: 'Invalid mode' });
}
```

### Rate Limiting

```javascript
// Prevent abuse
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
});

app.use('/api/', limiter);
```

### HTTPS

```bash
# In production, always use HTTPS
# Frontend: https://yourdomain.com
# Backend: https://api.yourdomain.com
```

---

## Troubleshooting

### Frontend shows "● offline (mock)"

**Problem**: Backend not connecting

**Solutions**:
1. Check backend is running: `curl http://localhost:8000/api/health`
2. Verify API URL: `console.log(CONFIG.API_URL)`
3. Check CORS headers in browser DevTools
4. Check for network errors in DevTools → Network tab
5. Verify firewall isn't blocking port

### "CORS error" in console

**Problem**: Cross-Origin Resource Sharing blocked

**Solutions**:
1. Add CORS to backend:
   ```javascript
   app.use(cors());
   ```
2. Specify allowed origins:
   ```javascript
   app.use(cors({ origin: 'http://localhost:3000' }));
   ```

### Messages return "[mock]" prefix

**Problem**: Using mock mode

**Solution**: Disable mock mode
```javascript
setBackendConfig('http://localhost:8000/api', '', false);
```

### Backend returns 500 error

**Problem**: LLM API error

**Check**:
1. LLM API keys are set: `echo $OPENAI_API_KEY`
2. LLM provider is accessible
3. Rate limits haven't been exceeded
4. Check backend logs for details

### Slow responses

**Problem**: Performance issue

**Solutions**:
1. Verify LLM provider isn't rate-limited
2. Check network latency
3. Monitor CPU/memory on backend
4. Consider streaming responses
5. Optimize LLM model choice

---

## Next Steps

1. ✅ Choose a backend framework (Node, Python, Go, etc)
2. ✅ Implement `/api/health` and `/api/chat` endpoints
3. ✅ Set up LLM API access (OpenAI, Anthropic, local)
4. ✅ Enable CORS for your frontend domain
5. ✅ Connect frontend: `setBackendConfig('...', '', false)`
6. ✅ Test in browser
7. ✅ Deploy to production

**You're ready to power TENSORA with your AI backend!** 🚀
