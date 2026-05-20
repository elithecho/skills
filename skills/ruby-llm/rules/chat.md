# Chat with RubyLLM

Learn how to have conversations with AI models, work with different providers, and handle multi-modal inputs.

## Starting a Conversation

Create a chat instance and ask questions:

```ruby
chat = RubyLLM.chat
response = chat.ask "Explain the concept of 'Convention over Configuration' in Rails."

puts response.content
# => "Convention over Configuration (CoC) is a core principle of Ruby on Rails..."

puts "Model Used: #{response.model_id}"
puts "Tokens: #{response.input_tokens} input, #{response.output_tokens} output"
```

The `ask` method adds your message to conversation history, sends it to the AI, and returns a `RubyLLM::Message` object.

## Continuing the Conversation

```ruby
# Continuing the previous chat...
response = chat.ask "Can you give a specific example in Rails?"

# Access the full conversation history
chat.messages.each do |message|
  puts "[#{message.role.to_s.upcase}] #{message.content.lines.first.strip}"
end
```

## Guiding AI Behavior with System Prompts

```ruby
chat = RubyLLM.chat

# Set instructions
chat.with_instructions "You are a helpful assistant that explains Ruby concepts simply."

response = chat.ask "What is a variable?"
# => "Imagine you have a special box..."

# Append additional instructions
chat.with_instructions "Use exactly one short paragraph.", append: true
```

## Working with Different Models

```ruby
# Use specific models
chat_claude = RubyLLM.chat(model: 'claude-sonnet-4-5')
chat_gemini = RubyLLM.chat(model: 'gemini-2.5-pro')

# Change model on existing chat
chat = RubyLLM.chat(model: 'gpt-5-nano')
chat.with_model('claude-sonnet-4-5')
```

## Multi-modal Conversations

### Working with Images

```ruby
chat = RubyLLM.chat(model: 'gpt-5.2')

# Local image
response = chat.ask "Describe this logo.", with: "path/to/ruby_logo.png"

# Image from URL
response = chat.ask "What kind of architecture?", with: "https://example.com/eiffel_tower.jpg"

# Multiple images
response = chat.ask "Compare these.", with: ["screenshot_v1.png", "screenshot_v2.png"]
```

### Working with Videos

```ruby
chat = RubyLLM.chat(model: 'gemini-2.5-flash')
response = chat.ask "What happens in this video?", with: "path/to/demo.mp4"
```

### Working with Audio

```ruby
chat = RubyLLM.chat(model: 'gpt-4o-audio-preview')
response = chat.ask "Please transcribe this meeting.", with: "path/to/meeting.mp3"
```

### Working with PDFs

```ruby
chat = RubyLLM.chat(model: 'claude-sonnet-4-5')
response = chat.ask "Summarize this paper.", with: "path/to/paper.pdf"
```

### Automatic File Type Detection

RubyLLM automatically detects file types:

```ruby
response = chat.ask "Analyze these files", with: [
  "diagram.png",
  "report.pdf",
  "meeting_notes.txt",
  "recording.mp3"
]
```

## Controlling Response Behavior

### Temperature

```ruby
# Low temperature - more deterministic
factual_chat = RubyLLM.chat.with_temperature(0.2)
response1 = factual_chat.ask "What is the boiling point of water?"

# High temperature - more creative
creative_chat = RubyLLM.chat.with_temperature(0.9)
response2 = creative_chat.ask "Write a short poem about the color blue."
```

### Provider-Specific Parameters

```ruby
chat = RubyLLM.chat.with_params(response_format: { type: 'json_object' })
response = chat.ask "What is the square root of 64? Answer with JSON."
```

## Raw Content Blocks

Pass custom content directly to providers:

```ruby
raw_block = RubyLLM::Content::Raw.new([
  { type: 'text', text: 'Analysis prompt' },
  { type: 'text', text: "Today's request: #{summary}" }
])

chat = RubyLLM.chat
chat.add_message(role: :system, content: raw_block)
chat.ask(raw_block)
```

### Anthropic Prompt Caching

```ruby
system_block = RubyLLM::Providers::Anthropic::Content.new(
  "You are a release-notes assistant.",
  cache: true
)

chat = RubyLLM.chat(model: 'claude-sonnet-4-5')
chat.add_message(role: :system, content: system_block)
```

## Getting Structured Output

### Using RubyLLM::Schema

```ruby
require 'ruby_llm/schema'

class PersonSchema < RubyLLM::Schema
  string :name, description: "Person's full name"
  integer :age, description: "Person's age in years"
  string :city, required: false
end

chat = RubyLLM.chat
response = chat.with_schema(PersonSchema).ask("Generate a person named Alice who is 30")

puts response.content # => {"name" => "Alice", "age" => 30}
```

### Using Manual JSON Schemas

```ruby
person_schema = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    age: { type: 'integer' }
  },
  required: ['name', 'age'],
  additionalProperties: false
}

chat = RubyLLM.chat
response = chat.with_schema(person_schema).ask("Generate a person")
```

## Tracking Token Usage

```ruby
response = chat.ask "Explain the Ruby GIL."

puts "Input Tokens: #{response.input_tokens}"
puts "Output Tokens: #{response.output_tokens}"
puts "Total: #{response.input_tokens + response.output_tokens}"

# Calculate cost
model_info = RubyLLM.models.find(response.model_id)
if model_info.input_price_per_million && model_info.output_price_per_million
  cost = response.input_tokens * model_info.input_price_per_million / 1_000_000
  cost += response.output_tokens * model_info.output_price_per_million / 1_000_000
  puts "Estimated Cost: $#{format('%.6f', cost)}"
end
```

## Chat Event Handlers

```ruby
chat = RubyLLM.chat

# Called at first chunk received
chat.on_new_message do
  print "Assistant > "
end

# Called after complete response
chat.on_end_message do |message|
  puts "Response complete!"
  puts "Used #{message.input_tokens + message.output_tokens} tokens" if message
end

# Called when AI decides to use a tool
chat.on_tool_call do |tool_call|
  puts "AI calling: #{tool_call.name}"
end

# Called after tool returns result
chat.on_tool_result do |result|
  puts "Tool returned: #{result}"
end
```

## Raw Responses

Access the raw provider response:

```ruby
response = chat.ask("What is the capital of France?")
puts response.raw.body
```
