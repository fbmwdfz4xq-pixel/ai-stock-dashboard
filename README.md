# AI 股票分析面板 (AI Stock Analysis Dashboard)

## 🌐 1. 在线访问 URL
> **Live Demo:** https://ai-stock-dashboard-yourname.onrender.com

## 🛠 2. 项目完整技术栈说明
本项目采用轻量级全栈架构，确保在 Render.com 上快速部署，实现 AI 对股票数据的实时分析与存储：
* **前端 (Frontend):** 原生 HTML5 + JavaScript (Fetch API) + Tailwind CSS (通过 CDN 引入，极致轻量，无需打包构建)。
* **后端 (Backend):** Node.js + Express 框架 (提供 RESTful API，安全隐藏 API Keys，解决前后端跨域问题)。
* **AI 引擎 (LLM API):** OpenAI API (`gpt-3.5-turbo`)。
* **股票数据源:** Yahoo Finance API (通过 `yahoo-finance2` npm 包获取实时行情数据)。
* **数据存储:** Supabase (基于云端的 PostgreSQL 关系型数据库，用于持久化记录用户的查询历史和分析结果)。
* **云端部署:** Render.com (Web Service 免运维托管平台)。

---

## 🧠 3. Prompt 代码：强制 LLM 只输出严格 JSON
为了防止 LLM “乱说话”（例如输出 Markdown 标记代码块 \`\`\`json...\`\`\` 或者前置开场白导致解析崩溃），我们在代码中进行了双重限制：使用了 OpenAI 的 `response_format` 强制开启 JSON 模式，并设计了极其严格的 System Prompt。

**核心代码实现：**
```javascript
// 1. 严格的 Prompt 设计
const systemPrompt = `
You are a top-tier financial analyst. Analyze the given stock market data and provide an analysis.
CRITICAL INSTRUCTION: You MUST return ONLY a raw, valid JSON object. Do not include any conversational text, greetings, or markdown formatting (DO NOT use \`\`\`json).
The JSON MUST perfectly match this exact schema:
{
  "summary": "String (Short summary of the stock's current situation)",
  "sentiment": "Bullish" | "Neutral" | "Bearish",
  "risk_level": "High" | "Medium" | "Low"
}
`;

// 2. 调用 LLM 时的配置
const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    response_format: { type: "json_object" }, // 强制模型输出 JSON 格式
    messages: [
        { role: "system", content: systemPrompt },
        { role: "user", content: `Here is the stock data for ${ticker}: ${JSON.stringify(marketData)}` }
    ],
    temperature: 0.2 // 降低随机性，确保格式和逻辑高度稳定
});
