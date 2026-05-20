# Tools - Function Calling

Let AI call your Ruby code. Connect to databases, APIs, or any external system.

## What Are Tools?

Tools bridge the gap between AI models and the real world. They allow the model to delegate tasks to your application code.

Common use cases:
- Fetching real-time data (weather, stock prices)
- Database interaction
- External API calls
- Calculations
- Executing business logic

## Creating a Tool

Define a tool by creating a class that inherits from `RubyLLM::Tool`:

```ruby
class Weather < RubyLLM::Tool
  description "Gets current weather for a location"

  params do
    string :latitude, description: "Latitude (e.g., 52.5200)"
    string :longitude, description: "Longitude (e.g., 13.4050)"
  end

  def execute(latitude:, longitude:)
    url = "https://api.open-meteo.com/v1/forecast?latitude=#{latitude}&longitude=#{longitude}&current=temperature_2m,wind_speed_10m"
    response = Faraday.get(url)
    JSON.parse(response.body)
  rescue => e
    { error: e.message }
  end
end
```

### Tool Components

1. **Inheritance**: Must inherit from `RubyLLM::Tool`
2. **`description`**: Defines what the tool does (used by the model for tool selection)
3. **`params`**: DSL for describing input schema (v1.9+)
4. **`execute` Method**: Contains your Ruby code

### Tool Name

The class name is automatically converted to snake_case:
- `WeatherLookup` becomes `weather_lookup`

Override with:
```ruby
def name
  "Weather"
end
```

## Declaring Parameters

### params DSL (v1.9+)

```ruby
class Scheduler < RubyLLM::Tool
  description "Books a meeting"

  params do
    object :window do
      string :start, description: "ISO8601 start time"
      string :finish, description: "ISO8601 end time"
    end

    array :participants, of: :string
    any_of :format do
      string enum: %w[virtual in_person]
      null
    end
  end

  def execute(window:, participants:, format: nil)
    # ...
  end
end
```

### Using param Helper

```ruby
class Distance < RubyLLM::Tool
  description "Calculates distance between two cities"
  param :origin, desc: "Origin city name"
  param :destination, desc: "Destination city name"
  param :units, type: :string, required: false

  def execute(origin:, destination:, units: "metric")
    # ...
  end
end
```

### Manual JSON Schema

```ruby
class Lookup < RubyLLM::Tool
  description "Performs catalog lookups"

  params type: "object",
    properties: {
      sku: { type: "string", description: "Product SKU" }
    },
    required: %w[sku],
    additionalProperties: false,
    strict: true

  def execute(sku:)
    # ...
  end
end
```

## Returning Rich Content

Tools can return files for the AI to analyze:

```ruby
class AnalyzeTool < RubyLLM::Tool
  description "Analyzes data and returns visualizations"
  param :query

  def execute(query:)
    chart_path = generate_chart(query)

    RubyLLM::Content.new(
      "Analysis complete for: #{query}",
      [chart_path]
    )
  end
end
```

## Custom Initialization

```ruby
class DocumentSearch < RubyLLM::Tool
  description "Searches documents"
  param :query, desc: "The search query"
  param :limit, type: :integer, required: false

  def initialize(database)
    @database = database
  end

  def execute(query:, limit: 5)
    @database.search(query, limit: limit)
  end
end

search_tool = DocumentSearch.new(MyDatabase)
chat.with_tool(search_tool)
```

## Using Tools in Chat

```ruby
chat = RubyLLM.chat(model: 'gpt-5.2')
chat.with_tool(Weather.new)

response = chat.ask "What's the weather in Berlin? (Lat: 52.52, Long: 13.40)"
puts response.content
# => "Current weather at 52.52, 13.4: Temperature: 12.5°C..."
```

### Multiple Tools

```ruby
chat.with_tools(Weather, Calculator, SearchTool)

# Replace all tools
chat.with_tools(NewTool, AnotherTool, replace: true)

# Clear tools
chat.with_tools(replace: true)
```

### Tool Call Controls (v1.13+)

```ruby
# Model decides if a tool is needed
chat.with_tools(Weather, Calculator, choice: :auto)

# Model must call a tool
chat.with_tools(Weather, Calculator, choice: :required)

# Disable tool calls
chat.with_tools(Weather, Calculator, choice: :none)

# Force specific tool
chat.with_tools(Weather, Calculator, choice: :weather)

# Allow multiple tool calls (default)
chat.with_tools(Weather, Calculator, calls: :many)

# Allow one tool call
chat.with_tools(Weather, Calculator, calls: :one)
```

## Tool Execution Flow

1. **User Query**: Your message is sent to the model
2. **Model Decision**: Model decides a tool is needed
3. **Tool Call Request**: Model responds with tool name and arguments
4. **RubyLLM Execution**: RubyLLM finds the tool and calls `execute`
5. **Tool Result**: The result is sent back to the model
6. **Final Response**: Model generates natural language response
7. **Return**: RubyLLM returns the final message

## Monitoring with Callbacks

```ruby
chat = RubyLLM.chat(model: 'gpt-5.2')
      .with_tool(Weather)
      .on_tool_call do |tool_call|
        puts "Calling tool: #{tool_call.name}"
        puts "Arguments: #{tool_call.arguments}"
      end
      .on_tool_result do |result|
        puts "Tool returned: #{result}"
      end
```

### Example: Limiting Tool Calls

```ruby
call_count = 0
max_calls = 10

chat = RubyLLM.chat(model: 'gpt-5.2')
      .with_tool(Weather)
      .on_tool_call do |tool_call|
        call_count += 1
        raise "Tool call limit exceeded" if call_count > max_calls
      end
```

## Advanced Tool Metadata

### Provider-Specific Parameters

```ruby
class TodoTool < RubyLLM::Tool
  description "Adds a task to the TODO list"
  param :title

  with_params cache_control: { type: "ephemeral" }

  def execute(title:)
    Todo.create!(title: title)
    "Added \"#{title}\" to the list."
  end
end
```

## Halting Tool Continuation

Skip the LLM's summary after tool execution:

```ruby
class SaveFileTool < RubyLLM::Tool
  description "Save content to a file"
  param :path
  param :content

  def execute(path:, content:)
    File.write(path, content)
    halt "Saved to #{path}"  # Returns directly, no LLM commentary
  end
end
```

Use `halt` for:
- Token savings
- Sub-agent delegation
- Precise responses

## Debugging Tools

```bash
export RUBYLLM_DEBUG=true
```

You'll see:
```
RubyLLM: Tool weather_lookup called with: {:latitude=>52.52, :longitude=>13.4}
RubyLLM: Tool weather_lookup returned: "Current weather at 52.52..."
```

## Error Handling in Tools

- **Recoverable errors**: Return `{ error: "description" }`
- **Unrecoverable errors**: Raise an exception

```ruby
def execute(location:)
  return { error: "Location too short" } if location.length < 3

  # Fetch weather data...
rescue Faraday::ConnectionFailed
  { error: "Weather service unavailable" }
end
```

## Security Considerations

> Treat arguments as potentially untrusted user input.

- **NEVER** use `eval`, `system`, `send`, or direct SQL interpolation
- **Validate and Sanitize** all inputs
- **Principle of Least Privilege**: Only access needed resources
