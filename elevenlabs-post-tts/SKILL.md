---
name: elevenlabs-post-tts
description: Convert Astro blog posts to audio using the ElevenLabs text-to-speech API. Use this skill whenever the user wants to: generate an audio version of a blog post written in Astro (Markdown/MDX); create an MP3 to embed in a blog page; turn their blog content into a listenable narration; add a "listen to this post" feature to their Astro site. Trigger when the user says things like "convert this post to audio", "generate audio for my blog post", "make an MP3 for this article", "add audio to my Astro post", "use ElevenLabs for my blog", or shares a .md/.mdx file and wants it narrated. Always use this skill when ElevenLabs + Astro blog or blog post are mentioned together.
---

# ElevenLabs — Astro Blog Post to Audio

Converts an Astro blog post (Markdown or MDX) into a high-quality `.mp3` file using the ElevenLabs TTS API, ready to embed or download.

---

## Prerequisites

The user must provide their **ElevenLabs API key**. If not yet provided, ask for it before proceeding. Keys are found at: https://elevenlabs.io/app/settings/api-keys

---

## Workflow

### Step 1 — Get the blog post content

The user may provide the post in any of these ways:
- Paste the raw Markdown/MDX directly in chat
- Upload a `.md` or `.mdx` file (check `/mnt/user-data/uploads/`)
- Share the text of the post inline

### Step 2 — Extract clean text from Markdown/MDX

Before sending to ElevenLabs, strip everything that should NOT be read aloud:

