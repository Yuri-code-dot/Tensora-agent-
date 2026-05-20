# ◢ TENSORA - Neural Agent Interface

Tensora is a multi-modal AI agent orchestration platform with three distinct processing modes for different task complexities and parallelization strategies.

## Features

- 🚀 **3 Processing Modes**: INSTANT, EXTREME, MAX
- 🤖 **Agent-Based Architecture**: 10 parallel agents in MAX mode
- 📊 **Real-Time Agent Logging**: Monitor agent activity live
- 💾 **Local Chat History**: Persistent storage per mode
- 🎨 **Dark Cyberpunk UI**: Responsive, modern design
- ⚡ **Fast & Lightweight**: Pure HTML/CSS/JavaScript (no dependencies)
- 🔗 **Backend Ready**: Easy integration with any REST API

## Quick Start

### 1. Open in Browser
```bash
# Simply open Index.html in any modern browser
# No server required for initial testing
```

### 2. Connect Your Backend

In browser console:
```javascript
// Development with mock (testing UI)
setBackendConfig('http://localhost:8000/api', '', true);

// Production (connect real backend)
setBackendConfig('https://api.yourdomain.com/api', 'your-api-key', false);
```

### 3. Start Chatting
- Select mode (INSTANT, EXTREME, or MAX)
- Type your prompt
- Press Send or Shift+Enter

## Processing Modes

### ▶ INSTANT
Fast, single-AI direct responses. Ideal for quick queries.

- **Single LLM call** to a fast assistant
- **Response time**: < 3 seconds
- **Use case**: Quick questions, simple tasks

```
User: "What is AI?"
→ Fast Assistant processes
→ Instant response
```

### ▶ EXTREME
Deep step-by-step reasoning with feedback loops. Best for complex analysis.

**Pipeline**: PLANNER → WRITER → REVIEWER → FINALIZER

1. **PLANNER**: Breaks down task into steps
2. **WRITER**: Drafts response based on plan
3. **REVIEWER**: Critiques the draft
4. **FINALIZER**: Revises based on feedback

- **4 sequential LLM calls**
- **Response time**: 5-15 seconds
- **Use case**: Complex analysis, detailed content, refined outputs

```
User: "Write a technical proposal"
→ PLANNER: Plans structure
→ WRITER: Drafts content
→ REVIEWER: Critiques draft
→ FINALIZER: Polishes
→ Comprehensive proposal
```

### ▶ MAX
Parallel multi-agent processing for comprehensive solutions.

**10 Parallel Agents**:
- RESEARCH - Topic investigation
- CODING - Programming solutions
- REVIEW - Quality assurance
- OPTIMIZE - Performance analysis
- TESTING - Test scenarios
- UI - Interface/UX design
- LOGIC - Reasoning validation
- DATA - Data strategies
- DEBUG - Troubleshooting
- DEPLOY - Deployment planning

Then **SYNTH** synthesizes all responses into final answer.

- **10 concurrent LLM calls** + synthesis
- **Response time**: 10-20 seconds
- **Use case**: Comprehensive analysis, multi-aspect problems, full solution planning

```
User: "Build a recommendation system"
→ 10 agents work in parallel:
   ├─ RESEARCH investigates best practices
   ├─ CODING designs architecture
   ├─ REVIEW checks quality
   ├─ OPTIMIZE suggests improvements
   ├─ TESTING plans test strategy
   ├─ UI designs interface
   ├─ LOGIC validates reasoning
   ├─ DATA designs data model
   ├─ DEBUG identifies edge cases
   └─ DEPLOY plans deployment
→ SYNTH combines all insights
→ Comprehensive system design
```

## Architecture

```
┌──────────────────────────────────────────────┐
│        TENSORA Frontend (Index.html)         │
│  - 3 Processing Modes                        │
│  - Chat History Management                   │
│  - Agent Activity Logging                    │
│  - Real-time UI Updates                      │
└───────────────────┬──────────────────────────┘
                    │ HTTP REST API
                    │ (or mock mode)
                    ▼
        ┌───────────────────────────┐
        │   Your Backend Server     │
        │ (Express, Flask, Go, etc) │
        └────────────┬──────────────┘
                     │
                     ▼
        ┌───────────────────────────┐
        │    LLM Provider           │
        │ (OpenAI, Claude, Local)   │
        └───────────────────────────┘
```

## Backend Integration

