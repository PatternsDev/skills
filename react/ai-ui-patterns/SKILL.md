---
name: ai-ui-patterns
description: Design patterns for building AI-powered interfaces like chatbots and intelligent assistants in React.
context: fork
allowed-tools: Read, Grep, Glob
paths:
  - "**/*.tsx"
  - "**/*.jsx"
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
related_skills:
  - "hooks-pattern"
  - "hoc-pattern"
---

# AI UI Patterns

Building AI-powered interfaces – from chatbots to intelligent assistants – requires careful integration of backend AI services with reactive UI components. In this chapter, we explore design patterns in React for such interfaces, focusing on **two implementations**: a plain React app (using Vite) and a Next.js app. We'll use **OpenAI's API** (via the Vercel AI SDK) as our AI engine, and TailwindCSS for styling. Key topics include prompt management, streaming responses, input debouncing, error handling, and how these patterns differ between Vite and Next.js. We also highlight reusable component patterns and **Vercel's AI UI components (AI Elements)** for building polished chat UIs.

## When to Use

- Use this when building conversational AI interfaces that stream responses from LLMs
- This is helpful for integrating OpenAI, Anthropic, or other AI providers into React applications
- Use this when you need patterns for prompt management, streaming, error handling, and AI-specific UI

## Instructions

- Use the Vercel AI SDK's `useChat` hook for managing conversation state and streaming responses
- Keep API keys on the server — use Next.js API routes or a separate backend for AI calls
- Enable streaming (`stream: true`) for responsive real-time output in chat interfaces
- Debounce input for autocomplete features; disable input during response streaming for chat
- Build reusable components (ChatMessage, InputBox) decoupled from data-fetching logic

## Details

> **Note:** While this article uses OpenAI as an example, the Vercel AI SDK supports multiple model providers including **Gemini**, **OpenAI**, and **Anthropic**. You can easily swap between providers through the SDK's unified interface – we're just choosing one option for demonstration purposes.

### Introduction: AI Interfaces in React

AI-driven user interfaces (UIs) have become popular with the rise of LLMs like ChatGPT. Unlike traditional UIs, AI interfaces often involve conversational interactions, dynamic content streaming, and asynchronous backend calls. This introduces unique challenges and patterns for React developers. A typical AI chat interface consists of a **frontend** (for user input and displaying responses) and a **backend** (to call the AI model). The backend is essential to keep API keys and heavy processing off the client for security and performance. Tools like Vercel's **AI SDK** make it easier to connect to providers (OpenAI, HuggingFace, etc.) and stream responses in real-time. We'll explore how to set up both a Next.js app and a Vite (React) app to handle these concerns, and discuss best practices that apply to both.

**Key patterns covered:**

- Structuring AI prompt data and managing conversation state
- Streaming AI responses to the UI for real-time feedback
- Debouncing user input to avoid spamming the API
- Error handling and fallbacks in the UX
- Reusable UI components for messages, inputs, and more (with TailwindCSS)
- Architectural differences: Next.js route handlers vs. Vite with a Node backend

By the end, you'll be equipped to build a responsive, robust AI-powered UI in React, whether you prefer Next.js or a Vite toolchain.

### Project Setup and Tools

Before diving into code, ensure you have the necessary packages and configurations:

- **React & Vite:** Initialize a Vite + React project (e.g. `npm create vite@latest my-ai-app -- --template react`). For Next.js, you can use `npx create-next-app` or the Next 13 App Router templates. Both will work – we'll highlight differences as we go.

- **TailwindCSS:** Set up Tailwind in your project for quick styling.

- **OpenAI API & Vercel AI SDK:** Install OpenAI's library or the Vercel AI SDK. We will use **Vercel's AI SDK** (`npm i ai`) which provides helpful React hooks (`useChat`, `useCompletion`) and server utilities. This SDK is framework-agnostic, working with Next.js, vanilla React, Svelte, and more. It simplifies streaming and state management, and is free/open-source.

- **API Keys:** Get your OpenAI API key from the OpenAI dashboard and store it safely. In Next.js, put it in `.env.local` (e.g. `OPENAI_API_KEY=sk-...`) and never commit it. In a Vite app, **do not** expose the key in client code – instead, use a backend proxy or environment variable on the server.

