# 🚀 TENSORA Backend Quick Start

**5-Minute Setup Guide**

## What You Have

| Mode | Provider | Speed | Cost | Best For |
|------|----------|-------|------|----------|
| **INSTANT** ⚡ | Groq | < 2s | FREE | Quick responses |
| **EXTREME** 🟦 | Qwen | 3-8s | Cheap | Balanced reasoning |
| **MAX** 🧠 | OpenAI | 5-15s | $$ | Complex tasks |

---

## Step 1️⃣: Get API Keys (2 min)

### Option A: Groq (Required - FASTEST & FREE!)
```bash
1. Go to: https://console.groq.com/keys
2. Sign up / Login
3. Create API key
4. Copy the key starting with "gsk_"
```

### Option B: Qwen (Required - AFFORDABLE)
```bash
1. Go to: https://dashscope.aliyuncs.com/
2. Sign up with Alibaba account
3. Create API key
4. Copy the key starting with "sk_"
```

### Option C: OpenAI (Required - MOST CAPABLE)
```bash
1. Go to: https://platform.openai.com/api-keys
2. Create new secret key
3. Copy the key starting with "sk-"
```

---

## Step 2️⃣: Configure Backend (1 min)

### Edit `.env` file:
```bash
# Copy .env file content below and edit with YOUR keys

GROQ_API_KEY=gsk_YOUR_KEY_HERE
QWEN_API_KEY=sk_YOUR_KEY_HERE
OPENAI_API_KEY=sk-YOUR_KEY_HERE

PORT=8000
NODE_ENV=development
```

---

## Step 3️⃣: Install & Start (1 min)

```bash
# Install dependencies
npm install

# Start backend
npm start

# You should see:
# ╔════════════════════════════════════════════════════════════╗
# ║          ◢ TENSORA BACKEND RUNNING ◢                      ║
# ║  🚀 Server: http://localhost:8000                         ║
# ║  ⚡ INSTANT mode:  Groq (mixtral-8x7b)                    ║
# ║  🟦 EXTREME mode:  Qwen (qwen-turbo)                      ║
# ║  🧠 MAX mode:      OpenAI (gpt-4-turbo)                   ║
# ╚════════════════════════════════════════════════════════════╝
```

---

## Step 4️⃣: Connect Frontend (1 min)

**In your browser console (F12):**

```javascript
// Connect to your backend
setBackendConfig('http://localhost:8000/api', '', false);
```

You should see header change to: **● connected**

---

## Step 5️⃣: Test It! 🎉

1. Select **INSTANT** mode
2. Type: "What is AI?"
3. Hit **SEND**

You should get response in < 2 seconds! ⚡

---

## How Each Mode Works

### ⚡ INSTANT (Groq - Fastest)
```
User: "Explain quantum computing"
↓
Groq (mixtral-8x7b) processes
↓
< 2 seconds response
```

### 🟦 EXTREME (Qwen - Balanced)
```
User: "Write a business proposal for SaaS"
↓
Qwen PLANNER: Plans structure
↓
Qwen WRITER: Drafts content
↓
Qwen REVIEWER: Critiques
↓
Qwen FINALIZER: Refines
↓
8 seconds, polished response
```

### 🧠 MAX (OpenAI - Comprehensive)
```
User: "Design a recommendation system"
↓
10 agents run in PARALLEL:
├─ RESEARCH: Best practices
├─ CODING: Architecture
├─ REVIEW: Quality checks
├─ OPTIMIZE: Performance
├─ TESTING: Test strategy
├─ UI: Interface design
├─ LOGIC: Reasoning validation
├─ DATA: Data modeling
├─ DEBUG: Edge cases
└─ DEPLOY: Deployment plan
↓
SYNTH: Combines all insights
↓
15 seconds, comprehensive solution
```

---

## Troubleshooting

### "Cannot find module 'groq-sdk'"
```bash
npm install groq-sdk openai @anthropic-ai/sdk express cors dotenv
```

### "API key not found"
```bash
# Make sure .env file has your keys
cat .env

# Or set manually:
export GROQ_API_KEY=gsk_...
export QWEN_API_KEY=sk_...
export OPENAI_API_KEY=sk-...

npm start
```

### Backend not responding
```bash
# Check it's running
curl http://localhost:8000/api/health

# Should return:
# {"status":"ok","timestamp":"...","version":"1.0.0"}
```

### Frontend still shows "offline"
```javascript
// In browser console:
console.log(CONFIG);
// Make sure API_URL is: http://localhost:8000/api
```

---

## Pricing & Costs

| Provider | Model | Cost | Free Tier |
|----------|-------|------|-----------|
| **Groq** | mixtral-8x7b | FREE | ∞ (free tier) |
| **Qwen** | qwen-turbo | $0.0008/1K tokens | $1 credit |
| **OpenAI** | gpt-4-turbo | $0.01/1K input tokens | Trial $5 |

---

## Production Deployment

### Using Docker
```bash
docker build -t tensora-backend .
docker run -p 8000:8000 \
  -e GROQ_API_KEY=gsk_... \
  -e QWEN_API_KEY=sk_... \
  -e OPENAI_API_KEY=sk-... \
  tensora-backend
```

### Using Railway/Render/Heroku
1. Push repo to GitHub
2. Connect repo to deployment service
3. Add env vars in dashboard
4. Deploy!

---

## Next Steps

- ✅ Test each mode (INSTANT, EXTREME, MAX)
- ✅ Try different prompts
- ✅ Monitor response times
- ✅ Deploy to production
- ✅ Share with friends!

---

**🎉 YOU'RE ALL SET! START YOUR BACKEND AND BEGIN TENSORING! 🚀**
