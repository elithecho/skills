# Moderation

Identify potentially harmful content in text using AI moderation models.

## Basic Content Moderation

```ruby
result = RubyLLM.moderate("This is a safe message about Ruby programming")

puts result.flagged?  # => false

puts result.results
# => [{"flagged" => false, "categories" => {...}, "category_scores" => {...}}]

puts "Moderation ID: #{result.id}"
puts "Model used: #{result.model}"
```

## Understanding Moderation Results

```ruby
result = RubyLLM.moderate("Some user input text")

# Check overall flagging
if result.flagged?
  puts "Content was flagged for: #{result.flagged_categories.join(', ')}"
else
  puts "Content appears safe"
end

# Category scores (0.0 to 1.0, higher = more likely)
scores = result.category_scores
puts "Sexual content score: #{scores['sexual']}"
puts "Harassment score: #{scores['harassment']}"
puts "Violence score: #{scores['violence']}"

# Boolean flags for each category
categories = result.categories
puts "Contains hate speech: #{categories['hate']}"
puts "Contains self-harm content: #{categories['self-harm']}"
```

### Moderation Categories

- **Sexual**: Sexually explicit or suggestive content
- **Hate**: Content promoting hate based on identity
- **Harassment**: Content intended to harass, threaten, or bully
- **Self-harm**: Content promoting or encouraging self-harm
- **Sexual/minors**: Sexual content involving minors
- **Hate/threatening**: Hateful content with threats
- **Violence**: Content promoting or glorifying violence
- **Violence/graphic**: Graphic violent content
- **Self-harm/intent**: Expressing intent to self-harm
- **Self-harm/instructions**: Instructions for self-harm
- **Harassment/threatening**: Harassment with threats

## Alternative Calling Methods

```ruby
# Direct class method
result = RubyLLM::Moderation.moderate("Your content here")

# With explicit model
result = RubyLLM.moderate(
  "Content to check",
  model: "text-moderation-007",
  provider: "openai"
)
```

## Choosing Models

```ruby
# Specific model
result = RubyLLM.moderate("Content", model: "text-moderation-007")

# Configure default
RubyLLM.configure do |config|
  config.default_moderation_model = "text-moderation-007"
end
```

## Integration Patterns

### Pre-Chat Moderation

```ruby
def safe_chat_response(user_input)
  moderation = RubyLLM.moderate(user_input)

  if moderation.flagged?
    flagged_categories = moderation.flagged_categories.join(', ')
    return {
      error: "Content flagged for: #{flagged_categories}",
      safe: false
    }
  end

  response = RubyLLM.chat.ask(user_input)
  {
    content: response.content,
    safe: true
  }
end
```

### Custom Threshold Handling

```ruby
def assess_content_risk(text)
  result = RubyLLM.moderate(text)
  scores = result.category_scores

  high_risk = scores.any? { |_, score| score > 0.8 }
  medium_risk = scores.any? { |_, score| score > 0.5 }

  case
  when high_risk
    { risk: :high, action: :block, message: "Content blocked" }
  when medium_risk
    { risk: :medium, action: :review, message: "Content flagged for review" }
  else
    { risk: :low, action: :allow, message: "Content approved" }
  end
end
```

## Error Handling

```ruby
begin
  result = RubyLLM.moderate("User content")

  if result.flagged?
    handle_unsafe_content(result)
  else
    process_safe_content(content)
  end
rescue RubyLLM::ConfigurationError => e
  logger.error "Moderation not configured: #{e.message}"
rescue RubyLLM::RateLimitError => e
  logger.warn "Moderation rate limited: #{e.message}"
rescue RubyLLM::Error => e
  logger.error "Moderation failed: #{e.message}"
end
```

## Configuration Requirements

Moderation requires OpenAI API key:

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = ENV['OPENAI_API_KEY']
  config.default_moderation_model = "omni-moderation-latest"
end
```

## Best Practices

### Content Safety Strategy

- Always moderate user-generated content before sending to LLMs
- Handle false positives gracefully with human review
- Log moderation decisions for auditing
- Provide clear feedback to users

### Performance Considerations

- Cache moderation results for repeated content
- Use background jobs for large volumes
- Implement fallbacks when services unavailable

### User Experience

```ruby
def user_friendly_moderation(content)
  result = RubyLLM.moderate(content)
  return { approved: true } unless result.flagged?

  categories = result.flagged_categories
  message = case
  when categories.include?('harassment')
    "Please keep interactions respectful and constructive."
  when categories.include?('sexual')
    "This content appears inappropriate for our platform."
  when categories.include?('violence')
    "Please avoid content that promotes violence or harm."
  else
    "This content doesn't meet our community guidelines."
  end

  { approved: false, message: message, categories: categories }
end
```

## Rails Integration

```ruby
class MessageController < ApplicationController
  def create
    content = params[:message]
    moderation_result = RubyLLM.moderate(content)

    if moderation_result.flagged?
      render json: {
        error: "Message not allowed",
        categories: moderation_result.flagged_categories
      }, status: :unprocessable_entity
    else
      message = Message.create!(content: content, user: current_user)
      render json: message, status: :created
    end
  end
end

class ModerationJob < ApplicationJob
  def perform(message_ids)
    Message.where(id: message_ids).each do |message|
      result = RubyLLM.moderate(message.content)
      message.update!(
        moderation_flagged: result.flagged?,
        moderation_categories: result.flagged_categories,
        moderation_scores: result.category_scores
      )
    end
  end
end
```
