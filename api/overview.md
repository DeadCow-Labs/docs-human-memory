# API Reference

This section provides detailed documentation for the Memory SDK API.

## Core Classes

### MemorySDK

The main class that provides access to all SDK functionality.

```python
from memory_sdk import MemorySDK

# Initialize with default settings from environment
memory = MemorySDK()

# Initialize with custom settings
memory = MemorySDK(
    user_id="user123",
    openai_model="gpt-4o",
    openai_embedding_model="text-embedding-3-small",
    memories_table="memories",
    max_tokens=300,
    temperature=0.7,
    log_level="INFO"
)
```

### Memory

Represents a single memory with all its associated data.

```python
from memory_sdk import Memory

# Memory objects are typically created by the SDK, but you can create them manually
memory_obj = Memory(
    id="123e4567-e89b-12d3-a456-426614174000",
    user_id="user123",
    created_at="2023-05-15T14:30:00Z",
    content="Had coffee with Alex at Lighthouse Cafe.",
    reflection="This memory captures a social interaction...",
    embedding=[0.023, -0.045, ...],  # Vector of floats
    emotional_tone="joyful",
    location="Lighthouse Cafe",
    tags=["coffee", "social", "Alex"]
)
```

## API Methods

### Memory Creation

| Method | Description |
|--------|-------------|
| `save(content, user_id=None, **kwargs)` | Save a single memory |
| `save_batch(contents, user_id=None, **kwargs)` | Save multiple memories in batch |

### Memory Retrieval

| Method | Description |
|--------|-------------|
| `get(memory_id, user_id=None)` | Get a memory by ID |
| `search(query=None, filter_conditions=None, date_range=None, limit=10, offset=0, user_id=None)` | Search memories with text and filters |
| `find_similar(query, min_similarity=0.7, limit=10, filter_conditions=None, date_range=None, user_id=None)` | Find semantically similar memories |
| `get_all(limit=100, offset=0, date_range=None, user_id=None)` | Get all memories with pagination |

### Memory Management

| Method | Description |
|--------|-------------|
| `update(memory_id, updates, user_id=None)` | Update specific fields of a memory |
| `delete(memory_id, user_id=None)` | Delete a memory by ID |
| `delete_all(user_id=None, confirmation=False)` | Delete all memories for a user (requires confirmation) |

### Memory Analytics

| Method | Description |
|--------|-------------|
| `analyze_emotions(date_range=None, group_by=None, user_id=None)` | Analyze emotional patterns in memories |
| `analyze_topics(min_topic_count=2, date_range=None, compare_to=None, user_id=None)` | Discover common topics and themes |
| `analyze_frequency(group_by="day", date_range=None, user_id=None)` | Analyze memory frequency over time |
| `cluster_memories(min_cluster_size=3, max_clusters=10, date_range=None, user_id=None)` | Group similar memories into clusters |
| `analyze_connections(min_connection_strength=0.5, limit=100, user_id=None)` | Find connections between memories |
| `find_connections(memory_id, min_similarity=0.6, limit=10, user_id=None)` | Find memories connected to a specific memory |

### Custom Processing

| Method | Description |
|--------|-------------|
| `register_processor(processor)` | Register a custom memory processor |
| `unregister_processor(processor_name)` | Remove a custom processor |
| `get_registered_processors()` | Get list of all registered processors |

## Memory Object Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `id` | str | Unique identifier for the memory |
| `user_id` | str | Identifier for the memory owner |
| `created_at` | datetime | Timestamp when the memory was created |
| `content` | str | The original memory text content |
| `reflection` | str | AI-generated reflection on the memory |
| `embedding` | List[float] | Vector representation for similarity search |
| `emotional_tone` | str | Identified emotional tone of the memory |
| `location` | Optional[str] | Optional location associated with the memory |
| `tags` | List[str] | Keywords or categories for the memory |
| `metadata` | Dict[str, Any] | Additional data from custom processors |

## Error Handling

The SDK uses custom exceptions to provide clear error messages:

```python
from memory_sdk import MemorySDK, MemoryError, ConfigurationError, DatabaseError

try:
    memory = MemorySDK()
    result = memory.save("This is a test memory")
except ConfigurationError as e:
    print(f"Configuration issue: {e}")
except DatabaseError as e:
    print(f"Database issue: {e}")
except MemoryError as e:
    print(f"General memory error: {e}")
```

## Detailed API Documentation

- [Memory Creation](memory-creation.md)
- [Memory Retrieval](memory-retrieval.md)
- [Memory Management](memory-management.md)
- [Memory Analytics](memory-analytics.md)
- [Custom Processing](custom-processing.md) 