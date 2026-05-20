# Scale with Async

Handle hundreds of concurrent AI requests on modest hardware using Ruby's async ecosystem.

## Why Async for LLMs?

LLM operations take 5-60 seconds and spend 99% of that time waiting for tokens. Traditional thread-based queues (Sidekiq, GoodJob) have a problem:

```ruby
# With 25 worker threads:
# - 26th user waits in line
# - Thread is 99% idle during LLM call
# - Wastes resources
```

Async solves this with fibers instead of threads:
- **Threads**: OS-managed, preemptive, heavy
- **Fibers**: Userspace, cooperative, lightweight (thousands can share a few connections)

## How RubyLLM Works with Async

RubyLLM automatically becomes non-blocking in async context:

```ruby
require 'async'
require 'ruby_llm'

Async do
  10.times.map do
    Async do
      message = RubyLLM.chat.ask "Explain quantum computing"
      puts message.content
    end
  end.map(&:wait)
end
```

This works because RubyLLM uses `Net::HTTP`, which cooperates with Ruby's fiber scheduler.

## Concurrent Operations

### Multiple Chat Requests

```ruby
def process_questions(questions)
  Async do
    tasks = questions.map do |question|
      Async do
        response = RubyLLM.chat.ask(question)
        { question: question, answer: response.content }
      end
    end
    tasks.map(&:wait)
  end.result
end

results = process_questions(["What is Ruby?", "Explain metaprogramming", "What are symbols?"])
```

### Batch Embeddings

```ruby
def generate_embeddings(texts, batch_size: 100)
  Async do
    embeddings = []
    texts.each_slice(batch_size) do |batch|
      task = Async do
        response = RubyLLM.embed(batch)
        response.vectors
      end
      embeddings.concat(task.wait)
    end
    texts.zip(embeddings)
  end.result
end
```

### Parallel Analysis

```ruby
def analyze_document(content)
  Async do
    summary_task = Async do
      RubyLLM.chat.ask("Summarize: #{content}")
    end
    sentiment_task = Async do
      RubyLLM.chat.ask("Is this positive or negative: #{content}")
    end
    {
      summary: summary_task.wait.content,
      sentiment: sentiment_task.wait.content
    }
  end.result
end
```

## Background Processing with Async::Job

Use `Async::Job` for background processing.

### Setup with Falcon (Recommended)

```ruby
# Gemfile
gem 'falcon'
gem 'async-job-adapter-active_job'
```

```ruby
# config/application.rb
config.active_job.queue_adapter = :async_job
```

```ruby
# config/initializers/async_job_adapter.rb
require 'async/job/processor/inline'

Rails.application.configure do
  config.async_job.define_queue "default" do
    dequeue Async::Job::Processor::Inline
  end
end
```

Start with `bin/dev`. Jobs run concurrently without additional infrastructure.

### Note on Puma

With Puma, use Redis-backed processor:

```ruby
# Gemfile
gem 'async-job-processor-redis'
```

```ruby
# config/initializers/async_job_adapter.rb
require 'async/job/processor/redis'

Rails.application.configure do
  config.async_job.define_queue "default" do
    dequeue Async::Job::Processor::Redis
  end
end
```

Run in Procfile.dev:
```
web: bin/rails server
redis: redis-server
async_job: bundle exec async-job-adapter-active_job-server
```

### Your Jobs Work Unchanged

```ruby
class DocumentAnalyzerJob < ApplicationJob
  def perform(document_id)
    document = Document.find(document_id)
    
    # Runs in async context automatically
    response = RubyLLM.chat.ask("Analyze: #{document.content}")
    document.update!(analysis: response.content)
  end
end
```

### Mixing Job Adapters

```ruby
# Default adapter for regular jobs
config.active_job.queue_adapter = :solid_queue

# Base class for LLM jobs
class LLMJob < ApplicationJob
  self.queue_adapter = :async_job
end

class ChatResponseJob < LLMJob
  def perform(conversation_id, message)
    # Uses async-job
  end
end
```

## Rate Limiting with Semaphores

```ruby
require 'async/semaphore'

class RateLimitedProcessor
  def initialize(max_concurrent: 10)
    @semaphore = Async::Semaphore.new(max_concurrent)
  end

  def process_items(items)
    Async do
      items.map do |item|
        Async do
          @semaphore.acquire do
            response = RubyLLM.chat.ask("Process: #{item}")
            { item: item, result: response.content }
          end
        end
      end.map(&:wait)
    end
  end
end

processor = RateLimitedProcessor.new(max_concurrent: 5)
results = processor.process_items(["Item 1", "Item 2", "Item 3"])
```

## Summary

- LLM operations are perfect for async (99% waiting for I/O)
- RubyLLM automatically works with async
- Use async-job for LLM background jobs
- Use semaphores to manage rate limits
- Keep thread-based processors for CPU-intensive work
