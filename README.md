# Memory SDK

Store and structure human memories with AI.

Memory SDK is a Python library that allows you to save, structure, and analyze human memories using AI. It leverages OpenAI's API for semantic extraction and stores memories in a Supabase database with vector embeddings.

## Features

- AI-powered memory structuring
- Semantic extraction of meaning, emotions, and themes
- Vector embeddings for similarity search
- Supabase integration for storage and retrieval
- Custom memory processors

## Installation

You can install Memory SDK directly from PyPI:

```bash
pip install human-memory
```

## Quick Start

```python
from memory_sdk import MemorySDK

# Initialize SDK
memory = MemorySDK()

# Save a memory
result = memory.save("I went for a hike at Mount Rainier yesterday. The wildflowers were in full bloom.")

# Print memory details
print(f"Memory ID: {result.id}")
print(f"Emotional tone: {result.emotional_tone}")
print(f"Tags: {result.tags}")
print(f"Reflection: {result.reflection}")
```

## Research Inspiration

This package was inspired by the following research papers:

1. **Neural Encoding for Image Recall: Human-Like Memory** (2024)  
   Virgile Foussereau, Robin Dumas  
   [arXiv:2409.11750](https://arxiv.org/abs/2409.11750)  
   
   This paper presents a method inspired by human memory processes to bridge the gap between artificial and biological memory systems. The approach focuses on encoding images to mimic the high-level information retained by the human brain, achieving impressive results on natural images while showing human-like limitations for non-natural stimuli.

2. **Recent Advancement of Emotion Cognition in Large Language Models** (2024)  
   Yuyan Chen, Yanghua Xiao  
   [arXiv:2409.13354](https://arxiv.org/html/2409.13354v1)  
   
   This survey explores emotion cognition in large language models, aligning with Ulric Neisser's cognitive psychology theory. It examines how LLMs process emotions through stages like sensation, perception, imagination, retention, recall, problem-solving, and thinking, providing insights that informed our approach to emotional memory processing.

## Next Steps

- [Installation](guides/installation.md)
- [Getting Started](guides/getting-started.md)
- [API Reference](api/overview.md)
- [Examples](examples/basic-usage.md) 