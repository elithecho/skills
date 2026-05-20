# Model Registry

Access hundreds of AI models from all major providers with one simple API.

## The Model Registry

RubyLLM maintains an internal registry of known AI models with information:
- **id**: Provider's unique identifier
- **provider**: Source provider (openai, anthropic, etc.)
- **type**: Model function (chat, embedding, etc.)
- **name**: Human-friendly name
- **context_window**: Max input tokens
- **max_tokens**: Max output tokens
- **supports_vision**: Image processing capability
- **supports_functions**: Tool calling support
- **input_price_per_million**: Cost per 1M input tokens
- **output_price_per_million**: Cost per 1M output tokens
- **family**: Broader classification

## Refreshing the Registry

```ruby
# Refresh in-memory registry
RubyLLM.models.refresh!

# Save to disk (if using custom location)
RubyLLM.models.save_to_json

# Only remote providers (exclude local like Ollama)
RubyLLM.models.refresh!(remote_only: true)
```

In Rails:

```ruby
rails db:migrate
Model.refresh!  # If using database registry
```

## Exploring Models

### Listing and Filtering

```ruby
# All models
all_models = RubyLLM.models.all

# By type
chat_models = RubyLLM.models.chat_models
embedding_models = RubyLLM.models.embedding_models

# By provider
openai_models = RubyLLM.models.by_provider(:openai)

# By family
claude_family = RubyLLM.models.by_family('claude3_sonnet')

# Chain filters
openai_vision = RubyLLM.models.by_provider(:openai).select(&:supports_vision?)
```

### Finding a Specific Model

```ruby
model_info = RubyLLM.models.find('gpt-5.2')

if model_info
  puts "Model: #{model_info.name}"
  puts "Provider: #{model_info.provider}"
  puts "Context Window: #{model_info.context_window} tokens"
  puts "Supports Vision: #{model_info.supports_vision?}"
  puts "Input Price: $#{model_info.input_price_per_million}/M tokens"
end
```

### Model Aliases

RubyLLM uses aliases for convenience:

```ruby
# 'claude-sonnet-4-5' resolves to actual version
chat = RubyLLM.chat(model: 'claude-sonnet-4-5')
puts chat.model.id  # => "claude-3-5-sonnet-20241022"
```

### Provider-Specific Resolution

```ruby
# Get Claude from specific provider
model_anthropic = RubyLLM.models.find('claude-sonnet-4-5', :anthropic)
model_bedrock = RubyLLM.models.find('claude-sonnet-4-5', :bedrock)
```

## Custom Endpoints & Unlisted Models

### Custom OpenAI API Base URL

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = ENV['AZURE_OPENAI_KEY']
  config.openai_api_base = "https://YOUR_AZURE_RESOURCE.openai.azure.com"
end
```

### Assuming Model Existence

For models not in registry:

```ruby
# Custom deployment name
chat = RubyLLM.chat(
  model: 'my-company-gpt4o',
  provider: :openai,
  assume_model_exists: true
)

# Custom embedding model
embedding = RubyLLM.embed(
  "Test text",
  model: 'my-custom-embedder',
  provider: :openai,
  assume_model_exists: true
)

# Custom image model
image = RubyLLM.paint(
  "A beautiful landscape",
  model: 'my-custom-dalle',
  provider: :openai,
  assume_model_exists: true
)
```

Key points:
- `provider:` is mandatory
- No validation performed
- Capability checks bypassed
- Warning logged

## Database Model Registry (Rails)

When using `ruby_llm:install` generator:

```ruby
# Model is automatically associated
chat = Chat.create!(model: 'gpt-5.2')
chat.model  # => #<Model model_id: "gpt-5.2", ...>
chat.model.context_window  # => 128000
chat.model.supports_vision  # => true

# Query by model attributes
Chat.joins(:model).where(models: { provider: 'anthropic' })
Model.where(supports_functions: true)
```

Refresh models in Rails:

```ruby
Model.refresh!
```
