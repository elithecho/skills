# Audio Transcription

Convert speech to text with support for multiple languages and speaker diarization.

## Basic Transcription

```ruby
transcription = RubyLLM.transcribe("meeting.wav")

puts transcription.text
# => "Welcome to today's meeting. Let's discuss..."

puts transcription.model
# => "whisper-1"
```

Supports MP3, M4A, WAV, WebM, OGG, and more.

## Choosing Models

```ruby
# Whisper-1 (default)
RubyLLM.transcribe("audio.mp3", model: "whisper-1")

# GPT-4o Transcribe (faster, better for technical content)
RubyLLM.transcribe("audio.mp3", model: "gpt-4o-transcribe")

# GPT-4o Mini Transcribe (fastest, lowest cost)
RubyLLM.transcribe("audio.mp3", model: "gpt-4o-mini-transcribe")

# Diarization model (identifies speakers)
RubyLLM.transcribe("meeting.wav", model: "gpt-4o-transcribe-diarize")

# Gemini (Google's multimodal)
RubyLLM.transcribe("lecture.wav", model: "gemini-2.5-flash")
```

Configure default:

```ruby
RubyLLM.configure do |config|
  config.default_transcription_model = "gpt-4o-transcribe"
end
```

## Language Hints

Improve accuracy by specifying language:

```ruby
RubyLLM.transcribe("entrevista.mp3", language: "es")
RubyLLM.transcribe("conference.mp3", language: "fr")
```

Use ISO 639-1 codes (en, es, fr, de, etc.).

## Speaker Diarization

Identify different speakers:

```ruby
transcription = RubyLLM.transcribe(
  "team-meeting.wav",
  model: "gpt-4o-transcribe-diarize"
)

transcription.segments.each do |segment|
  puts "#{segment['speaker']}: #{segment['text']}"
  puts "  (#{segment['start']}s - #{segment['end']}s)"
end
# Output:
# A: Hi everyone.
#   (0.5s - 1.2s)
# B: Happy to be here.
#   (2.8s - 3.5s)
```

### Identifying Known Speakers

Provide reference clips to map speakers to names:

```ruby
transcription = RubyLLM.transcribe(
  "team-meeting.wav",
  model: "gpt-4o-transcribe-diarize",
  speaker_names: ["Alice", "Bob"],
  speaker_references: ["alice-voice.wav", "bob-voice.wav"]
)

# Now segments use provided names
```

Speaker references accept file paths, URLs, IO objects, or ActiveStorage attachments.

> Note: Gemini models return plain text without segment metadata. Use OpenAI's diarization models for speaker labels.

## Improving Accuracy with Prompts

```ruby
RubyLLM.transcribe(
  "developer-talk.mp3",
  prompt: "Discussion about Ruby, Rails, PostgreSQL, and Redis."
)

RubyLLM.transcribe(
  "product-demo.mp3",
  prompt: "Product demo for ZyntriQix, Digique Plus, and CynapseFive."
)
```

### Gemini Prompt Tips

Gemini treats transcription like any conversation. Use `prompt:` to steer formatting:

```ruby
RubyLLM.transcribe(
  "lecture.wav",
  model: "gemini-2.5-flash",
  prompt: "Return only the verbatim transcript."
)
```

Combine with `language:` for specific locale.

## Segments and Timestamps

```ruby
transcription = RubyLLM.transcribe("interview.mp3", model: "gpt-4o-transcribe")

puts "Duration: #{transcription.duration} seconds"

transcription.segments.each do |segment|
  puts "#{segment['start']}s - #{segment['end']}s: #{segment['text']}"
end
```

## Handling Longer Files

Default timeout is 5 minutes. Increase for longer audio:

```ruby
RubyLLM.configure do |config|
  config.request_timeout = 600 # 10 minutes
end
```

The API supports files up to 25 MB. For larger files, use compressed formats (MP3, M4A) or split into chunks.

## Error Handling

```ruby
begin
  transcription = RubyLLM.transcribe("audio.mp3")
  puts transcription.text
rescue RubyLLM::BadRequestError => e
  puts "Invalid request: #{e.message}"
rescue RubyLLM::TimeoutError => e
  puts "Transcription timed out: #{e.message}"
rescue RubyLLM::Error => e
  puts "Transcription failed: #{e.message}"
end
```