### Minimal Backend Example (Node.js)

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.post('/api/chat', async (req, res) => {
  const { prompt, system, agent, mode } = req.body;
  
  // Call your LLM here
  const reply = await callYourLLM(prompt, system);
  
  res.json({ reply, agent });
});

app.listen(8000);
```

### Frontend Connection

```javascript
// Browser console
setBackendConfig('http://localhost:8000/api', '', false);

// Now TENSORA will use your real backend!
```

### Full Backend Setup Guide

See **BACKEND_INTEGRATION.md** for:
- Detailed API specifications
- Complete code examples (Node.js, Python, Go)
- CORS configuration
- Testing & debugging
- Performance optimization
- Production deployment
- Security checklist

## Configuration

### Via Browser Console

```javascript
// Show current config
console.log(CONFIG);

// Change backend
setBackendConfig('https://api.example.com/api', 'api-key', false);

// Use mock mode
setBackendConfig('', '', true);
```

### Via Environment

Settings stored in localStorage:
- `tensora_api_url` - Backend API endpoint
- `tensora_api_key` - Optional API key
- `tensora_use_mock` - Use mock responses (true/false)

### Via .env File (Backend)

See **.env.example** for configuration template.

## Chat History

- Stored in browser localStorage
- Separate history per mode
- Survives page refreshes
- Clear individual mode with CLR button

**Storage structure:**
```javascript
{
  INSTANT: [ { role, content, mode, ts }, ... ],
  EXTREME: [ ... ],
  MAX: [ ... ]
}
```

## API Endpoints Required

### Health Check
```
GET /api/health
→ { status: "ok", timestamp: "..." }
```

### Chat Processing
```
POST /api/chat
← {
    prompt: "user message",
    system: "role/context",
    agent: "agent name",
    mode: "INSTANT|EXTREME|MAX",
    timestamp: 1705330496000
  }
→ {
    reply: "response text",
    agent: "agent name",
    tokens_used: 42
  }
```

## Browser Console Commands

```javascript
// Check backend status
checkBackendHealth();

// Clear all history
clearChat();

// View configuration
console.log(CONFIG);

// Update backend
setBackendConfig('http://localhost:8000/api', '', false);

// View chat history
console.log(history);
```

## Development vs Production

### Development
```javascript
// Mock mode for UI testing
setBackendConfig('', '', true);

// Or connect to local backend
setBackendConfig('http://localhost:8000/api', '', false);
```

### Production
```javascript
// Connect to production backend
setBackendConfig('https://api.yourdomain.com/api', 'your-api-key', false);
```

## Performance

| Mode | Calls | Typical Time |
|------|-------|--------------|
| INSTANT | 1 | < 3s |
| EXTREME | 4 (sequential) | 5-15s |
| MAX | 11 (10 parallel + synthesis) | 10-20s |

## Technical Stack

- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Storage**: Browser localStorage
- **Backend**: Any REST API (Express, Flask, Django, Go, etc)
- **LLM**: Any provider (OpenAI, Anthropic, local models, etc)

## Browser Support

- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+
- Mobile browsers (iOS Safari, Chrome Android)

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Enter | Send message |
| Shift+Enter | New line in input |
| Mode buttons | Switch between INSTANT/EXTREME/MAX |
| CLR button | Clear chat history for current mode |

## Status Indicator

Header shows backend connection status:
- `● connected` (green) - Backend is online
- `● offline (mock)` (red) - Using mock responses

## Troubleshooting

### Backend not connecting?
1. Check `setBackendConfig()` in console
2. Verify API endpoint is running
3. Check browser console (F12) for errors
4. Check network tab for failed requests
5. Ensure CORS is enabled on backend

### Messages not sending?
1. Check `/api/health` endpoint
2. Verify request format in browser console
3. Check LLM API keys on backend
4. Monitor backend logs

### Slow responses?
1. Check LLM provider rate limits
2. Optimize backend code
3. Consider streaming responses
4. Monitor network latency

## Examples

See **BACKEND_INTEGRATION.md** for complete backend examples:
- Node.js + Express + OpenAI
- Python + Flask + Claude
- Go + Gin + Local LLM
- Docker deployment
- Production setup

## License

MIT (See LICENSE file)

## Getting Help

1. Check browser console (F12 → Console)
2. Review BACKEND_INTEGRATION.md
3. Check network requests (F12 → Network)
4. Verify backend is running
5. Test endpoints with curl or Postman

---

**Ready to connect your AI backend? See BACKEND_INTEGRATION.md for complete setup!**
