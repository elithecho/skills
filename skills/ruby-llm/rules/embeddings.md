# Embeddings

Transform text into numerical vectors for semantic search, recommendations, and content similarity.

## Basic Embedding Generation

```ruby
embedding = RubyLLM.embed("Ruby is a programmer's best friend")

# The vector representation (array of floats)
vector = embedding.vectors
puts "Vector dimension: #{vector.length}" # e.g., 1536

# Metadata
puts "Model used: #{embedding.model}"
puts "Input tokens: #{embedding.input_tokens}"
```

## Embedding Multiple Texts

```ruby
texts = ["Ruby", "Python", "JavaScript"]
embeddings = RubyLLM.embed(texts)

puts "Number of vectors: #{embeddings.vectors.length}" # => 3
puts "First vector dimensions: #{embeddings.vectors.first.length}"
puts "Model used: #{embeddings.model}"
puts "Total input tokens: #{embeddings.input_tokens}"
```

Batching multiple texts is more performant than individual requests.

## Choosing Models

```ruby
# Specific OpenAI model
embedding = RubyLLM.embed("Test sentence", model: "text-embedding-3-large")

# Google model
embedding = RubyLLM.embed("Test sentence", model: "text-embedding-004")

# Custom model
embedding = RubyLLM.embed(
  "Custom model test",
  model: "my-custom-embedding-model",
  provider: :openai,
  assume_model_exists: true
)
```

Configure default:

```ruby
RubyLLM.configure do |config|
  config.default_embedding_model = "text-embedding-3-large"
end
```

## Choosing Dimensions

```ruby
embedding = RubyLLM.embed(
  "Test sentence",
  model: "text-embedding-3-small",
  dimensions: 512
)
```

Useful for:
- Vector databases with specific dimension requirements
- Consistent dimensionality across requests
- Optimizing storage and query performance

## Using Embedding Results

### Vector Properties

```ruby
embedding = RubyLLM.embed("Example text")

puts embedding.vectors.class   # => Array
puts embedding.vectors.first.class  # => Float
puts embedding.model          # => "text-embedding-3-small"
```

### Calculating Similarity

```ruby
require 'matrix'

embedding1 = RubyLLM.embed("I love Ruby programming")
embedding2 = RubyLLM.embed("Ruby is my favorite language")

vector1 = Vector.elements(embedding1.vectors)
vector2 = Vector.elements(embedding2.vectors)

# Cosine similarity (-1 to 1, closer to 1 = more similar)
similarity = vector1.inner_product(vector2) / (vector1.norm * vector2.norm)
puts "Similarity: #{similarity.round(4)}" # => e.g., 0.9123
```

## Error Handling

```ruby
begin
  embedding = RubyLLM.embed("Your text here")
rescue RubyLLM::Error => e
  puts "Embedding failed: #{e.message}"
end
```

## Performance Best Practices

- **Batching**: Always embed multiple texts in a single call when possible
- **Caching**: Store generated embeddings instead of regenerating
- **Dimensionality**: Ensure correct dimensions (e.g., 1536 for text-embedding-3-small)
- **Normalization**: Some vector databases work better with normalized vectors

## Rails Integration Example

With PostgreSQL and pgvector:

```ruby
# Migration:
# add_column :documents, :embedding, :vector, limit: 1536

class Document < ApplicationRecord
  has_neighbors :embedding

  before_save :generate_embedding, if: :content_changed?

  scope :search_by_similarity, ->(query_text, limit: 5) {
    query_embedding = RubyLLM.embed(query_text).vectors
    nearest_neighbors(:embedding, query_embedding, distance: :cosine).limit(limit)
  }

  private

  def generate_embedding
    return if content.blank?
    embedding_result = RubyLLM.embed(content)
    self.embedding = embedding_result.vectors
  rescue RubyLLM::Error => e
    errors.add(:base, "Failed to generate embedding: #{e.message}")
    throw :abort
  end
end

# Usage
Document.create(title: "Intro to Ruby", content: "Ruby is a dynamic language...")
results = Document.search_by_similarity("What is Ruby?")
```
