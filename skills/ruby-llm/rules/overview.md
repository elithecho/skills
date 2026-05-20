# RubyLLM Overview

Understand how RubyLLM works and how its components fit together.

## Core Components

### Chat

The Chat component is the primary interface for conversational AI:

```ruby
chat = RubyLLM.chat(model: "gpt-5-nano")
```

The chat object maintains conversation history, handles message formatting, and manages the request/response cycle.

### Messages

Messages are the fundamental unit of conversation. Each message has a role (user, assistant, system, or tool) and content:

```ruby
response = chat.ask("What is Ruby?")
# Creates a user message, sends it, and returns an assistant message
```

Messages can include various types of content - text, images, documents - depending on model capabilities.

### Tools

Tools allow AI models to call Ruby code during conversations:

```ruby
class Calculator < RubyLLM::Tool
  description "Performs basic arithmetic"
  param :expression, desc: "Mathematical expression to evaluate"

  def execute(expression:)
    { result: eval(expression) }
  end
end
```

### Providers

Providers are adapters connecting RubyLLM to specific AI services. Each provider implements the same interface while handling unique requirements.

### Configuration

Configuration works at three levels:

```ruby
# Global configuration
RubyLLM.configure do |config|
  config.openai_api_key = ENV["OPENAI_API_KEY"]
  config.default_model = "gpt-5-nano"
end

# Context configuration - isolated scope
context = RubyLLM.context do |config|
  config.openai_api_key = tenant.api_key
  config.default_model = "gpt-5.2"
end
chat = context.chat

# Instance configuration
chat = RubyLLM.chat(model: "claude-opus-4-5", temperature: 0.7)
```

## Design Principles

### Provider Agnostic

The framework treats all AI providers equally. Whether using OpenAI, Anthropic, or local models via Ollama, the code looks the same.

### Progressive Disclosure

Simple things should be simple, complex things should be possible:

```ruby
# Simple
response = RubyLLM.chat.ask("Hello")

# Advanced
chat = RubyLLM.chat(model: "gpt-5-nano", temperature: 0.2)
  .with_instructions("You are a helpful assistant")
  .with_tool(DatabaseQuery)
  .with_schema(ResponseFormat)
```

### Ruby Conventions

The framework follows Ruby idioms - descriptive method names, block-based configuration, and consistent error handling.

### Minimal Dependencies

RubyLLM depends only on:
- Faraday (HTTP)
- Zeitwerk (autoloading)
- Marcel (file type detection)

## How Providers Work

### Provider Detection

RubyLLM automatically determines which provider to use based on the model:

```ruby
# Automatic detection
chat = RubyLLM.chat(model: "gpt-5-nano")  # Uses OpenAI

# Explicit provider
chat = RubyLLM.chat(model: "llama-3", provider: :ollama)
```

### Capability Management

Different models have different capabilities. RubyLLM tracks these:

```ruby
model_info = RubyLLM.models.find("gpt-5.2")
puts model_info.capabilities
# => [:chat, :vision, :tools, :json_mode]
```

### Response Normalization

Each provider returns responses in different formats. RubyLLM normalizes these into consistent response objects.

## Rails Integration

RubyLLM integrates deeply with Rails through ActiveRecord mixins:

```ruby
class Conversation < ApplicationRecord
  acts_as_chat
end

conversation = Conversation.create!(model: "gpt-5-nano")
response = conversation.ask("How can I help you today?")
```

## Next Steps

1. Complete the [Getting Started](getting-started.md) guide
2. Learn about [Chat](chat.md) for conversational features
3. Explore [Tools](tools.md) to give AI access to your code
4. For Rails developers, see the [Rails Integration](rails.md) guide
