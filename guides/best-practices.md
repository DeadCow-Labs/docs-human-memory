# Best Practices

This guide provides recommendations for effectively using the Memory SDK in your applications.

## Memory Creation

### Writing Effective Memories

- **Be Specific**: Include concrete details in memories for better processing
- **Context Matters**: Add relevant context like when, where, and who was involved
- **Length Considerations**: While there's no strict limit, memories between 50-500 words are optimal for processing

```python
# Less effective memory
memory.save("Had coffee.")

# More effective memory
memory.save("Had coffee with Alex at Lighthouse Cafe on Tuesday morning. We discussed the upcoming project deadline and strategies for improving team communication.")
```

### Batch Processing

For efficiency when saving multiple memories, use batch operations:

```python
# Inefficient - makes multiple API calls
for text in memory_texts:
    memory.save(text)

# Efficient - makes a single API call
memory.save_batch(memory_texts)
```

## Memory Retrieval

### Semantic Search Best Practices

When using similarity search:

```python
# Set an appropriate limit to avoid processing unnecessary results
similar_memories = memory.find_similar("Looking for hiking memories", limit=5)

# Use a minimum similarity threshold (0-1 scale)
relevant_memories = memory.find_similar(
    "Important work conversations", 
    limit=10,
    min_similarity=0.75  # Only return fairly similar matches
)
```

### Combining Search Methods

Combine search methods for more targeted results:

```python
# Combine semantic search with filtering
hiking_memories = memory.find_similar(
    "Mountain hiking trips",
    filter_conditions={"tags": ["hiking", "outdoors"]},
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)
```

## Performance Optimization

### Memory Processing

- **Custom Models**: For specialized domains, use domain-specific OpenAI models
- **Embedding Dimensions**: Balance embedding quality vs storage requirements:
  - `text-embedding-3-small` (1536 dimensions): Default, good balance
  - `text-embedding-3-large` (3072 dimensions): Higher quality, larger storage

### Database Optimization

- **Indexing**: Ensure proper indexes are created for query patterns (see [Configuration Guide](configuration.md))
- **Pagination**: Always paginate large result sets

```python
# First page
results_page_1 = memory.search(query, limit=20, offset=0)

# Second page
results_page_2 = memory.search(query, limit=20, offset=20)
```

## Custom Processing

### Creating Effective Processors

Guidelines for custom memory processors:

```python
from memory_sdk import MemorySDK, Memory
from typing import Dict, Any

class LocationProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # 1. Keep processing focused and specific
        # 2. Return only necessary data
        # 3. Handle errors gracefully
        try:
            # Extract location mentions from content
            # This is a simplified example
            locations = extract_locations(memory.content)
            return {
                "detected_locations": locations,
                "primary_location": locations[0] if locations else None
            }
        except Exception as e:
            # Log the error but don't break the pipeline
            print(f"Location processing error: {e}")
            return {"detected_locations": [], "primary_location": None}

memory = MemorySDK()
memory.register_processor(LocationProcessor())
```

### Processor Order Matters

Processors are executed in the order they're registered. Arrange them logically:

```python
# Processors that provide data for later processors should come first
memory.register_processor(BasicExtractor())  # Extracts fundamental data
memory.register_processor(RelationshipAnalyzer())  # Uses BasicExtractor output
memory.register_processor(SentimentAnalyzer())  # Independent, can go anywhere
```

## Memory Analysis

### Analyzing Memory Patterns

Techniques for extracting insights:

```python
# Get memories over time for trend analysis
monthly_memories = memory.search(
    filter_conditions={},
    date_range={"start": "2023-01-01", "end": "2023-12-31"},
    group_by="month"
)

# Emotional tone distribution
emotional_distribution = memory.analyze_emotions(
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)

# Topic clustering
topics = memory.cluster_by_topic(
    min_cluster_size=5,
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)
```

## Security Considerations

### Protecting Sensitive Data

- **User Isolation**: Always use `user_id` to isolate user data
- **Data Minimization**: Only store necessary information
- **Supabase RLS**: Implement Row Level Security in Supabase (see [Configuration Guide](configuration.md))
- **PII Handling**: Consider implementing a processor to identify and handle PII

```python
class PiiRedactionProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Identify and redact PII from content before storing
        redacted_content = redact_pii(memory.content)
        # Store original content hash for verification if needed
        content_hash = hash_function(memory.content)
        
        # Update memory content with redacted version
        memory.content = redacted_content
        
        return {
            "has_pii": redacted_content != memory.content,
            "content_hash": content_hash
        }

memory = MemorySDK()
memory.register_processor(PiiRedactionProcessor())
```

## Next Steps

- Explore the [API Reference](../api/overview.md) for detailed method documentation
- Check out [Memory Structure](memory-structure.md) for details on how memories are structured
- See [Examples](../examples/basic-usage.md) for complete code samples 