# RubyLLM Configuration

Configure once, use everywhere. API keys, defaults, timeouts, and multi-tenant contexts.

## Quick Start

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = ENV['OPENAI_API_KEY']
  config.anthropic_api_key = ENV['ANTHROPIC_API_KEY']
end
```

That's it. RubyLLM uses sensible defaults for everything else.

## Provider Configuration

### API Keys

Configure API keys only for the providers you use:

```ruby
RubyLLM.configure do |config|
  # Remote providers
  config.openai_api_key = ENV['OPENAI_API_KEY']
  config.anthropic_api_key = ENV['ANTHROPIC_API_KEY']
  config.gemini_api_key = ENV['GEMINI_API_KEY']
  config.vertexai_project_id = ENV['GOOGLE_CLOUD_PROJECT']
  config.vertexai_location = ENV['GOOGLE_CLOUD_LOCATION']
  config.deepseek_api_key = ENV['DEEPSEEK_API_KEY']
  config.mistral_api_key = ENV['MISTRAL_API_KEY']
  config.perplexity_api_key = ENV['PERPLEXITY_API_KEY']
  config.openrouter_api_key = ENV['OPENROUTER_API_KEY']
  config.xai_api_key = ENV['XAI_API_KEY']

  # Local providers
  config.ollama_api_base = 'http://localhost:11434/v1'
  config.gpustack_api_base = ENV['GPUSTACK_API_BASE']

  # AWS Bedrock
  config.bedrock_api_key = ENV['AWS_ACCESS_KEY_ID']
  config.bedrock_secret_key = ENV['AWS_SECRET_ACCESS_KEY']
  config.bedrock_region = ENV['AWS_REGION']

  # Azure
  config.azure_api_base = ENV['AZURE_API_BASE']
  config.azure_api_key = ENV['AZURE_API_KEY']
end
```

### OpenAI Organization & Project Headers

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = ENV['OPENAI_API_KEY']
  config.openai_organization_id = ENV['OPENAI_ORG_ID']
  config.openai_project_id = ENV['OPENAI_PROJECT_ID']
end
```

## Custom Endpoints

### OpenAI-Compatible APIs

Connect to any OpenAI-compatible API:

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = ENV['CUSTOM_API_KEY']
  config.openai_api_base = "http://localhost:8080/v1"
end

chat = RubyLLM.chat(model: 'my-custom-model', provider: :openai, assume_model_exists: true)
```

### System Role Compatibility

```ruby
RubyLLM.configure do |config|
  # For servers that require 'system' role
  config.openai_use_system_role = true
  config.openai_api_base = "http://localhost:11434/v1"
end
```

## Default Models

```ruby
RubyLLM.configure do |config|
  config.default_model = 'claude-sonnet-4-5'
  config.default_embedding_model = 'text-embedding-3-large'
  config.default_image_model = 'dall-e-3'
end
```

Defaults if not configured:
- Chat: `gpt-5-nano`
- Embeddings: `text-embedding-3-small`
- Images: `gpt-image-1`

## Connection Settings

### Timeouts & Retries

```ruby
RubyLLM.configure do |config|
  config.request_timeout = 120        # Seconds (default: 120)
  config.max_retries = 3             # Retry attempts (default: 3)
  config.retry_interval = 0.1         # Initial retry delay
  config.retry_backoff_factor = 2    # Exponential backoff
end
```

### HTTP Proxy Support

```ruby
RubyLLM.configure do |config|
  config.http_proxy = "http://proxy.company.com:8080"
  # Or with auth:
  config.http_proxy = "http://user:pass@proxy.company.com:8080"
end
```

## Logging & Debugging

### Basic Logging

```ruby
RubyLLM.configure do |config|
  config.log_file = '/var/log/ruby_llm.log'
  config.log_level = :info  # :debug, :info, :warn

  # Or use Rails logger
  config.logger = Rails.logger
end
```

### Debug Options

```ruby
RubyLLM.configure do |config|
  config.log_level = :debug if ENV['RUBYLLM_DEBUG'] == 'true'
  config.log_stream_debug = true
end
```

## Contexts: Isolated Configurations

Create temporary configuration scopes:

```ruby
# Global config
RubyLLM.configure do |config|
  config.openai_api_key = ENV['OPENAI_PROD_KEY']
end

# Isolated context
ctx = RubyLLM.context do |config|
  config.openai_api_key = ENV['ANOTHER_PROVIDER_KEY']
  config.request_timeout = 180
end

ctx_chat = ctx.chat(model: 'gpt-5.2')
```

### Multi-Tenant Applications

```ruby
class TenantService
  def initialize(tenant)
    @context = RubyLLM.context do |config|
      config.openai_api_key = tenant.openai_key
      config.default_model = tenant.preferred_model
    end
  end

  def chat
    @context.chat
  end
end
```

## Rails Integration

```ruby
# config/initializers/ruby_llm.rb
RubyLLM.configure do |config|
  config.openai_api_key = Rails.application.credentials.openai_api_key
  config.logger = Rails.logger
  config.request_timeout = Rails.env.production? ? 120 : 30
end
```

### Important: `use_new_acts_as` Timing

If using `use_new_acts_as = true`, configure in `config/application.rb` **before** the Application class:

```ruby
# config/application.rb
RubyLLM.configure do |config|
  config.use_new_acts_as = true
end

module YourApp
  class Application < Rails::Application
    # ...
  end
end
```

## Configuration Reference

```ruby
RubyLLM.configure do |config|
  # Provider API Keys
  config.openai_api_key = String
  config.anthropic_api_key = String
  config.gemini_api_key = String
  config.vertexai_project_id = String
  config.deepseek_api_key = String
  config.mistral_api_key = String

  # Provider Endpoints
  config.openai_api_base = String
  config.ollama_api_base = String

  # Default Models
  config.default_model = String
  config.default_embedding_model = String
  config.default_image_model = String

  # Connection
  config.request_timeout = Integer
  config.max_retries = Integer
  config.http_proxy = String

  # Logging
  config.logger = Logger
  config.log_file = String
  config.log_level = Symbol
end
```
