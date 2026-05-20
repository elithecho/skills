# Streaming Responses

Display AI responses in real-time as they're generated.

## Basic Streaming

Provide a block to the `ask` method:

```ruby
chat = RubyLLM.chat

puts "Assistant:"
chat.ask "Write a short story about a adventurous ruby gem." do |chunk|
  print chunk.content
end
# => Output appears incrementally
```

RubyLLM normalizes different provider streaming formats into standardized `Chunk` objects.

## Understanding Chunks

Each block yield is a `RubyLLM::Chunk` instance:

```ruby
chat.ask "Write a story." do |chunk|
  # chunk.content    - Text fragment (can be nil for metadata chunks)
  # chunk.role      - Always :assistant for streamed responses
  # chunk.model_id  - The model generating the response
  # chunk.tool_calls - Hash with tool call info if model invokes a tool
  # chunk.input_tokens - Total input tokens (often nil until final chunk)
  # chunk.output_tokens - Cumulative output tokens
  # chunk.thinking - Optional thinking output
end
```

> Don't rely on token counts being present in every chunk - they're typically only accurate in the final chunk.

## Accumulated Response

The `ask` method returns the complete message after streaming finishes:

```ruby
chat = RubyLLM.chat
final_message = nil

puts "Assistant:"
final_message = chat.ask "Write a short haiku about programming." do |chunk|
  print chunk.content
end

puts "\n--- Final Message ---"
puts final_message.content
puts "Total Tokens: #{(final_message.input_tokens || 0) + (final_message.output_tokens || 0)}"
```

## Web Application Integration

### Rails with Turbo Streams

```ruby
# app/jobs/chat_stream_job.rb
class ChatStreamJob < ApplicationJob
  queue_as :default

  def perform(chat_id, user_message, stream_target_id)
    chat = Chat.find(chat_id)
    full_response = ""

    # Broadcast initial placeholder
    Turbo::StreamsChannel.broadcast_replace_to(
      "chat_#{chat.id}",
      target: stream_target_id,
      partial: "messages/streaming_message",
      locals: { content: "Thinking..." }
    )

    chat.ask(user_message) do |chunk|
      full_response << (chunk.content || "")
      Turbo::StreamsChannel.broadcast_replace_to(
        "chat_#{chat.id}",
        target: stream_target_id,
        partial: "messages/streaming_message",
        locals: { content: full_response }
      )
    end
  end
end
```

### Sinatra with Server-Sent Events (SSE)

```ruby
require 'sinatra'
require 'ruby_llm'

get '/stream_chat' do
  content_type 'text/event-stream'
  stream(:keep_open) do |out|
    chat = RubyLLM.chat
    begin
      chat.ask(params[:prompt] || "Tell me a fun fact.") do |chunk|
        out << "data: #{chunk.content.to_json}\n\n" if chunk.content
      end
      out << "event: complete\ndata: {}\n\n"
    rescue => e
      out << "event: error\ndata: #{ { error: e.message }.to_json }\n\n"
    ensure
      out.close
    end
  end
end
```

## Error Handling During Streaming

Errors occur mid-stream. The `ask` method raises after block execution finishes:

```ruby
begin
  chat = RubyLLM.chat
  puts "Assistant:"
  chat.ask("Generate a very long response...") do |chunk|
    print chunk.content
  end
rescue RubyLLM::Error => e
  puts "\n--- Error during streaming ---"
  puts "Error Type: #{e.class}"
  puts "Message: #{e.message}"
end
```

## Streaming with Tools

When tools are involved, streaming has distinct phases:

1. **Initial Response Stream**: Chunks arrive until model decides to call a tool
2. **Tool Call Chunk(s)**: Contains `chunk.tool_calls` information
3. **Pause**: RubyLLM executes the tool
4. **Resumed Response Stream**: Final response chunks after tool result

```ruby
chat = RubyLLM.chat(model: 'gpt-5.2').with_tool(Weather)

puts "Assistant:"
chat.ask("What's the weather in Berlin?") do |chunk|
  if chunk.tool_calls
    puts "\n[TOOL CALL DETECTED: #{chunk.tool_calls.values.first.name}]"
  elsif chunk.content
    print chunk.content
  end
end
```

The streaming block needs to handle:
- Text content chunks
- Tool call information chunks
- Metadata-only chunks
