# Error Handling

Handle errors gracefully when working with AI providers.

## Error Hierarchy

All errors inherit from `RubyLLM::Error`:

```ruby
RubyLLM::Error                    # Base error
    RubyLLM::BadRequestError      # 400: Invalid request
    RubyLLM::UnauthorizedError    # 401: API key issues
    RubyLLM::PaymentRequiredError # 402: Billing issues
    RubyLLM::ForbiddenError       # 403: Permission issues
    RubyLLM::ContextLengthExceededError  # Context exceeded
    RubyLLM::RateLimitError      # 429: Rate limit
    RubyLLM::ServerError         # 500: Server error
    RubyLLM::ServiceUnavailableError  # 502/503/504
    RubyLLM::OverloadedError     # 529: Service overloaded

# Non-API Errors
RubyLLM::ConfigurationError   # Missing config
RubyLLM::ModelNotFoundError   # Model not in registry
RubyLLM::InvalidRoleError     # Invalid message role
```

## Basic Error Handling

```ruby
begin
  chat = RubyLLM.chat
  response = chat.ask "Translate 'hello' to French."
  puts response.content
rescue RubyLLM::Error => e
  puts "An API error occurred: #{e.message}"
rescue RubyLLM::ConfigurationError => e
  puts "Configuration missing: #{e.message}"
end
```

## Handling Specific Errors

```ruby
begin
  chat = RubyLLM.chat
  response = chat.ask "Generate a complex report."
rescue RubyLLM::UnauthorizedError
  puts "Check your API key configuration."
rescue RubyLLM::PaymentRequiredError
  puts "Check your account balance."
rescue RubyLLM::RateLimitError
  puts "Wait a moment before trying again."
rescue RubyLLM::ContextLengthExceededError
  puts "Reduce prompt size or use a larger context model."
rescue RubyLLM::ServiceUnavailableError
  puts "Service temporarily unavailable. Try again later."
rescue RubyLLM::BadRequestError => e
  puts "Invalid request: #{e.message}"
rescue RubyLLM::ModelNotFoundError => e
  puts "Model not found: #{e.message}"
rescue RubyLLM::Error => e
  puts "Unexpected error: #{e.message}"
end
```

## Accessing API Response Details

```ruby
begin
  chat = RubyLLM.chat
  response = chat.ask "Some query"
rescue RubyLLM::ForbiddenError => e
  puts "Access forbidden: #{e.message}"
  puts "Status Code: #{e.response&.status}"
  # e.response is a Faraday::Response object
end
```

## Error Handling During Streaming

Errors occur mid-stream. The `ask` method raises after block finishes:

```ruby
begin
  chat = RubyLLM.chat
  accumulated_content = ""
  chat.ask "Generate a very long story..." do |chunk|
    print chunk.content
    accumulated_content << chunk.content
  end
rescue RubyLLM::RateLimitError
  puts "\nPartial content received:"
  puts accumulated_content
rescue RubyLLM::Error => e
  puts "\nStream failed: #{e.message}"
end
```

## Handling Errors Within Tools

Two approaches:

### 1. Return Error to LLM

For recoverable errors the LLM might fix:

```ruby
class Weather < RubyLLM::Tool
  param :city

  def execute(city:)
    if city.blank?
      return { error: "City cannot be blank. Please provide a city name." }
    end
    # ... API call ...
  rescue Faraday::TimeoutError
    { error: "Weather API timed out. Please try again later." }
  end
end
```

### 2. Raise Error for Application

For unrecoverable errors:

```ruby
class DatabaseQueryTool < RubyLLM::Tool
  param :query

  def execute(query:)
    User.find_by_sql(query)
  rescue ActiveRecord::ConnectionNotEstablished => e
    raise e  # Let application handle it
  rescue StandardError => e
    { error: "Query failed: #{e.message}" }
  end
end
```

## Automatic Retries

RubyLLM automatically retries transient failures:

```ruby
RubyLLM.configure do |config|
  config.max_retries = 5  # Default: 3
  config.retry_interval = 0.5  # Default: 0.1
  config.retry_backoff_factor = 2
end
```

Retries:
- Network timeouts
- Connection failures
- Rate limits
- Server errors

Does NOT retry:
- `ContextLengthExceededError`

## Debugging

Enable debug logging:

```bash
export RUBYLLM_DEBUG=true
```

This logs detailed request/response information.

## Best Practices

1. **Be Specific**: Rescue specific error classes
2. **Log Errors**: Include context (model, input data)
3. **User Feedback**: Provide clear, friendly messages
4. **Fallbacks**: Try different models, use cached data
5. **Monitor**: Track error frequency in production
