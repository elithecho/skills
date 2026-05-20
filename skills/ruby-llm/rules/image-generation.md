# Image Generation

Generate images from text descriptions using AI models like DALL-E 3 and Imagen.

## Basic Image Generation

```ruby
image = RubyLLM.paint("A photorealistic image of a red panda coding Ruby on a laptop")

# For models returning a URL:
if image.url
  puts "Image URL: #{image.url}"
end

# For models returning Base64 (like Imagen):
if image.base64?
  puts "MIME Type: #{image.mime_type}"
  puts "Data size: ~#{image.data.length} bytes"
end

# Revised prompt (some models revise for better results)
if image.revised_prompt
  puts "Revised Prompt: #{image.revised_prompt}"
end

puts "Model Used: #{image.model_id}"
```

## Choosing Models

```ruby
# GPT-Image-1
image = RubyLLM.paint("Impressionist painting of a Parisian cafe", model: "gpt-image-1")

# Google's Imagen 3
image = RubyLLM.paint("Cyberpunk city at night", model: "imagen-3.0-generate-002")

# Custom model
image = RubyLLM.paint(
  "A sunset",
  model: "my-custom-image-model",
  provider: :openai,
  assume_model_exists: true
)
```

Configure default:

```ruby
RubyLLM.configure do |config|
  config.default_image_model = "gpt-image-1"
end
```

## Image Sizes

Some models like DALL-E 3 allow specifying dimensions:

```ruby
# Square (default)
image = RubyLLM.paint("a fluffy white cat", size: "1024x1024")

# Wide landscape
image = RubyLLM.paint("panoramic mountain landscape", size: "1792x1024")

# Tall portrait
image = RubyLLM.paint("a knight before castle gate", size: "1024x1792")
```

## Working with Generated Images

### Accessing Image Data

```ruby
# image.url    - URL for providers like OpenAI
# image.data   - Base64-encoded data for providers like Google
# image.mime_type - e.g., "image/png"
# image.base64? - Boolean for encoding type
```

### Saving Images Locally

```ruby
image = RubyLLM.paint("A steampunk mechanical owl")
saved_path = image.save("steampunk_owl.png")
puts "Image saved to #{saved_path}"
```

### Getting Raw Image Blob

```ruby
image = RubyLLM.paint("Abstract geometric patterns")
image_blob = image.to_blob

puts "Image blob size: #{image_blob.bytesize} bytes"
```

### Rails Active Storage Integration

```ruby
class Product < ApplicationRecord
  has_one_attached :generated_image
end

def generate_and_attach_image(product, prompt)
  image = RubyLLM.paint(prompt)
  filename = "#{product.slug}-#{Time.current.to_i}.png"
  image_io = StringIO.new(image.to_blob)

  product.generated_image.attach(
    io: image_io,
    filename: filename,
    content_type: image.mime_type || 'image/png'
  )

  product.update(
    image_prompt: prompt,
    image_revised_prompt: image.revised_prompt,
    image_model: image.model_id
  )
end
```

## Prompt Engineering

```ruby
# Simple - often generic results
image1 = RubyLLM.paint("dog")

# Detailed - better results
image2 = RubyLLM.paint(
  "A photorealistic golden retriever puppy playing fetch " \
  "in a sunny park, shallow depth of field, captured with a DSLR camera."
)

# Specify style
image3 = RubyLLM.paint("Majestic mountain range, oil painting in style of Bob Ross")
```

**Tips:**
- **Subject**: Be specific ("red panda" vs "animal")
- **Action/Setting**: Describe what's happening
- **Style**: Specify ("photorealistic", "watercolor", "pixel art")
- **Details**: Add adjectives ("fluffy", "ancient", "glowing")
- **Composition**: Mention framing ("close-up", "wide angle")
- **Lighting**: Describe ("soft morning light", "dramatic sunset")
- **Mood**: Convey ("serene", "chaotic", "mysterious")

## Error Handling

```ruby
begin
  image = RubyLLM.paint("Your prompt here")
  puts "Image URL: #{image.url}"
rescue RubyLLM::BadRequestError => e
  # Often indicates content policy violation
  puts "Generation failed: #{e.message}"
rescue RubyLLM::Error => e
  puts "Error: #{e.message}"
end
```

## Content Safety

AI image services have content filters. Avoid generating:
- Violent or hateful imagery
- Sexually explicit content
- Images of real people without consent
- Copyrighted characters

## Performance Considerations

- Image generation takes 5-20 seconds
- **Use Background Jobs**: Don't block web requests
- **Configure Timeouts**: Set appropriate network timeouts
- **Caching**: Store generated images rather than regenerating
