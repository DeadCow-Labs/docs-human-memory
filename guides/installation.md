# Installation

## Requirements

Memory SDK requires Python 3.8 or later and depends on the following packages:
- openai >= 1.0.0
- supabase >= 2.0.0
- python-dotenv >= 1.0.0

## Install from Source

Currently, the package can be installed directly from the source repository:

```bash
# Clone the repository
git clone https://github.com/yourusername/memory-sdk.git
cd memory-sdk

# Install the package
pip install -e .
```

## Environment Setup

Memory SDK requires the following environment variables:

```bash
# OpenAI API Key
export OPENAI_API_KEY=your_openai_api_key

# Supabase Configuration
export SUPABASE_URL=your_supabase_project_url
export SUPABASE_KEY=your_supabase_api_key
```

Alternatively, you can use a `.env` file with the python-dotenv package:

```
OPENAI_API_KEY=your_openai_api_key
SUPABASE_URL=your_supabase_project_url
SUPABASE_KEY=your_supabase_api_key
```

Then load it in your application:

```python
from dotenv import load_dotenv

load_dotenv()  # Load environment variables from .env file
```

## Supabase Database Setup

Memory SDK requires a Supabase project with a properly configured `memories` table:

1. Create a new Supabase project at [supabase.com](https://supabase.com/)
2. Enable the Vector extension in your Supabase database by running:

```sql
-- Enable the vector extension
create extension if not exists vector;
```

3. Create the memories table with the following schema:

```sql
create table
  public.memories (
    id uuid primary key,
    user_id text not null,
    created_at timestamp with time zone not null,
    content text not null,
    reflection text not null,
    embedding vector(1536) not null,
    emotional_tone text not null,
    location jsonb not null,
    tags text[] not null
  );
```

4. Set up appropriate Row Level Security (RLS) policies to secure your data.

5. Create an index for faster similarity searches (optional but recommended):

```sql
create index on memories 
using ivfflat (embedding vector_cosine_ops)
with (lists = 100);
```

Now you're ready to use Memory SDK with your Supabase database! 