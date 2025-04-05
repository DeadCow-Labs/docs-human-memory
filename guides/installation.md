# Installation Guide

This guide will help you install and set up the Memory SDK.

## Requirements

- Python 3.8 or later
- Dependencies:
  - `openai`
  - `supabase`
  - `python-dotenv`

## Install from PyPI

The easiest way to install Memory SDK is directly from PyPI:

```bash
pip install human-memory
```

## Install from Source

Alternatively, you can install from source:

```bash
# Clone the repository
git clone https://github.com/deadcow-labs/human-memory.git
cd human-memory

# Install the package
pip install -e .
```

## Environment Setup

The Memory SDK requires API keys for OpenAI and Supabase. You can set these as environment variables:

```bash
export OPENAI_API_KEY=your_openai_api_key
export SUPABASE_URL=your_supabase_url
export SUPABASE_KEY=your_supabase_service_key
```

Alternatively, you can use a `.env` file with the `python-dotenv` package:

```
OPENAI_API_KEY=your_openai_api_key
SUPABASE_URL=your_supabase_url
SUPABASE_KEY=your_supabase_service_key
```

Then load it in your application:

```python
from dotenv import load_dotenv

load_dotenv()  # Load environment variables from .env file
```

## Supabase Database Setup

1. Create a new Supabase project at [https://supabase.com](https://supabase.com)

2. Enable the Vector extension:
```sql
-- Enable the Vector extension
CREATE EXTENSION IF NOT EXISTS vector;
```

3. Create a `memories` table with the following schema:
```sql
CREATE TABLE memories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    content TEXT NOT NULL,
    reflection TEXT,
    embedding VECTOR(1536),
    emotional_tone TEXT,
    location TEXT,
    tags TEXT[],
    metadata JSONB DEFAULT '{}'::JSONB
);
```

4. Set up Row Level Security (RLS) policies:
```sql
-- Enable RLS
ALTER TABLE memories ENABLE ROW LEVEL SECURITY;

-- Create a policy that allows users to see only their memories
CREATE POLICY "Users can see their own memories" ON memories
    FOR SELECT
    USING (auth.uid()::text = user_id);

-- Create a policy that allows users to insert their own memories
CREATE POLICY "Users can insert their own memories" ON memories
    FOR INSERT
    WITH CHECK (auth.uid()::text = user_id);
```

5. Create an index for faster similarity searches:
```sql
-- Create an index for vector similarity searches
CREATE INDEX IF NOT EXISTS memories_embedding_idx 
ON memories 
USING ivfflat (embedding vector_cosine_ops) 
WITH (lists = 100);
```

## Next Steps

- [Getting Started](getting-started.md)
- [Configuration](configuration.md)
- [Memory Structure](memory-structure.md) 