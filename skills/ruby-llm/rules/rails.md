# Rails Integration

Rails + AI made simple. Persist chats with ActiveRecord. Stream with Hotwire.

## Quick Setup

```bash
rails generate ruby_llm:install
rails db:migrate
rails ruby_llm:load_models # v1.13+
```

### Adding a Chat UI

```bash
rails generate ruby_llm:chat_ui
```

This creates controllers, views with Turbo streaming, and routes. Visit `http://localhost:3000/chats`.

### Generator Options

```bash
# Default
rails generate ruby_llm:install

# Custom model names
rails generate ruby_llm:install chat:Conversation message:ChatMessage
rails generate ruby_llm:install chat:Discussion message:DiscussionMessage tool_call:FunctionCall model:AIModel
```

## Setting Up Models

### With Model Registry (v1.7+)

```ruby
# app/models/chat.rb
class Chat < ApplicationRecord
  acts_as_chat  # Defaults: messages: :messages, model: :model
  
  belongs_to :user, optional: true
end

# app/models/message.rb
class Message < ApplicationRecord
  acts_as_message  # Defaults: chat: :chat, tool_calls: :tool_calls, model: :model
  
  validates :role, presence: true
  validates :chat, presence: true
end

# app/models/tool_call.rb
class ToolCall < ApplicationRecord
  acts_as_tool_call  # Defaults: message: :message, result: :result
end

# app/models/model.rb
class Model < ApplicationRecord
  acts_as_model  # Defaults: chats: :chats
end
```

### Legacy Mode

```ruby
# app/models/chat.rb
class Chat < ApplicationRecord
  acts_as_chat message_class: 'Message',
               tool_call_class: 'ToolCall'
end

# app/models/message.rb
class Message < ApplicationRecord
  acts_as_message chat_class: 'Chat',
                  chat_foreign_key: 'chat_id'
end
```

## Persistence Flow

When calling `chat_record.ask("What is the capital of France?")`:

1. Saves the user message
2. Calls the AI provider
3. Creates an empty assistant message (before API call or on first chunk for streaming)
4. Updates the message on success, destroys on failure

## Working with Chats

```ruby
# Create chat
chat_record = Chat.create!(model: 'gpt-5-nano', user: current_user)

# Ask question - persistence runs automatically
response = chat_record.ask "What is the capital of France?"
puts chat_record.messages.last.content  # => "The capital of France is Paris."

# Continue conversation
chat_record.ask "Tell me more about that city"
puts "Conversation length: #{chat_record.messages.count}"  # => 4
```

### System Instructions

```ruby
chat_record = Chat.create!(model: 'gpt-5-nano')
chat_record.with_instructions("You are a Ruby expert.")
chat_record.with_instructions("Be concise.", append: true)
```

### Using Tools

```ruby
class Weather < RubyLLM::Tool
  description "Gets current weather"
  param :city
  
  def execute(city:)
    "Weather in #{city} is sunny."
  end
end

chat_record = Chat.create!(model: 'gpt-5-nano')
chat_record.with_tool(Weather)
response = chat_record.ask("What's the weather in Paris?")
```

### File Attachments

```ruby
# Single file - type auto-detected
chat_record.ask("What's in this file?", with: "diagram.png")

# Multiple files
chat_record.ask("Analyze these", with: ["report.pdf", "chart.jpg"])

# Works with file uploads
chat_record.ask("Analyze this", with: params[:uploaded_file])
```

### Structured Output

```ruby
class PersonSchema < RubyLLM::Schema
  string :name
  integer :age
end

chat_record = Chat.create!(model: 'gpt-5-nano')
response = chat_record.with_schema(PersonSchema).ask("Generate a person")
puts response.content  # => {"name" => "Marie", "age" => 28}
```

## Provider Overrides

```ruby
chat = Chat.create!(
  model: 'claude-sonnet-4-5',
  provider: 'bedrock'
)
```

## Custom Contexts

```ruby
# Multi-tenant with DB-backed registry
custom_context = RubyLLM.context do |config|
  config.openai_api_key = 'sk-customer-key'
end

chat = Chat.create!(
  model: 'gpt-5.2',
  context: custom_context
)
```

## Streaming with Hotwire/Turbo

```ruby
# app/jobs/chat_stream_job.rb
class ChatStreamJob < ApplicationJob
  def perform(chat_id, user_message)
    chat = Chat.find(chat_id)
    
    chat.ask(user_message) do |chunk|
      # Broadcast to Turbo Stream
      Turbo::StreamsChannel.broadcast_replace_to(
        "chat_#{chat.id}",
        target: "message_#{chat.messages.last.id}",
        partial: "messages/content",
        locals: { content: chunk.content }
      )
    end
  end
end
```

### Instant User Messages

```ruby
def create
  @chat = Chat.find(params[:chat_id])
  @chat.add_message(role: :user, content: params[:content])
  
  ChatStreamJob.perform_later(@chat.id)
  
  respond_to { |format| format.turbo_stream }
end
```

## Customizing Models

### Custom Names with Registry

```ruby
class Conversation < ApplicationRecord
  acts_as_chat messages: :chat_messages,
               message_class: 'ChatMessage',
               model: :ai_model,
               model_class: 'AiModel'
end

class ChatMessage < ApplicationRecord
  acts_as_message chat: :conversation,
                  tool_calls: :ai_tool_calls,
                  tool_call_class: 'AIToolCall',
                  model: :ai_model
end
```

### Namespaced Models

```ruby
module Admin
  class BotChat < ApplicationRecord
    acts_as_chat messages: :bot_messages,
                 message_class: 'Admin::BotMessage'
  end
end
```

## Validation Note

> You cannot use `validates :content, presence: true` on Message model. The persistence flow creates empty assistant messages.

For validation support, override persistence methods - see official docs for details.
