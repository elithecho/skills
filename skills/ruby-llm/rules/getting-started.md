# Getting Started with RubyLLM

Start building AI apps in Ruby in 5 minutes. Chat, generate images, create embeddings - all with one gem.

## Installation

Add RubyLLM to your Gemfile:

```ruby
bundle add ruby_llm
```

## Rails Quick Setup

For Rails applications, use the generator to set up database-backed conversations:

```bash
rails generate ruby_llm:install
```

This creates Chat and Message models with ActiveRecord persistence.

### Adding a Chat UI

```bash
rails generate ruby_llm:chat_ui
```

This creates controllers, views with Turbo streaming, and background jobs. Visit `http://localhost:3000/chats` to start chatting.

## Minimal Configuration

Configure API keys for the providers you want to use:

```ruby
# config/initializers/ruby_llm.rb
require 'ruby_llm'

RubyLLM.configure do |config|
  # Add keys ONLY for the providers you intend to use
  config.openai_api_key = ENV.fetch('OPENAI_API_KEY', nil)
  # config.anthropic_api_key = ENV.fetch('ANTHROPIC_API_KEY', nil)
end
```

## Your First Chat

Interact with language models using `RubyLLM.chat`:

```ruby
# Create a chat instance (uses the configured default model)
chat = RubyLLM.chat

# Ask a question
response = chat.ask "What is Ruby on Rails?"

# The response is a RubyLLM::Message object
puts response.content
# => "Ruby on Rails, often shortened to Rails, is a server-side web application..."

# Access metadata
puts "Model Used: #{response.model_id}"
puts "Tokens Used: #{response.input_tokens} input, #{response.output_tokens} output"
```

## Generating an Image

Generate images using models like DALL-E 3 via `RubyLLM.paint`:

```ruby
# Generate an image
image = RubyLLM.paint("A photorealistic red panda coding Ruby")

# Access the image URL
if image.url
  puts image.url
  # => "https://oaidalleapiprodscus.blob.core.windows.net/..."
end

# Save the image locally
image.save("red_panda.png")
```

## Creating an Embedding

Create numerical vector representations of text:

```ruby
# Create an embedding
embedding = RubyLLM.embed("Ruby is optimized for programmer happiness.")

# Access the vector (an array of floats)
vector = embedding.vectors
puts "Vector dimension: #{vector.length}" # e.g., 1536

# Access metadata
puts "Model used: #{embedding.model}"
```

## What's Next?

- [Chat](chat.md) - Detailed chat interactions
- [Working with Models](models.md) - Choosing models, custom endpoints
- [Using Tools](tools.md) - Letting AI call your code
- [Streaming Responses](streaming.md) - Real-time responses
- [Rails Integration](rails.md) - Database persistence
- [Configuration](configuration.md) - Full configuration options
- [Error Handling](error-handling.md) - Building robust applications