### Setting Up AI Endpoints (Next.js vs. Vite)

**Next.js Implementation:** Next.js allows us to create **route handlers** as serverless functions. We can define an API route that the React front-end will call for AI responses:

```typescript
// app/api/chat/route.ts (Next.js)
import { Configuration, OpenAIApi } from 'openai-edge';
import { OpenAIStream, StreamingTextResponse } from 'ai';

export const runtime = 'edge';

const config = new Configuration({ apiKey: process.env.OPENAI_API_KEY });
const openai = new OpenAIApi(config);

export async function POST(req: Request) {
  const { messages } = await req.json();
  const response = await openai.createChatCompletion({
    model: 'gpt-3.5-turbo',
    stream: true,
    messages: messages.map((m: any) => ({ role: m.role, content: m.content }))
  });
  const stream = OpenAIStream(response);
  return new StreamingTextResponse(stream);
}
```

In this handler, we receive a JSON body containing an array of messages (chat history). We call OpenAI's chat completion with `stream: true` to get a streaming response. We then wrap the response in a `StreamingTextResponse` provided by the AI SDK to pipe it back to the client in chunks. The Next.js API route keeps our API key on the server and streams data efficiently.

**Vite (React) Implementation:** In a Vite app, there's no built-in server, so we need to create our own backend for the OpenAI calls. This can be a simple Node/Express server:

```javascript
// backend/server.js (Node/Express for Vite app)
import express from 'express';
import { Configuration, OpenAIApi } from 'openai';

const app = express();
app.use(express.json());

const config = new Configuration({ apiKey: process.env.OPENAI_API_KEY });
const openai = new OpenAIApi(config);

app.post('/api/chat', async (req, res) => {
  try {
    const { messages = [] } = req.body;
    const systemMsg = { role: 'system', content: 'You are a helpful assistant.' };
    const inputMessages = [systemMsg, ...messages];
    const response = await openai.createChatCompletion({
      model: 'gpt-3.5-turbo',
      stream: false,
      messages: inputMessages
    });
    const content = response.data.choices[0].message?.content;
    res.json({ content });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

app.listen(6000, () => console.log('API server listening on http://localhost:6000'));
```

During development, you can configure the Vite dev server to proxy `/api` calls to this backend (e.g. in `vite.config.js`, set `server.proxy['/api'] = 'http://localhost:6000'`). The key is that the React app calls a **relative `/api/chat` endpoint**, which the proxy/hosting will route to your server code. This keeps the OpenAI key hidden.

**Enabling Streaming in Node:** The above Express example returns the full response after completion (`stream: false` for simplicity). To stream in Node, you can use OpenAI's HTTP stream: set `stream: true` and handle the response as a stream of data. This involves reading the `response.data` stream and flushing chunks to the client with `res.write()`. If you choose to stick with full responses (no streaming), the UI patterns still largely apply – but streaming greatly improves UX.

### Prompt Handling and Conversation State

At the heart of any AI interface is **prompt management** – assembling user input (and context) into a prompt or message sequence for the AI model. In a chat scenario, we maintain a list of messages, each with a role and content. OpenAI's Chat API expects messages in the format `{ role: 'user' | 'assistant' | 'system', content: string }`. We typically start with a system message (to set the assistant's behavior or context), followed by alternating user and assistant messages as the conversation progresses.

**State management in React:** We can store the conversation in component state. Using the Vercel SDK's React hook:

```jsx
import { useChat } from 'ai/react';

function ChatInterface() {
  const { messages, input, handleInputChange, handleSubmit } = useChat();
  // ...
}
```

The `useChat` hook handles a lot for us: it manages the `messages` state (an array of message objects), an `input` state for the current text input, and provides `handleInputChange` and `handleSubmit` helpers. By default, `useChat()` will POST to `/api/chat` when you submit.

**Manual state handling:** If you aren't using `useChat`, you can manage state with `useState` or context. On form submit, call your API and then update the messages array by appending the user query and the assistant's response.

**System prompts and context:** A common pattern is including an initial system message describing the assistant's role or knowledge base. For example, if building a docs helper, system content might be "You are a documentation assistant. Answer with examples from the docs."

