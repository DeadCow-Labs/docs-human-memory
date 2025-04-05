# Memory Structure

This guide explains the structure of memory objects in the Memory SDK and how they're stored in your Supabase database.

## Basic Memory Structure

Each memory in the SDK is represented by the `Memory` class with the following attributes:

```python
class Memory:
    id: str  # UUID for the memory
    user_id: str  # Identifier for the memory owner
    created_at: datetime  # When the memory was created
    content: str  # The original memory text content
    reflection: str  # AI-generated reflection on the memory
    embedding: List[float]  # Vector representation for similarity search
    emotional_tone: str  # Identified emotional tone of the memory
    location: Optional[str]  # Optional location associated with the memory
    tags: List[str]  # Keywords or categories for the memory
    # Additional fields from custom processors may be present
```

## Database Schema

The memories are stored in a Supabase table with the following schema:

```sql
CREATE TABLE memories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    content TEXT NOT NULL,
    reflection TEXT,
    embedding VECTOR(1536),  -- Dimensionality depends on embedding model
    emotional_tone TEXT,
    location TEXT,
    tags TEXT[],
    metadata JSONB DEFAULT '{}'::JSONB  -- Stores additional processed data
);
```

The `metadata` field is a flexible JSONB column that stores outputs from custom processors and additional attributes that aren't part of the standard schema.

## Memory Processing Flow

When you save a memory using `memory.save()`, the following happens:

1. **Core Field Creation**: A Memory object is initialized with your provided content and user_id
2. **AI Processing**: The content is processed to generate:
   - A reflection using OpenAI's models (introspective analysis of the memory)
   - Vector embeddings for similarity search
   - Emotional tone analysis
   - Tags extraction

3. **Custom Processing**: Any registered custom processors are run
4. **Database Storage**: The memory is saved to your Supabase database
5. **Return**: The complete Memory object is returned with all processed fields

## Understanding Memory Fields

### Content

The raw text of the memory, exactly as provided by the user or application.

### Reflection

An AI-generated introspection on the memory, providing context, insights, or patterns that might not be explicitly stated in the content. For example:

```
Content: "I spent the afternoon at the beach watching the sunset. The colors were incredible."
Reflection: "This memory captures a moment of peace and appreciation for natural beauty. 
The user seems to have taken time for mindfulness and enjoying the present moment, 
which is associated with improved well-being and reduced stress."
```

### Embedding

A high-dimensional vector (array of floating-point numbers) that represents the semantic meaning of the memory. This enables:

- Similarity search (finding memories with similar meaning)
- Clustering memories by themes
- Identifying patterns across memories

The dimensionality of the embedding depends on the model used (default: 1536 dimensions with text-embedding-3-small).

### Emotional Tone

A categorization of the primary emotional tone of the memory, such as:
- Joyful
- Sad
- Anxious
- Nostalgic
- Proud
- Frustrated
- Neutral

### Tags

Automatically extracted keywords or themes from the memory, useful for categorization and filtering. For example:
```
Content: "Had lunch with Sarah at the new Italian restaurant downtown. The pasta was amazing!"
Tags: ["food", "restaurants", "social", "friends", "italian cuisine"]
```

### Metadata

A flexible JSONB object that stores:

1. Additional attributes from the memory processing pipeline
2. Outputs from custom processors
3. Any application-specific data you want to associate with the memory

Accessing metadata in Python:
```python
memory = memory_sdk.save("This is a memory with metadata")
print(memory.metadata.get("custom_field"))  # Access a specific metadata field
```

## Next Steps

- Learn about [Configuration](configuration.md) options for the SDK
- Explore [Best Practices](best-practices.md) for working with memory structures
- Check the [API Reference](../api/overview.md) for detailed information on memory manipulation 