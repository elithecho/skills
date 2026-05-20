# Agentic Workflows

Build workflow-oriented AI systems with plain Ruby orchestration.

## Workflow Patterns

A workflow is orchestration code that coordinates one or more agents.

### Sequential Workflow

Use when each step depends on the previous one:

```ruby
class ResearchAgent < RubyLLM::Agent
  model "gemini-2.5-pro"
  instructions "Return concise, reliable key facts."
end

class WriterAgent < RubyLLM::Agent
  model "gpt-5"
  instructions "Write a clear article from research notes."
end

class ResearchWriterWorkflow
  def create_article(topic)
    research = ResearchAgent.new.ask(topic).content
    WriterAgent.new.ask(research).content
  end
end

workflow = ResearchWriterWorkflow.new
article = workflow.create_article("Ruby 3.3 features")
```

### Routing Workflow

Use when requests fall into clear categories:

```ruby
class CodeAgent < RubyLLM::Agent
  model "gpt-5"
  instructions "Be precise and practical."
end

class CreativeAgent < RubyLLM::Agent
  model "gpt-5"
  instructions "Be creative and expressive."
end

class FactualAgent < RubyLLM::Agent
  model "gemini-2.5-pro"
  instructions "Prioritize accuracy."
end

class TaskClassifierAgent < RubyLLM::Agent
  model "gpt-5-mini"
  instructions "Classify as: code, creative, or factual."
end

class ModelRouterWorkflow
  def call(query)
    agent_for(query).new.ask(query).content
  end

  private

  def agent_for(query)
    case classify(query)
    when :code then CodeAgent
    when :creative then CreativeAgent
    else FactualAgent
    end
  end

  def classify(query)
    TaskClassifierAgent.new.ask(query).content.downcase.to_sym
  end
end
```

### Parallel Workflow

Use when independent analyses can run simultaneously:

```ruby
require 'async'

class SentimentAgent < RubyLLM::Agent
  instructions "Return one word: positive, negative, or neutral."
end

class SummaryAgent < RubyLLM::Agent
  instructions "Summarize in one sentence."
end

class KeywordAgent < RubyLLM::Agent
  instructions "Extract exactly 5 keywords."
end

class ParallelAnalyzer
  def analyze(text)
    Async do |task|
      sentiment = task.async { SentimentAgent.new.ask(text).content }
      summary = task.async { SummaryAgent.new.ask(text).content }
      keywords = task.async { KeywordAgent.new.ask(text).content }

      {
        sentiment: sentiment.wait,
        summary: summary.wait,
        keywords: keywords.wait
      }
    end.wait
  end
end
```

### Fan-Out/Fan-In Workflow

Use when multiple specialists produce outputs for synthesis:

```ruby
require 'async'

class SecurityReviewAgent < RubyLLM::Agent
  model "gpt-5"
  instructions "Review code for security issues."
end

class PerformanceReviewAgent < RubyLLM::Agent
  model "gpt-5.2"
  instructions "Review code for performance issues."
end

class StyleReviewAgent < RubyLLM::Agent
  model "gpt-5-mini"
  instructions "Review style against Ruby conventions."
end

class ReviewSynthesizerAgent < RubyLLM::Agent
  instructions "Summarize prioritized findings."
end

class CodeReviewSystem
  def review_code(code)
    Async do |task|
      security = task.async { SecurityReviewAgent.new.ask(code).content }
      performance = task.async { PerformanceReviewAgent.new.ask(code).content }
      style = task.async { StyleReviewAgent.new.ask(code).content }

      ReviewSynthesizerAgent.new.ask(
        "security: #{security.wait}\n\n" \
        "performance: #{performance.wait}\n\n" \
        "style: #{style.wait}"
      ).content
    end.wait
  end
end
```

### Evaluation Loop

Use when you have clear quality criteria and want iterative refinement:

```ruby
class DraftAgent < RubyLLM::Agent
  instructions "Produce the best possible draft."
end

class CriticAgent < RubyLLM::Agent
  schema do
    string :verdict, enum: ["pass", "revise"]
    string :feedback
  end
  instructions "Review draft and return verdict + feedback."
end

class EvaluatorOptimizerWorkflow
  MAX_ROUNDS = 3

  def call(task)
    draft = DraftAgent.new.ask(task).content

    MAX_ROUNDS.times do
      verdict, feedback = review(task: task, draft: draft)
      return draft if verdict == "pass"
      draft = revise(task: task, draft: draft, feedback: feedback)
    end

    draft
  end

  private

  def review(task:, draft:)
    result = CriticAgent.new.ask("Task: #{task}\n\nDraft: #{draft}").content
    [result.fetch("verdict"), result.fetch("feedback")]
  end

  def revise(task:, draft:, feedback:)
    DraftAgent.new.ask("Task: #{task}\n\nDraft: #{draft}\n\nFeedback: #{feedback}").content
  end
end
```

## RAG as a Workflow Step

### Best Practices

**Don't use tool descriptions as prompts.** Tool descriptions are used by the model to understand what a tool does, but they shouldn't be included as part of the prompt sent to the model. Let the model use tool descriptions directly through the tool-calling mechanism.

### Setup

```ruby
# Gemfile
gem 'neighbor'  # For pgvector
gem 'ruby_llm'

# Migration
class CreateDocuments < ActiveRecord::Migration[7.1]
  def change
    create_table :documents do |t|
      t.text :content
      t.string :title
      t.vector :embedding, limit: 1536
      t.timestamps
    end
    add_index :documents, :embedding, using: :hnsw, opclass: :vector_l2_ops
  end
end
```

### Document Model

```ruby
class Document < ApplicationRecord
  has_neighbors :embedding

  before_save :generate_embedding, if: :content_changed?

  private

  def generate_embedding
    response = RubyLLM.embed(content)
    self.embedding = response.vectors
  end
end
```

### Retrieval Tool

```ruby
class DocumentSearch < RubyLLM::Tool
  description "Searches knowledge base"
  param :query

  def execute(query:)
    embedding = RubyLLM.embed(query).vectors
    documents = Document.nearest_neighbors(:embedding, embedding, distance: "euclidean").limit(3)
    documents.map { |doc| "#{doc.title}: #{doc.content.truncate(500)}" }.join("\n\n---\n\n")
  end
end
```

### Answering Agent

```ruby
class SupportWithDocsAgent < RubyLLM::Agent
  tools DocumentSearch
  instructions "Search for context before answering. Cite sources."
end

response = SupportWithDocsAgent.new.ask("What is our refund policy?").content
```

## Error Handling

Follow patterns from the Tools guide:
- Return `{ error: "description" }` for recoverable errors
- Raise exceptions for unrecoverable errors
- Use retry middleware for transient failures
