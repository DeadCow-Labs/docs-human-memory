# Memory SDK Documentation

> Store and structure human memories with AI

Memory SDK is a Python library that enables storing structured memory objects in Supabase. It leverages OpenAI's API to transform raw user messages into rich, structured memory data with semantic understanding.

## Features

- **AI-powered memory structuring**: Convert raw text into meaningful memory objects
- **Semantic extraction**: Automatically extract content summaries, insights, emotional tone, location context, and tags
- **Vector embeddings**: Store and retrieve memories based on semantic similarity
- **Supabase integration**: Persistently store memories in a scalable database

## Quick Start

```python
# Install the package
pip install memory-sdk

# Set environment variables
import os
os.environ["OPENAI_API_KEY"] = "your-openai-key"
os.environ["SUPABASE_URL"] = "your-supabase-url"
os.environ["SUPABASE_KEY"] = "your-supabase-key"

# Create a memory
from memory_sdk import MemorySDK

sdk = MemorySDK(user_id="user123")
memory = sdk.save("I had a great day working on my new project!")

print(f"Memory created: {memory.content}")
```

## Next Steps

- [Installation](guides/installation.md) - Detailed installation instructions
- [Getting Started](guides/getting-started.md) - Learn the basics of using Memory SDK
- [API Reference](api/reference.md) - Complete API documentation
- [Examples](examples/basic.md) - Code examples for common tasks 