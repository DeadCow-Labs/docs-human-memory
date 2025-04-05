# Getting Started

This guide will walk you through the basics of using Memory SDK to store structured memories in Supabase.

## Prerequisites

Before you start, make sure you have:

1. Installed Memory SDK
2. Set up your environment variables for OpenAI and Supabase
3. Created the `memories` table in your Supabase database

If you haven't done these steps, refer to the [Installation Guide](installation.md).

## Basic Usage

### Initialize the SDK

First, create an instance of the `MemorySDK` class with a user ID:

```python
from memory_sdk import MemorySDK

# The user ID can be any string that identifies your user
user_id = "user_123"
sdk = MemorySDK(user_id)
```

The SDK will automatically load your configuration from environment variables.

### Save a Memory

To create and save a memory, use the `save()` method with a raw text message:

```python
raw_message = """
Today I spent some time working on my portfolio website. I made good progress 
on the design and I'm happy with how it's coming along. I still need to finish 
the project showcase section, but I'm confident I can complete it by next week.
"""

memory = sdk.save(raw_message)
```

This will:
1. Process the raw message using OpenAI's API to extract structured data
2. Create a vector embedding for the content
3. Save the memory to your Supabase database

### Access Memory Data

The returned memory object contains all the structured data:

```python
# Access memory fields
print(f"Memory ID: {memory.id}")
print(f"Content: {memory.content}")
print(f"Reflection: {memory.reflection}")
print(f"Emotional Tone: {memory.emotional_tone}")
print(f"Location Type: {memory.location.type}")
print(f"Location Name: {memory.location.name}")
print(f"Tags: {', '.join(memory.tags)}")
print(f"Created At: {memory.created_at}")
```

## Understanding the Memory Structure

Each memory object contains:

- **id**: A unique UUID for the memory
- **user_id**: The user ID you provided when initializing the SDK
- **created_at**: A UTC timestamp when the memory was created
- **content**: A concise summary of the message
- **reflection**: An insight or observation about the user from the message
- **embedding**: A vector representation of the content for similarity searches
- **emotional_tone**: The detected emotional tone (e.g., "hopeful", "anxious")
- **location**: A context object with:
  - **type**: "mental", "physical", "digital", or "described"
  - **name**: A description of the location
- **tags**: A list of relevant topics or themes

## Next Steps

Now that you understand the basics of Memory SDK, you can:

- Explore the [API Reference](../api/reference.md) for detailed documentation
- Check out [Examples](../examples/basic.md) for more usage scenarios
- Learn about [Memory Structure](memory-structure.md) in depth 