**Single-turn vs multi-turn:** If your interface is a single question answering (no conversation memory), you could use the `useCompletion` hook from the Vercel SDK instead. For chatbots and multi-turn dialogs, `useChat` is the go-to pattern, since it retains and sends the message history on each request.


### Streaming AI Responses to the UI

One hallmark of modern AI UI is **streaming output**: as the AI generates tokens, the user sees the reply appearing in real-time. This is crucial for better UX because model-generated answers can be lengthy or slow. Instead of waiting many seconds in silence, streaming lets us display partial results immediately.

**How streaming works:** When we enabled `stream: true` on the OpenAI API, the response is sent as a sequence of chunks (data events) rather than one JSON blob. The Vercel AI SDK simplifies consumption of these chunks. On the server, we turned the response into a text stream (`StreamingTextResponse`). On the client side, the `useChat` hook handles reading this stream and updating the messages state incrementally as new text arrives.

If you implement streaming manually in React (without the SDK), you would do something like:

```javascript
const res = await fetch('/api/chat', { method: 'POST', body: JSON.stringify({ messages }) });
const reader = res.body.getReader();
const decoder = new TextDecoder();
let partial = "";
while(true) {
  const { value, done } = await reader.read();
  if (done) break;
  partial += decoder.decode(value);
  setAssistantMessage(partial);
}
```

**Auto-scrolling:** One UX detail when streaming is ensuring the latest message is visible. A pattern to handle this is auto-scrolling the message container on update with a `useEffect` watching the messages array length.

**Partial rendering and completion:** Show a visual indicator during streaming – for example, a blinking cursor or "AI is typing…" message. Once the stream finishes, finalize the message display.


### Input Handling and Debouncing

For chat interactions, you usually send the query when the user submits the form. In some AI applications, however, you might want to react to input continuously – for example, **autocomplete suggestions** or **real-time validation by AI**. In such cases, **debouncing** is important.

**Why debounce?** Calling the OpenAI API on every keystroke would be extremely inefficient and costly. Debouncing delays the API call until the user has stopped typing for a short period.

```jsx
const [draft, setDraft] = useState("");

useEffect(() => {
  if (!draft) return;
  const timeout = setTimeout(() => {
    getSuggestion(draft);
  }, 500);
  return () => clearTimeout(timeout);
}, [draft]);
```

For a simple chatbot with explicit "send" action, debouncing is usually not needed – you send when the user hits Enter. However, it's still useful to **disable the input** or **prevent multiple submissions** while an AI response is in progress.


### Error Handling and Resilience

Robust error handling is vital in AI applications:

- **Try/Catch around API calls:** On the server, wrap the OpenAI call in try/catch. Return a proper error response if something fails.

- **Client-side error state:** Handle cases where the response indicates an error.

```javascript
try {
  await sendMessage({ text: input });
} catch (error) {
  console.error("Failed to send message:", error);
}
```

- **User feedback:** Always inform the user when something goes wrong. Display the error inline in the chat – e.g., as a special "system" message saying "*Sorry, something went wrong. Please try again.*"

- **Retry mechanism:** Consider allowing the user to retry with a "Try again" button.

- **Validation errors:** Validate on the client before calling the API. Disable send on empty input, or truncate inputs that exceed some length.


### Building the UI: Components and Styling Patterns

**Chat message components:** Create a `ChatMessage` component that renders a single message bubble. Based on role, style it differently:

```jsx
function ChatMessage({ role, content }) {
  const isUser = role === 'user';
  return (
    <div className={`flex ${isUser ? 'justify-end' : 'justify-start'} mb-2`}>
      <div className={`max-w-xl px-4 py-2 rounded-lg ${
        isUser ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-900'
      }`}>
        {content}
      </div>
    </div>
  );
}
```

**Input component:**

```jsx
function InputBox({ value, onChange, onSubmit, disabled }) {
  return (
    <form onSubmit={onSubmit} className="flex gap-2">
      <input
        type="text"
        value={value}
        onChange={onChange}
        disabled={disabled}
        className="flex-1 border rounded px-3 py-2"
        placeholder="Type your message..."
      />
      <button type="submit" disabled={disabled} className="bg-blue-500 text-white px-4 py-2 rounded">
        Send
      </button>
    </form>
  );
}
```

