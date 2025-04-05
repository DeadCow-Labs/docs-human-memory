# Analytics Guide

This guide explains how to use the Memory SDK's analytics capabilities to derive insights from your memory data.

## Overview

The Memory SDK provides built-in analytics functions that help you:

1. Understand emotional patterns in memories
2. Identify common themes and topics
3. Track memory trends over time
4. Discover connections between memories

## Emotional Analysis

### Analyzing Emotional Patterns

```python
from memory_sdk import MemorySDK

memory = MemorySDK()

# Get emotional distribution across all memories
emotion_distribution = memory.analyze_emotions()
print(emotion_distribution)
# Output: {'joyful': 0.35, 'neutral': 0.25, 'nostalgic': 0.15, 'sad': 0.10, ...}

# Analyze emotions for a specific date range
summer_emotions = memory.analyze_emotions(
    date_range={"start": "2023-06-01", "end": "2023-08-31"}
)

# Compare emotional patterns over different periods
emotion_trends = memory.analyze_emotions(
    group_by="month",
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)
```

### Visualizing Emotional Data

While the SDK doesn't include visualization tools directly, you can easily use the analytics output with libraries like Matplotlib or Plotly:

```python
import matplotlib.pyplot as plt

# Basic emotion pie chart
emotions = memory.analyze_emotions()
plt.figure(figsize=(10, 6))
plt.pie(emotions.values(), labels=emotions.keys(), autopct='%1.1f%%')
plt.title('Emotional Distribution of Memories')
plt.show()

# Emotion trends over time
monthly_emotions = memory.analyze_emotions(group_by="month")
# Plot as stacked bar chart, line chart, etc.
```

## Topic Analysis

### Discovering Memory Themes

```python
# Get common topics across all memories
topics = memory.analyze_topics(min_topic_count=3)
print(topics)
# Output: [{'name': 'Work', 'count': 15, 'subtopics': ['meetings', 'projects', ...]}, ...]

# Find emerging topics in recent memories
recent_topics = memory.analyze_topics(
    date_range={"start": "2023-10-01", "end": "2023-12-31"},
    compare_to={"start": "2023-07-01", "end": "2023-09-30"}
)
print(recent_topics['emerging'])  # Topics that have increased in frequency
print(recent_topics['declining'])  # Topics that have decreased in frequency
```

### Topic Clustering

Group similar memories into clusters based on their content:

```python
# Cluster memories by semantic similarity
clusters = memory.cluster_memories(
    min_cluster_size=3,
    max_clusters=10,
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)

for i, cluster in enumerate(clusters):
    print(f"Cluster {i+1}: {cluster['label']}")
    print(f"  Size: {len(cluster['memories'])}")
    print(f"  Top terms: {', '.join(cluster['top_terms'])}")
    print(f"  Sample memory: {cluster['memories'][0].content[:100]}...")
```

## Temporal Analysis

### Tracking Memory Patterns Over Time

```python
# Memory frequency over time
time_distribution = memory.analyze_frequency(
    group_by="week",
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)

# Find most active days/times
activity_patterns = memory.analyze_frequency(
    group_by="hour_of_day",
    date_range={"start": "2023-01-01", "end": "2023-12-31"}
)
```

### Memory Trends and Seasonality

```python
# Detect seasonality in memory topics
seasonal_patterns = memory.analyze_topics(
    group_by="month",
    date_range={"start": "2022-01-01", "end": "2023-12-31"}
)

# Check which topics appear more in certain seasons
summer_topics = memory.analyze_topics(
    filter_conditions={"month": [6, 7, 8]},  # Summer months
    date_range={"start": "2022-01-01", "end": "2023-12-31"}
)
```

## Network Analysis

### Analyzing Memory Connections

```python
# Find connections between memories based on shared entities/concepts
memory_network = memory.analyze_connections(
    min_connection_strength=0.5,
    limit=100
)

# For a specific memory, find related memories
memory_id = "some-memory-uuid"
related = memory.find_connections(memory_id, min_similarity=0.6)
```

## Custom Analytics with SQL

For advanced analytics, you can access the underlying Supabase database directly:

```python
from memory_sdk import MemorySDK
import os
from supabase import create_client

# Initialize SDK
memory = MemorySDK()

# Access Supabase directly for custom queries
supabase = create_client(
    os.environ.get("SUPABASE_URL"),
    os.environ.get("SUPABASE_KEY")
)

# Example: Complex query not available through SDK
result = supabase.table("memories") \
    .select("created_at, emotional_tone, content") \
    .gte("created_at", "2023-01-01") \
    .lt("created_at", "2024-01-01") \
    .eq("user_id", memory.user_id) \
    .execute()

# Process results
memories_by_date = {}
for item in result.data:
    date = item['created_at'].split('T')[0]
    if date not in memories_by_date:
        memories_by_date[date] = []
    memories_by_date[date].append(item)
```

## Exporting Analytics Data

```python
# Export emotion analysis to CSV
emotion_data = memory.analyze_emotions(group_by="day")
emotion_df = pd.DataFrame(emotion_data)
emotion_df.to_csv("emotion_analysis.csv", index=False)

# Export topic distribution
topic_data = memory.analyze_topics()
topic_df = pd.DataFrame(topic_data)
topic_df.to_csv("topic_analysis.csv", index=False)
```

## Next Steps

- Learn about [Memory Structure](memory-structure.md) to understand the data being analyzed
- Explore [Best Practices](best-practices.md) for optimizing your analytics workflows
- Check the [API Reference](../api/analytics.md) for detailed analytics method documentation 