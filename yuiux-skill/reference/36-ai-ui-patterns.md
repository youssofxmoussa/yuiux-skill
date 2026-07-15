# 36 — AI UI Patterns

> AI interfaces have unique UX requirements. Responses are slow, outputs are probabilistic, and users need to understand what the model can and can't do. Design for uncertainty, latency, and graceful failure.

---

## Core Challenges in AI UX

```
1. Latency         — AI responses can take 2–30 seconds
2. Uncertainty     — AI can be wrong; users need to know this
3. Streaming       — output arrives token by token, not all at once
4. Context limits  — conversations have memory limits
5. Hallucination   — models confidently produce incorrect information
6. Moderation      — some inputs must be rejected
```

---

## Streaming Text UI

```tsx
// Stream tokens as they arrive — never wait for complete response
function StreamingMessage({ content, isStreaming }: { content: string; isStreaming: boolean }) {
  return (
    <div
      className={`ai-message ${isStreaming ? 'streaming' : ''}`}
      aria-live={isStreaming ? 'polite' : undefined}
      aria-atomic="false"
      aria-label={isStreaming ? 'AI is responding' : undefined}
    >
      {/* Render markdown */}
      <MarkdownRenderer content={content} />
      
      {/* Blinking cursor while streaming */}
      {isStreaming && (
        <span className="streaming-cursor" aria-hidden>▊</span>
      )}
    </div>
  );
}

// Streaming from API
async function streamChat(messages: Message[], onToken: (token: string) => void) {
  const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ messages }),
  });
  
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  if (!response.body) throw new Error('No response body');
  
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    const chunk = decoder.decode(value, { stream: true });
    // Parse SSE format
    const lines = chunk.split('\n').filter(line => line.startsWith('data: '));
    
    for (const line of lines) {
      const data = line.slice(6);
      if (data === '[DONE]') return;
      
      try {
        const { content } = JSON.parse(data);
        if (content) onToken(content);
      } catch { /* skip malformed chunks */ }
    }
  }
}

// Chat hook with streaming
function useStreamingChat() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [streamingContent, setStreamingContent] = useState('');
  const [isStreaming, setIsStreaming] = useState(false);
  const [error, setError] = useState('');
  const abortRef = useRef<AbortController | null>(null);
  
  async function sendMessage(userContent: string) {
    const userMsg: Message = { role: 'user', content: userContent };
    setMessages(prev => [...prev, userMsg]);
    setStreamingContent('');
    setIsStreaming(true);
    setError('');
    
    abortRef.current = new AbortController();
    
    try {
      let accumulated = '';
      
      await streamChat(
        [...messages, userMsg],
        (token) => {
          accumulated += token;
          setStreamingContent(accumulated);
        }
      );
      
      // Commit streamed message to history
      const aiMsg: Message = { role: 'assistant', content: accumulated };
      setMessages(prev => [...prev, aiMsg]);
      setStreamingContent('');
    } catch (err) {
      if ((err as Error).name === 'AbortError') return;
      setError('Failed to get response. Please try again.');
    } finally {
      setIsStreaming(false);
      abortRef.current = null;
    }
  }
  
  function stopStreaming() {
    abortRef.current?.abort();
  }
  
  return { messages, streamingContent, isStreaming, error, sendMessage, stopStreaming };
}
```

---

## Chat UI

```tsx
function ChatInterface() {
  const { messages, streamingContent, isStreaming, error, sendMessage, stopStreaming } = useStreamingChat();
  const [input, setInput] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);
  
  // Auto-scroll to bottom while streaming
  useEffect(() => {
    if (isStreaming) {
      messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
    }
  }, [streamingContent, isStreaming]);
  
  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const trimmed = input.trim();
    if (!trimmed || isStreaming) return;
    sendMessage(trimmed);
    setInput('');
  }
  
  return (
    <div className="chat-interface">
      {/* Message list */}
      <main
        className="chat-messages"
        role="log"
        aria-label="Conversation"
        aria-live="polite"
        aria-atomic="false"
      >
        {messages.length === 0 && <ChatEmptyState />}
        
        {messages.map((msg, i) => (
          <ChatMessage key={i} message={msg} />
        ))}
        
        {/* Streaming AI response */}
        {isStreaming && (
          <div className="chat-message assistant">
            <AIAvatar />
            <StreamingMessage content={streamingContent} isStreaming />
          </div>
        )}
        
        {/* Error state */}
        {error && (
          <div role="alert" className="chat-error">
            <AlertCircleIcon aria-hidden />
            <span>{error}</span>
            <button type="button" onClick={() => sendMessage(messages[messages.length - 1].content)}>
              Retry
            </button>
          </div>
        )}
        
        <div ref={messagesEndRef} aria-hidden />
      </main>
      
      {/* Input area */}
      <footer className="chat-input-area">
        <form onSubmit={handleSubmit} className="chat-form">
          <textarea
            value={input}
            onChange={e => setInput(e.target.value)}
            onKeyDown={e => {
              if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                handleSubmit(e as any);
              }
            }}
            placeholder="Ask anything… (Shift+Enter for new line)"
            rows={1}
            disabled={isStreaming}
            className="chat-input"
            aria-label="Message"
          />
          
          {isStreaming ? (
            <button
              type="button"
              onClick={stopStreaming}
              className="btn btn-outline btn-icon"
              aria-label="Stop generating"
            >
              <StopIcon aria-hidden />
            </button>
          ) : (
            <button
              type="submit"
              disabled={!input.trim()}
              className="btn btn-primary btn-icon"
              aria-label="Send message"
            >
              <SendIcon aria-hidden />
            </button>
          )}
        </form>
        
        <p className="chat-disclaimer">
          AI can make mistakes. Verify important information.
        </p>
      </footer>
    </div>
  );
}
```