```python
import re

def extract_clean_text(md_content: str) -> str:
    # Remove YAML frontmatter (--- ... ---)
    md_content = re.sub(r'^---[\s\S]*?---\n', '', md_content, flags=re.MULTILINE)

    # Remove MDX/JSX component tags (e.g. <MyComponent ... />)
    md_content = re.sub(r'<[A-Z][^>]*/?>', '', md_content)
    md_content = re.sub(r'</[A-Z][^>]*>', '', md_content)

    # Remove HTML tags
    md_content = re.sub(r'<[^>]+>', '', md_content)

    # Remove code blocks (``` ... ```)
    md_content = re.sub(r'```[\s\S]*?```', '[code block]', md_content)

    # Remove inline code
    md_content = re.sub(r'`[^`]+`', '', md_content)

    # Remove image syntax ![alt](url)
    md_content = re.sub(r'!\[.*?\]\(.*?\)', '', md_content)

    # Convert links [text](url) -> just the text
    md_content = re.sub(r'\[([^\]]+)\]\([^\)]+\)', r'\1', md_content)

    # Remove markdown heading symbols (#, ##, etc.) but keep the text
    md_content = re.sub(r'^#+\s+', '', md_content, flags=re.MULTILINE)

    # Remove bold/italic markers
    md_content = re.sub(r'(\*\*|__)(.*?)\1', r'\2', md_content)
    md_content = re.sub(r'(\*|_)(.*?)\1', r'\2', md_content)

    # Remove horizontal rules
    md_content = re.sub(r'^---+$', '', md_content, flags=re.MULTILINE)

    # Remove emojis (they get narrated awkwardly)
    md_content = re.sub(r'[^\x00-\x7F]+', '', md_content)

    # Collapse multiple blank lines
    md_content = re.sub(r'\n{3,}', '\n\n', md_content)

    return md_content.strip()
```

> **Always show the user a preview** of the cleaned text (first 300 chars) and confirm before generating audio.

### Step 3 — Select voice

Default voices (use one unless the user specifies):
- `EXAVITQu4vr4xnSDxMaL` — **Sarah** (female, warm, clear — great for personal/lifestyle blogs)
- `JBFqnCBsd6RMkjVDRZzb` — **George** (male, confident — great for tech/dev blogs)
- `cgSgspJ2msm6clMCkdW9` — **Jessica** (female, expressive — great for storytelling/opinion posts)

If the user wants to browse voices: https://elevenlabs.io/voice-library

### Step 4 — Generate audio

```python
import requests

ELEVENLABS_API_KEY = "<user_api_key>"
VOICE_ID = "EXAVITQu4vr4xnSDxMaL"  # default: Sarah
TEXT = clean_text  # output from extract_clean_text()

url = f"https://api.elevenlabs.io/v1/text-to-speech/{VOICE_ID}"

headers = {
    "xi-api-key": ELEVENLABS_API_KEY,
    "Content-Type": "application/json"
}

payload = {
    "text": TEXT,
    "model_id": "eleven_multilingual_v2",
    "voice_settings": {
        "stability": 0.5,
        "similarity_boost": 0.75
    }
}

response = requests.post(url, headers=headers, json=payload)

if response.status_code == 200:
    slug = "my-post"  # derive from frontmatter title if available
    output_path = f"/mnt/user-data/outputs/{slug}.mp3"
    with open(output_path, "wb") as f:
        f.write(response.content)
    print(f"Audio saved: {output_path}")
else:
    print(f"Error {response.status_code}: {response.text}")
```

### Step 5 — Deliver + suggest Astro integration

After generating, use `present_files` to deliver the MP3. Then suggest how to embed it in the Astro post (see **Astro Integration** section below).

---

## Handling Long Posts (>5,000 chars)

ElevenLabs has a **5,000 character limit** per request. Long blog posts must be chunked:

```python
def split_text(text: str, max_chars: int = 4800) -> list[str]:
    """Split at paragraph boundaries to keep natural flow."""
    paragraphs = text.split('\n\n')
    chunks, current = [], ""
    for para in paragraphs:
        if len(current) + len(para) + 2 <= max_chars:
            current += ("\n\n" if current else "") + para
        else:
            if current:
                chunks.append(current)
            current = para
    if current:
        chunks.append(current)
    return chunks
```

Merge resulting MP3 chunks with `pydub`:

```python
from pydub import AudioSegment

combined = AudioSegment.empty()
for chunk_path in chunk_paths:
    combined += AudioSegment.from_mp3(chunk_path)

combined.export(f"/mnt/user-data/outputs/{slug}.mp3", format="mp3")
```

Install if needed: `pip install pydub --break-system-packages`

---

## Output Naming

Derive the filename from the post's frontmatter `slug` or `title`:
- Post titled "Getting Started with Astro" -> `getting-started-with-astro.mp3`
- If no title available -> `blog-post-audio.mp3`

Always save to `/mnt/user-data/outputs/`.

---

## Astro Integration

After delivering the MP3, suggest the user place it in their Astro project and embed it like this:

**Option A — Native HTML audio player (simplest):**
```astro
---
// In your blog post layout or .mdx file
const audioSrc = "/audio/my-post.mp3"; // place MP3 in public/audio/
---

<audio controls class="w-full my-4">
  <source src={audioSrc} type="audio/mpeg" />
  Your browser does not support audio playback.
</audio>
```

**Option B — Reusable Astro component:**
```astro
---
// src/components/AudioPlayer.astro
const { src, title } = Astro.props;
---

<div class="audio-player">
  <p class="text-sm text-gray-500">Listen to this post</p>
  <audio controls src={src}>
    Your browser does not support audio.
  </audio>
</div>
```

Then in your MDX post:
```mdx
import AudioPlayer from '../../components/AudioPlayer.astro';

<AudioPlayer src="/audio/my-post.mp3" title="My Post Title" />
```

**Where to put the MP3 file:**
- Place in `public/audio/your-post-slug.mp3`
- Reference as `/audio/your-post-slug.mp3` in the `src` attribute

---

## Voice Settings Reference

| Setting | Range | Recommended for Blog |
|---|---|---|
| `stability` | 0-1 | `0.5` for personal blogs; `0.65` for technical/dev blogs |
| `similarity_boost` | 0-1 | `0.75` default; `0.8` for more consistent narration |
| `speed` | 0.7-1.2 | `1.0` default; `0.9` for dense technical content |

---

## Model Reference

| Model | Best For |
|---|---|
| `eleven_multilingual_v2` | Default — best quality, supports Spanish, English, and 30+ languages |
| `eleven_turbo_v2_5` | Faster generation for quick previews |

Use `eleven_multilingual_v2` by default. If the blog post is in **Spanish**, this model handles it natively — no language parameter needed.

---

## Error Handling

| Code | Meaning | Action |
|---|---|---|
| `401` | Invalid API key | Ask user to check their key at elevenlabs.io |
| `422` | Validation error | Text may be empty or voice_id invalid |
| `429` | Quota exceeded | User may have hit their monthly character limit |
| `400` | Bad request | Check payload structure and content type |

---

## Example Interaction

**User:** "Here's my Astro blog post about CSS Grid. Convert it to audio for embedding."

**Claude should:**
1. Ask for API key if not provided
2. Extract and clean the Markdown (strip frontmatter, code blocks, etc.)
3. Show a 300-char preview of the cleaned text for confirmation
4. Generate the MP3 using Sarah or George voice
5. Return the `.mp3` file via `present_files`
6. Suggest the Astro embed snippet with the correct filename
