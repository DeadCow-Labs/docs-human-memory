# Basic Usage Examples

This page provides examples for common operations with the Memory SDK.

## Installation and Setup

First, install the Memory SDK and set up your environment:

```bash
# Install from PyPI
pip install memory-sdk

# Set up environment variables (or use a .env file)
export OPENAI_API_KEY=your_openai_api_key
export SUPABASE_URL=your_supabase_url
export SUPABASE_KEY=your_supabase_service_key
```

## Initializing the SDK

```python
from memory_sdk import MemorySDK

# Initialize with default settings from environment variables
memory = MemorySDK()

# Or initialize with custom settings
memory = MemorySDK(
    user_id="user123",
    openai_model="gpt-4o",
    openai_embedding_model="text-embedding-3-small"
)
```

## Saving Memories

### Saving a Single Memory

```python
# Save a simple memory
result = memory.save("I went for a hike at Mount Rainier yesterday. The wildflowers were in full bloom and the views were spectacular.")

# Print the memory ID
print(f"Memory saved with ID: {result.id}")

# The result contains all processed data
print(f"Emotional tone: {result.emotional_tone}")
print(f"Tags: {result.tags}")
print(f"Reflection: {result.reflection}")
```

### Saving Multiple Memories

```python
# Batch save multiple memories
memories = [
    "Had lunch with Sarah at Cafe Nero. We discussed our upcoming trip to Portugal.",
    "Finished reading 'Project Hail Mary' by Andy Weir. Fantastic sci-fi novel with an unexpected friendship theme.",
    "Took my dog for a walk at the park. He chased squirrels and played with other dogs."
]

results = memory.save_batch(memories)

# Process the results
for i, result in enumerate(results):
    print(f"Memory {i+1} - ID: {result.id}, Tone: {result.emotional_tone}")
```

## Retrieving Memories

### Getting a Memory by ID

```python
# Get a specific memory by ID
memory_id = "123e4567-e89b-12d3-a456-426614174000"  # Example UUID
memory_obj = memory.get(memory_id)

if memory_obj:
    print(f"Retrieved memory: {memory_obj.content}")
    print(f"Created at: {memory_obj.created_at}")
    print(f"Reflection: {memory_obj.reflection}")
else:
    print("Memory not found")
```

### Searching Memories

```python
# Text-based search
results = memory.search("hike mountains")
print(f"Found {len(results)} memories")

# Filtered search
hiking_memories = memory.search(
    query="hike",
    filter_conditions={"tags": ["outdoors", "hiking"]},
    limit=5
)

# Date range search
recent_memories = memory.search(
    date_range={"start": "2023-05-01", "end": "2023-05-31"},
    limit=10
)
```

### Finding Similar Memories

```python
# Find memories semantically similar to a query
similar = memory.find_similar(
    "Outdoor activities with friends",
    min_similarity=0.75,
    limit=5
)

print(f"Found {len(similar)} similar memories")
for mem in similar:
    print(f"- {mem.content[:100]}... (similarity: {mem.similarity_score:.2f})")
```

## Updating and Deleting Memories

### Updating a Memory

```python
# Update specific fields of a memory
memory.update(
    memory_id="123e4567-e89b-12d3-a456-426614174000",
    updates={
        "tags": ["hiking", "mountains", "photography"],
        "location": "Mount Rainier National Park"
    }
)
```

### Deleting Memories

```python
# Delete a specific memory
memory.delete("123e4567-e89b-12d3-a456-426614174000")

# Delete all memories (requires confirmation)
memory.delete_all(confirmation=True)
```

## Memory Analytics

```python
# Analyze emotional patterns
emotions = memory.analyze_emotions()
print("Emotional distribution:", emotions)

# Analyze memory topics
topics = memory.analyze_topics(min_topic_count=2)
print("Common topics:", topics)

# Analyze memory frequency by month
frequency = memory.analyze_frequency(group_by="month")
print("Monthly memory frequency:", frequency)
```

## Working with Custom Processors

```python
from memory_sdk import MemorySDK, Memory
from typing import Dict, Any

# Define a custom processor
class LocationExtractor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # This is a simplified example - in real use, you would
        # use NLP or other techniques to extract locations
        locations = []
        location_keywords = ["at", "in", "near", "to"]
        words = memory.content.split()
        
        for i, word in enumerate(words):
            if word.lower() in location_keywords and i < len(words) - 1:
                # Simplistic approach: take the next word as potential location
                potential_location = words[i+1].strip(",.;:")
                if potential_location[0].isupper():  # Check if it's capitalized
                    locations.append(potential_location)
        
        return {
            "extracted_locations": locations,
            "main_location": locations[0] if locations else None
        }

# Register the processor with the SDK
memory = MemorySDK()
memory.register_processor(LocationExtractor())

# Now when you save memories, the processor will be applied
result = memory.save("I went hiking in Yosemite last weekend.")
print("Extracted locations:", result.metadata.get("extracted_locations"))
print("Main location:", result.metadata.get("main_location"))
```

## Complete Example Application

Here's a complete example that demonstrates several features:

```python
import os
from memory_sdk import MemorySDK
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Initialize the SDK
memory = MemorySDK()

def add_memory():
    content = input("Enter your memory: ")
    result = memory.save(content)
    print(f"Memory saved! ID: {result.id}")
    print(f"Reflection: {result.reflection}")
    print(f"Emotional tone: {result.emotional_tone}")
    print(f"Tags: {result.tags}")
    return result

def search_memories():
    query = input("Enter search query: ")
    results = memory.search(query, limit=5)
    
    if not results:
        print("No memories found matching your query.")
        return
    
    print(f"Found {len(results)} memories:")
    for i, mem in enumerate(results):
        print(f"{i+1}. {mem.content[:100]}..." if len(mem.content) > 100 else mem.content)
        print(f"   Created: {mem.created_at}")
        print(f"   Tags: {mem.tags}")
        print()

def find_similar_memories():
    memory_id = input("Enter memory ID to find similar memories: ")
    mem = memory.get(memory_id)
    
    if not mem:
        print("Memory not found.")
        return
    
    print(f"Finding memories similar to: {mem.content[:100]}...")
    similar = memory.find_similar(mem.content, limit=3)
    
    print(f"Found {len(similar)} similar memories:")
    for i, sim_mem in enumerate(similar):
        if sim_mem.id != mem.id:  # Skip the original memory
            print(f"{i+1}. {sim_mem.content[:100]}..." if len(sim_mem.content) > 100 else sim_mem.content)
            print(f"   Similarity: {sim_mem.similarity_score:.2f}")
            print()

def main():
    while True:
        print("\nMemory SDK Example Application")
        print("1. Add a new memory")
        print("2. Search memories")
        print("3. Find similar memories")
        print("4. Exit")
        
        choice = input("Select an option (1-4): ")
        
        if choice == "1":
            add_memory()
        elif choice == "2":
            search_memories()
        elif choice == "3":
            find_similar_memories()
        elif choice == "4":
            print("Goodbye!")
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()
```

## Next Steps

- Explore [Memory Structure](../guides/memory-structure.md) to understand how your data is organized
- Learn about [Configuration Options](../guides/configuration.md) for customizing the SDK
- See [Advanced Examples](advanced-usage.md) for more complex scenarios 