---

## AI Response Loading States

```tsx
// Skeleton for AI-generated content — different from regular data loading
function AIThinkingSkeleton() {
  return (
    <div className="ai-thinking" role="status" aria-label="AI is thinking">
      <div className="ai-avatar" aria-hidden />
      
      {/* Animated dots */}
      <div className="thinking-dots" aria-hidden>
        <span className="thinking-dot" />
        <span className="thinking-dot" />
        <span className="thinking-dot" />
      </div>
    </div>
  );
}
```

```css
.thinking-dots {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 12px 16px;
}

.thinking-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: hsl(var(--muted-foreground));
  animation: thinking-bounce 1.4s ease-in-out infinite;
}

.thinking-dot:nth-child(2) { animation-delay: 0.2s; }
.thinking-dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes thinking-bounce {
  0%, 80%, 100% { transform: translateY(0);    opacity: 0.4; }
  40%           { transform: translateY(-8px); opacity: 1; }
}

@media (prefers-reduced-motion: reduce) {
  .thinking-dot { animation: none; opacity: 0.6; }
}
```

---

## Prompt Input UX

```tsx
// Rich prompt input with character count and suggestions
function PromptInput({ onSubmit, maxLength = 4000 }: PromptInputProps) {
  const [value, setValue] = useState('');
  const [focused, setFocused] = useState(false);
  const textareaRef = useRef<HTMLTextAreaElement>(null);
  
  // Auto-resize textarea
  useEffect(() => {
    const el = textareaRef.current;
    if (!el) return;
    el.style.height = 'auto';
    el.style.height = `${Math.min(el.scrollHeight, 200)}px`; // max 200px
  }, [value]);
  
  const remaining = maxLength - value.length;
  const nearLimit = remaining < 200;
  
  return (
    <div className={`prompt-input-wrapper ${focused ? 'focused' : ''}`}>
      <textarea
        ref={textareaRef}
        value={value}
        onChange={e => setValue(e.target.value)}
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        maxLength={maxLength}
        placeholder="Describe what you want to build…"
        className="prompt-input"
        aria-label="Prompt"
        aria-describedby="prompt-char-count"
        rows={1}
      />
      
      <div className="prompt-input-footer">
        {/* Char count — only show near limit */}
        {nearLimit && (
          <span
            id="prompt-char-count"
            className={`prompt-char-count ${remaining < 50 ? 'critical' : 'warning'}`}
            aria-live="polite"
          >
            {remaining} characters remaining
          </span>
        )}
        
        <button
          type="button"
          onClick={() => { onSubmit(value); setValue(''); }}
          disabled={!value.trim()}
          className="btn btn-primary"
        >
          Generate
        </button>
      </div>
    </div>
  );
}
```

---

## Uncertainty & Disclaimer UI

```tsx
// AI confidence indicator
function AIResponseMeta({ confidence, sources }: { confidence?: number; sources?: Source[] }) {
  return (
    <div className="ai-response-meta">
      {confidence !== undefined && confidence < 0.7 && (
        <div className="ai-uncertainty-banner" role="status">
          <InfoIcon aria-hidden />
          <span>
            This response may not be fully accurate. I'm less confident about this topic.
          </span>
        </div>
      )}
      
      {sources?.length ? (
        <details className="ai-sources">
          <summary>Sources ({sources.length})</summary>
          <ul>
            {sources.map((s, i) => (
              <li key={i}>
                <a href={s.url} target="_blank" rel="noreferrer noopener">
                  {s.title}
                </a>
              </li>
            ))}
          </ul>
        </details>
      ) : null}
      
      {/* Copy, regenerate actions */}
      <div className="ai-response-actions">
        <button type="button" onClick={copyToClipboard} aria-label="Copy response">
          <CopyIcon aria-hidden /> Copy
        </button>
        <button type="button" onClick={regenerate} aria-label="Regenerate response">
          <RefreshIcon aria-hidden /> Regenerate
        </button>
        <button type="button" onClick={thumbsUp} aria-label="Good response">
          <ThumbsUpIcon aria-hidden />
        </button>
        <button type="button" onClick={thumbsDown} aria-label="Bad response">
          <ThumbsDownIcon aria-hidden />
        </button>
      </div>
    </div>
  );
}
```

---

## AI UI Checklist

- [ ] Streaming enabled — don't wait for complete response
- [ ] Streaming cursor shows while tokens arrive
- [ ] "Stop generating" button available while streaming
- [ ] Thinking/loading state with animated dots before first token
- [ ] Auto-scroll to bottom during streaming
- [ ] Error state with retry option if stream fails
- [ ] AI disclaimer text visible near input/response
- [ ] Copy button on AI responses
- [ ] Regenerate button on AI responses
- [ ] Thumbs up/down feedback on responses
- [ ] Input disabled during streaming
- [ ] Keyboard: Enter sends, Shift+Enter adds newline
- [ ] Context limit warning when approaching token limit
- [ ] `role="log"` on message list (appropriate for chat)
- [ ] `aria-live="polite"` on streaming content area
- [ ] Screen reader: announce when AI starts and finishes responding
