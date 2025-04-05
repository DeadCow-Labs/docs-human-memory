# Configuration Guide

This guide explains how to configure the Memory SDK for optimal performance and integration with your application.

## Environment Variables

The Memory SDK relies on certain environment variables to connect to OpenAI and Supabase:

```python
# Required variables
OPENAI_API_KEY=your_openai_api_key
SUPABASE_URL=your_supabase_url
SUPABASE_KEY=your_supabase_service_key  # Anon key works, but service key has more permissions

# Optional variables
MEMORY_USER_ID=default_user_id  # Used if not explicitly provided when saving memories
OPENAI_MODEL=gpt-4o  # Default model for memory reflections and processing
OPENAI_EMBEDDING_MODEL=text-embedding-3-small  # Model for generating embeddings
MEMORIES_TABLE=memories  # The name of your Supabase table for memories
```

You can set these variables in your environment or use a `.env` file with the `python-dotenv` package.

## SDK Initialization Options

When initializing the SDK, you can customize various settings:

```python
from memory_sdk import MemorySDK

# Standard initialization with environment variables
memory = MemorySDK()

# Custom initialization
memory = MemorySDK(
    user_id="user123",  # Override MEMORY_USER_ID environment variable
    openai_model="gpt-4-turbo",  # Override OPENAI_MODEL environment variable
    openai_embedding_model="text-embedding-3-large",  # Override OPENAI_EMBEDDING_MODEL
    memories_table="custom_memories_table",  # Override MEMORIES_TABLE 
    max_tokens=300,  # Maximum tokens for reflection generation
    temperature=0.7,  # Temperature for reflection generation (0.0-1.0)
    log_level="DEBUG",  # Set logging level (DEBUG, INFO, WARNING, ERROR)
)
```

## Custom Memory Processors

You can extend the SDK with custom memory processors:

```python
from memory_sdk import MemorySDK, Memory
from typing import Dict, Any, Optional

class CustomProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Add custom processing logic
        return {
            "custom_field": "Custom analysis of: " + memory.content[:20],
            "another_field": len(memory.content.split())
        }

# Register the custom processor
memory = MemorySDK()
memory.register_processor(CustomProcessor())

# When you save a memory, your processor will be invoked
result = memory.save("This is a custom processed memory")
print(result.custom_field)  # Access your custom field
```

## Supabase Configuration

For optimal performance, consider these Supabase configurations:

### RLS Policies

If you're using Row Level Security (RLS) in Supabase, ensure your policies allow proper access:

```sql
-- Example policy that restricts users to their own memories
CREATE POLICY "Users can only access their own memories"
ON memories
FOR ALL
USING (auth.uid()::text = user_id);
```

### Indexing for Performance

Create indexes to optimize memory retrieval:

```sql
-- Index for faster similarity searches
CREATE INDEX IF NOT EXISTS memories_embedding_idx 
ON memories 
USING ivfflat (embedding vector_cosine_ops) 
WITH (lists = 100);

-- Index for text search on content
CREATE INDEX IF NOT EXISTS memories_content_idx 
ON memories 
USING GIN (to_tsvector('english', content));

-- Index for timestamp-based searches
CREATE INDEX IF NOT EXISTS memories_created_at_idx 
ON memories(created_at);
```

## Performance Considerations

- **Batch Operations**: For inserting multiple memories, use the `save_batch()` method instead of calling `save()` multiple times.
- **Query Optimization**: Limit embedding similarity searches with `limit` parameter to avoid processing unnecessary results.
- **Caching**: Consider implementing a caching layer for frequently accessed memories.

## Next Steps

- Learn about [Memory Structure](memory-structure.md) to understand how your data is organized
- Explore [Best Practices](best-practices.md) for optimal SDK usage
- Check the [API Reference](../api/overview.md) for detailed method documentation 