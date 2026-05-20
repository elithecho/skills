# Agents

Define reusable AI assistants with class-based configuration, runtime context, and prompt conventions.

## What Are Agents?

Agents are a class-based way to define chat setup once and reuse it everywhere.

```ruby
class SupportAgent < RubyLLM::Agent
  model "gpt-5-nano"
  instructions "You are a concise support assistant."
  tools SearchDocs, LookupAccount
end

response = SupportAgent.new.ask "How do I reset my API key?"
```

Agents work in two modes:
- **Plain Ruby**: `.chat` returns `RubyLLM::Chat`
- **Rails**: `.create/.find` returns your ActiveRecord model

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat  # Activates Rails integration
  model "gpt-5-nano"
  instructions "You are a helpful assistant."
  tools SearchDocs, LookupAccount
end

chat = WorkAssistant.create!(user: current_user)
```

## Defining an Agent

```ruby
class WorkAssistant < RubyLLM::Agent
  model "gpt-5-nano"
  instructions "You are a helpful assistant."
  tools SearchDocs, LookupAccount
  temperature 0.2
  params max_output_tokens: 256
end
```

Supported macros:
- `model` - Model selection
- `tools` - Tool definitions
- `instructions` - System prompts
- `temperature` - Response creativity
- `thinking` - Extended thinking
- `params` - Provider parameters
- `headers` - HTTP headers
- `schema` - Structured output
- `context` - Configuration context
- `chat_model` - Rails-backed mode
- `inputs` - Runtime inputs

### Inline Schema

```ruby
class CriticAgent < RubyLLM::Agent
  schema do
    string :verdict, enum: ["pass", "revise"]
    string :feedback
  end
end
```

## Runtime Context and Inputs

Agents support runtime-evaluated values:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat
  inputs :workspace

  instructions { "You are helping #{workspace.name}" }
end
```

`chat` is always available:
- In `.chat` mode: `RubyLLM::Chat`
- In `.create/.find` mode: your record

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat

  instructions current_date_time: -> { Time.current.strftime("%B %d, %Y") },
    display_name: -> { chat.user.display_name_or_email }

  tools do
    [
      TodoTool.new(chat: chat),
      GoogleDriveListTool.new(user: chat.user)
    ]
  end
end
```

## Prompt Management

### Default Instructions Prompt

Calling `instructions` with no arguments enables prompt lookup:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat
  instructions
end
```

RubyLLM looks for:
- `app/prompts/work_assistant/instructions.txt.erb`

### Prompt Shorthand

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat
  instructions display_name: -> { chat.user.display_name_or_email }
end
```

### Prompt Helper

```ruby
instructions { prompt("instructions", display_name: chat.user.display_name_or_email) }
```

### Naming Conventions

- `WorkAssistant` -> `app/prompts/work_assistant/...`
- `Admin::SupportAgent` -> `app/prompts/admin/support_agent/...`

## Using an Agent

### Plain Ruby Chat

```ruby
chat = WorkAssistant.chat
response = chat.ask("Hello")
puts response.content
```

### Instance API

```ruby
agent = WorkAssistant.new
agent.ask("Hello")
```

Agent instances delegate full `RubyLLM::Chat` API:
- `model`, `messages`, `tools`, `params`, `headers`, `schema`
- `ask`, `say`, `complete`
- `add_message`, `reset_messages!`, `each`
- `with_tool`, `with_tools`, `with_model`, `with_temperature`
- `on_new_message`, `on_end_message`, `on_tool_call`, `on_tool_result`

Access underlying chat via `agent.chat`.

## Rails-Backed Agents

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat
  model "gpt-5-nano"
  instructions "You are a helpful assistant."
  tools SearchDocs, LookupAccount
end
```

```ruby
# Create persisted chat with agent config
chat = WorkAssistant.create!(user: current_user)

# Load existing chat (runtime config only)
chat = WorkAssistant.find(params[:id])

# Explicitly persist instructions
WorkAssistant.sync_instructions!(chat)
```

Instruction persistence:
- `create/create!`: Applies and persists instructions
- `find`: Applies instructions at runtime only
- `sync_instructions!`: Explicitly persists

## When to Use Agents vs RubyLLM.chat

Use `RubyLLM.chat` for one-off conversations:

```ruby
chat = RubyLLM.chat(model: "gpt-5-nano")
chat.with_instructions("Explain this clearly.")
```

Use agents for reusable behavior:

```ruby
class WorkAssistant < RubyLLM::Agent
  model "gpt-5-nano"
  instructions "You are a helpful assistant."
  tools SearchDocs, LookupAccount
end

WorkAssistant.new.ask("Help me find docs.")
```

Think of `RubyLLM.chat` as ad-hoc and `RubyLLM::Agent` as reusable architecture.