**Composition:**

```jsx
function ChatInterface() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat();
  
  return (
    <div className="flex flex-col h-screen max-w-2xl mx-auto p-4">
      <div className="flex-1 overflow-y-auto">
        {messages.map((msg, i) => (
          <ChatMessage key={i} role={msg.role} content={msg.content} />
        ))}
      </div>
      <InputBox 
        value={input} 
        onChange={handleInputChange} 
        onSubmit={handleSubmit}
        disabled={isLoading}
      />
    </div>
  );
}
```

This separation of concerns makes it easy to test and swap UI parts. The logic (`useChat`) is decoupled from the display components.


### Vercel AI Elements (Pre-Built Chat UI Components)

Vercel's **AI Elements** library offers a set of ready-made React components specifically designed for AI chat interfaces:

- **Conversation**: A container that renders a list of messages with auto-scrolling.
- **Prompt**: An input component optimized for chat prompts.
- **TypingIndicator**: Shows when the AI is "thinking" or streaming a response.
- **ErrorBoundary/ErrorMessage**: Handle and display errors gracefully.

```jsx
import { Conversation, Prompt, TypingIndicator } from '@vercel/ai-elements';

function ChatApp() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat();
  
  return (
    <div className="h-screen flex flex-col">
      <Conversation messages={messages} />
      {isLoading && <TypingIndicator />}
      <Prompt 
        value={input} 
        onChange={handleInputChange} 
        onSubmit={handleSubmit}
      />
    </div>
  );
}
```

### Putting It All Together

1. **Backend API Route:** Whether using Next.js route handlers or a separate Express server, create an endpoint that receives messages, calls the AI model, and streams the response back.

2. **State Management:** Use the Vercel AI SDK's `useChat` hook (or roll your own with `useState`) to manage the conversation state.

3. **Streaming:** Enable streaming on both server and client for responsive UX.

4. **Debouncing & Rate Limiting:** For features like autocomplete, debounce API calls. For chat, disable input during response streaming.

5. **Error Handling:** Wrap API calls in try/catch, provide user feedback on errors, and consider retry mechanisms.

6. **Reusable Components:** Build presentational components (`ChatMessage`, `InputBox`) that are decoupled from data-fetching logic. Consider using AI Elements for production-ready components.

7. **Styling:** Use TailwindCSS (or your preferred styling solution) to create a clean, responsive chat interface.

### Architectural Comparison: Next.js vs. Vite

| Aspect | Next.js | Vite + Node Backend |
|--------|---------|---------------------|
| **API Routes** | Built-in (`pages/api/` or `app/api/`) | Separate Express/Node server required |
| **Streaming** | Native support with Edge Runtime | Manual implementation with `res.write()` |
| **Deployment** | Vercel (optimized) or self-host | Deploy frontend (static) + backend separately |
| **Complexity** | Lower (all-in-one) | Higher (two codebases) |
| **Flexibility** | Framework conventions | Full control |

For most AI chat applications, **Next.js** provides a simpler developer experience with its integrated API routes and streaming support. However, if you have an existing Vite/React app or prefer more control, the patterns described here work well with a separate backend.

## Source

- [patterns.dev/react/ai-ui-patterns](https://patterns.dev/react/ai-ui-patterns)

### References

- [Vercel AI SDK Documentation](https://ai-sdk.dev/)
- [Vercel Academy: Basic Chatbot](https://vercel.com/academy/ai-sdk/basic-chatbot)
- [Vercel Academy: AI Elements](https://vercel.com/academy/ai-sdk/ai-elements)
- [Building Chatbot in Next.js using Vercel AI SDK](https://blog.saeloun.com/2023/07/13/building-chatbot-in-next-js-using-vercel-ai-sdk/)
- [Build a Docs-Aware Chatbot with React, Vite, Node and OpenAI](https://dev.to/cloudinary/build-a-docs-aware-chatbot-with-react-vite-node-and-openai-plus-fun-dalle-avatars-1chi)
