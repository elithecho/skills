# Extended Thinking

Give reasoning models more time and budget to deliberate, with optional access to thinking output.

## What is Extended Thinking?

Extended Thinking gives supported models more time and computation budget to deliberate before answering. It can improve results on multi-step tasks like coding, math, and logic, at the expense of latency and cost.

## Controlling Extended Thinking

Use `with_thinking` to control models that support thinking:

```ruby
chat = RubyLLM.chat(model: 'claude-opus-4.5')
  .with_thinking(effort: :high, budget: 8000)

response = chat.ask("What is 15 * 23?")

response.thinking&.text
response.thinking&.signature
response.content
```

`with_thinking` requires at least one of `effort` or `budget`:

```ruby
chat.with_thinking(effort: :low)
chat.with_thinking(budget: 10_000)
chat.with_thinking(effort: :none)
```

## Effort and Budget

- `effort`: Qualitative depth (`:low`, `:medium`, `:high`)
- `budget`: Token cap for models that accept it

RubyLLM sends `effort` and `budget` exactly as provided. Check your provider's docs for supported values.

## Streaming with Thinking

Thinking content is delivered alongside normal content:

```ruby
chat = RubyLLM.chat(model: 'claude-opus-4.5')
  .with_thinking(effort: :medium)

chat.ask("Solve step by step: What is 127 * 43?") do |chunk|
  print chunk.thinking&.text
  print chunk.content
end
```

Some providers only expose thinking in the final response. In those cases:
- `response.thinking` is populated after stream completes
- `chunk.thinking` stays empty

## ActiveRecord Integration

When using `acts_as_chat` and `acts_as_message`, thinking output is persisted:

```ruby
# Migration:
# t.text :thinking_text
# t.text :thinking_signature
# t.integer :thinking_tokens

response = chat_record.ask("Explain quantum entanglement")
response.thinking&.text
response.thinking_tokens
```

### Manual Migration

```ruby
class AddThinkingToMessages < ActiveRecord::Migration[7.1]
  def change
    add_column :messages, :thinking_text, :text
    add_column :messages, :thinking_signature, :text
    add_column :messages, :thinking_tokens, :integer
    add_column :tool_calls, :thought_signature, :string
  end
end
```

## Provider Notes

- **Claude**: Uses thinking budget, returns text and signature
- **Anthropic**: Requires thinking budget
- **Bedrock**: Params are model-dependent
- **Gemini 2.5**: Uses token budget; Gemini 3 uses effort levels
- **OpenAI**: Accepts `effort` but may not return thinking
- **Perplexity**: Streams `<think>` blocks inside content
- **Mistral Magistral**: Always thinks, ignores `with_thinking` params
- **Ollama Qwen3**: Thinks by default, only accepts `effort: :none` to disable
- **Anthropic/Ollama**: Currently do not report thinking token counts
