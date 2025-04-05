# Advanced Usage Examples

This page provides more complex examples for advanced users of the Memory SDK.

## Custom Memory Processing Pipeline

This example demonstrates how to create a custom processing pipeline with multiple processors:

```python
from memory_sdk import MemorySDK, Memory
from typing import Dict, Any, List
import re
from datetime import datetime

# 1. Location extractor
class LocationProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Simple location extraction logic (in production, use NLP)
        locations = []
        location_keywords = ["at", "in", "near", "to", "from"]
        words = memory.content.split()
        
        for i, word in enumerate(words):
            if word.lower() in location_keywords and i < len(words) - 1:
                potential_location = words[i+1].strip(",.;:")
                if potential_location[0].isupper():
                    locations.append(potential_location)
        
        return {
            "extracted_locations": locations,
            "primary_location": locations[0] if locations else None
        }

# 2. People extractor
class PeopleProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Simple person extraction (in production, use NER)
        people = []
        people_indicators = ["with", "and", "met", "saw", "talked to", "spoke with"]
        
        for indicator in people_indicators:
            pattern = fr"{indicator} (\w+)"
            matches = re.finditer(pattern, memory.content, re.IGNORECASE)
            for match in matches:
                person = match.group(1)
                # Check if it's capitalized (likely a name)
                if person[0].isupper():
                    people.append(person)
        
        return {
            "people_mentioned": list(set(people)),  # Remove duplicates
            "people_count": len(set(people))
        }

# 3. Time reference processor
class TimeProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Extract time references
        time_references = []
        time_patterns = [
            r"yesterday",
            r"last (week|month|year|night|day)",
            r"on (Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday)",
            r"in (January|February|March|April|May|June|July|August|September|October|November|December)",
            r"(\d{1,2})(st|nd|rd|th)? of (January|February|March|April|May|June|July|August|September|October|November|December)",
            r"(\d{4})"  # Year
        ]
        
        for pattern in time_patterns:
            matches = re.finditer(pattern, memory.content, re.IGNORECASE)
            for match in matches:
                time_references.append(match.group(0))
        
        # Simple attempt to resolve relative dates
        resolved_date = None
        if time_references:
            # This is very simplified - in production use a proper date parser
            now = datetime.now()
            if any(ref.lower() == "yesterday" for ref in time_references):
                resolved_date = now.replace(day=now.day-1).strftime("%Y-%m-%d")
            # Add more date resolution logic here
        
        return {
            "time_references": time_references,
            "resolved_date": resolved_date
        }

# 4. Sentiment detail processor
class DetailedSentimentProcessor:
    def process(self, memory: Memory) -> Dict[str, Any]:
        # This would typically use a sentiment analysis API or model
        # Here we're just doing a simple keyword search
        positive_words = ["happy", "joy", "excited", "wonderful", "great", "amazing", "love", "enjoyed"]
        negative_words = ["sad", "angry", "upset", "disappointed", "terrible", "hate", "disliked"]
        
        pos_count = sum(1 for word in positive_words if word in memory.content.lower())
        neg_count = sum(1 for word in negative_words if word in memory.content.lower())
        
        if pos_count > neg_count:
            polarity = "positive"
            score = min(0.5 + (pos_count - neg_count) * 0.1, 1.0)
        elif neg_count > pos_count:
            polarity = "negative"
            score = max(-0.5 - (neg_count - pos_count) * 0.1, -1.0)
        else:
            polarity = "neutral"
            score = 0.0
        
        return {
            "sentiment": {
                "polarity": polarity,
                "score": score,
                "positive_words": [word for word in positive_words if word in memory.content.lower()],
                "negative_words": [word for word in negative_words if word in memory.content.lower()]
            }
        }

# Set up the SDK with the custom processors
memory = MemorySDK()
memory.register_processor(LocationProcessor())
memory.register_processor(PeopleProcessor())
memory.register_processor(TimeProcessor())
memory.register_processor(DetailedSentimentProcessor())

# Save a memory with the custom processing pipeline
result = memory.save(
    "Yesterday I had lunch with Alex at Central Park. We enjoyed the beautiful weather and discussed our upcoming project."
)

# Access the custom processor results
print("Memory saved with ID:", result.id)
print("Locations:", result.metadata.get("extracted_locations"))
print("People:", result.metadata.get("people_mentioned"))
print("Time references:", result.metadata.get("time_references"))
print("Sentiment:", result.metadata.get("sentiment"))
```

## Advanced Memory Querying

This example demonstrates advanced memory searching and analytics:

```python
from memory_sdk import MemorySDK
import matplotlib.pyplot as plt
import pandas as pd
from datetime import datetime, timedelta

# Initialize the SDK
memory = MemorySDK()

# Function to generate date range
def date_range(start_date, end_date, step=timedelta(days=1)):
    for n in range(int((end_date - start_date).days) + 1):
        yield start_date + timedelta(n)

# 1. Complex search with multiple filters
def complex_search():
    # Find memories about work meetings in the last month with positive sentiment
    end_date = datetime.now()
    start_date = end_date - timedelta(days=30)
    
    results = memory.search(
        query="meeting",
        filter_conditions={
            "tags": ["work", "meeting"],
            "emotional_tone": "positive"
        },
        date_range={
            "start": start_date.strftime("%Y-%m-%d"),
            "end": end_date.strftime("%Y-%m-%d")
        },
        limit=10
    )
    
    print(f"Found {len(results)} positive work meeting memories")
    for mem in results:
        print(f"- {mem.content[:100]}...")
        print(f"  Date: {mem.created_at}")
        print(f"  Emotion: {mem.emotional_tone}")
        print()

# 2. Combining semantic search with filters
def combined_search():
    # Find memories semantically similar to travel plans but only in cities
    similar = memory.find_similar(
        "Travel plans and itineraries",
        min_similarity=0.7,
        filter_conditions={"tags": ["city", "travel"]},
        limit=5
    )
    
    print(f"Found {len(similar)} memories about city travel plans:")
    for mem in similar:
        print(f"- {mem.content[:100]}...")
        print(f"  Similarity: {mem.similarity_score:.2f}")
        print(f"  Tags: {mem.tags}")
        print()

# 3. Advanced analytics - Emotional trends over time
def emotional_trends():
    # Analyze emotions over the past 3 months, grouped by week
    end_date = datetime.now()
    start_date = end_date - timedelta(days=90)
    
    emotion_data = memory.analyze_emotions(
        date_range={
            "start": start_date.strftime("%Y-%m-%d"),
            "end": end_date.strftime("%Y-%m-%d")
        },
        group_by="week"
    )
    
    # Convert to DataFrame for easier manipulation
    df = pd.DataFrame(emotion_data)
    
    # Plot the results
    if not df.empty:
        df.plot(kind='line', figsize=(12, 6))
        plt.title('Emotional Trends Over Time')
        plt.xlabel('Week')
        plt.ylabel('Emotion Frequency')
        plt.grid(True)
        plt.tight_layout()
        plt.savefig('emotional_trends.png')
        print("Emotional trends chart saved to emotional_trends.png")

# 4. Memory network analysis
def memory_connections():
    # Find connections between memories
    network = memory.analyze_connections(
        min_connection_strength=0.6,
        limit=100
    )
    
    # Print the strongest connections
    strongest = sorted(network['connections'], 
                        key=lambda x: x['strength'], 
                        reverse=True)[:5]
    
    print("Strongest memory connections:")
    for conn in strongest:
        memory1 = memory.get(conn['source_id'])
        memory2 = memory.get(conn['target_id'])
        print(f"Connection strength: {conn['strength']:.2f}")
        print(f"Memory 1: {memory1.content[:50]}...")
        print(f"Memory 2: {memory2.content[:50]}...")
        print(f"Connection type: {conn['connection_type']}")
        print()

# Run the examples
complex_search()
combined_search()
emotional_trends()
memory_connections()
```

## Integration with External Services

This example shows how to integrate Memory SDK with other services:

```python
from memory_sdk import MemorySDK, Memory
from typing import Dict, Any
import requests
import os
from datetime import datetime
import json

# Initialize the SDK
memory = MemorySDK()

# 1. Weather data integration
class WeatherProcessor:
    def __init__(self, api_key):
        self.api_key = api_key
    
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Extract location from memory or metadata
        location = None
        
        # Try to get location from metadata if LocationProcessor ran before
        if memory.metadata and "primary_location" in memory.metadata:
            location = memory.metadata["primary_location"]
        
        # If no location found, return empty result
        if not location:
            return {"weather_data": None}
        
        # Get weather data for the memory date
        # Note: In production, you'd need to handle date parsing more robustly
        try:
            # Get the date from memory.created_at or current date
            if hasattr(memory, 'created_at') and memory.created_at:
                date = memory.created_at.split('T')[0]
            else:
                date = datetime.now().strftime("%Y-%m-%d")
            
            # Call weather API (this is simplified)
            response = requests.get(
                f"https://api.weatherapi.com/v1/history.json",
                params={
                    "key": self.api_key,
                    "q": location,
                    "dt": date
                }
            )
            
            if response.status_code == 200:
                weather_data = response.json()
                return {
                    "weather_data": {
                        "location": location,
                        "date": date,
                        "condition": weather_data["forecast"]["forecastday"][0]["day"]["condition"]["text"],
                        "temp_c": weather_data["forecast"]["forecastday"][0]["day"]["avgtemp_c"],
                        "precipitation_mm": weather_data["forecast"]["forecastday"][0]["day"]["totalprecip_mm"]
                    }
                }
            else:
                return {"weather_data": None, "error": f"API error: {response.status_code}"}
                
        except Exception as e:
            return {"weather_data": None, "error": str(e)}

# 2. Calendar event integration
class CalendarProcessor:
    def __init__(self, calendar_api_token):
        self.api_token = calendar_api_token
    
    def process(self, memory: Memory) -> Dict[str, Any]:
        # This would connect to a calendar API (Google Calendar, Outlook, etc.)
        # Here we're simulating the response
        
        # Extract date references from the memory content or metadata
        date_reference = None
        if memory.metadata and "resolved_date" in memory.metadata:
            date_reference = memory.metadata["resolved_date"]
        
        if not date_reference:
            # Simple date extraction (would be more robust in production)
            if "yesterday" in memory.content.lower():
                date_reference = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d")
            elif "today" in memory.content.lower():
                date_reference = datetime.now().strftime("%Y-%m-%d")
        
        if not date_reference:
            return {"calendar_events": []}
        
        # Simulate getting calendar events for the date
        # In production, you would call the actual calendar API
        mock_events = [
            {
                "title": "Team Meeting",
                "start": f"{date_reference}T10:00:00",
                "end": f"{date_reference}T11:00:00",
                "attendees": ["Alex", "Sarah", "Michael"]
            },
            {
                "title": "Project Review",
                "start": f"{date_reference}T14:00:00",
                "end": f"{date_reference}T15:30:00",
                "attendees": ["John", "Emma"]
            }
        ]
        
        return {
            "calendar_events": mock_events,
            "event_date": date_reference
        }

# 3. Social media post integration
class SocialMediaProcessor:
    def __init__(self, api_credentials):
        self.credentials = api_credentials
    
    def process(self, memory: Memory) -> Dict[str, Any]:
        # Check if this memory should be shared on social media
        if "share" not in memory.content.lower() and "post" not in memory.content.lower():
            return {"social_media_share": False}
        
        # Prepare a post based on the memory content
        # In production, you would use NLP to create a good summary
        max_length = 280  # Twitter-like length
        content = memory.content
        
        if len(content) > max_length:
            # Simple truncation - in production use proper summarization
            content = content[:max_length-3] + "..."
        
        # Extract hashtags from memory tags
        hashtags = []
        if hasattr(memory, 'tags') and memory.tags:
            hashtags = ["#" + tag.replace(" ", "") for tag in memory.tags]
        
        # In production, you would actually post to social media here
        # This is just a simulation
        post = {
            "content": content,
            "hashtags": hashtags,
            "created_at": datetime.now().isoformat(),
            "platform": "Twitter"  # or whatever platform you're targeting
        }
        
        return {
            "social_media_share": True,
            "post": post,
            "post_url": "https://twitter.com/user/status/123456789"  # Simulated URL
        }

# Register the processors with the SDK
# In a real application, you would get these API keys from environment variables
weather_api_key = os.environ.get("WEATHER_API_KEY", "your_api_key")
calendar_api_token = os.environ.get("CALENDAR_API_TOKEN", "your_token")
social_media_credentials = json.loads(os.environ.get("SOCIAL_MEDIA_CREDENTIALS", "{}"))

memory.register_processor(WeatherProcessor(weather_api_key))
memory.register_processor(CalendarProcessor(calendar_api_token))
memory.register_processor(SocialMediaProcessor(social_media_credentials))

# Test with a memory
memory_content = "Yesterday I went for a walk in Central Park. The weather was beautiful and I met Sarah for coffee afterwards. We should share these photos on social media!"

result = memory.save(memory_content)

# Print the results
print(f"Memory saved with ID: {result.id}")
print("\nStandard SDK processing:")
print(f"Emotional tone: {result.emotional_tone}")
print(f"Tags: {result.tags}")

print("\nCustom processing results:")
if "weather_data" in result.metadata:
    weather = result.metadata["weather_data"]
    if weather:
        print(f"Weather: {weather['condition']}, {weather['temp_c']}°C in {weather['location']}")
    else:
        print("Weather data not available")

if "calendar_events" in result.metadata:
    events = result.metadata["calendar_events"]
    if events:
        print(f"\nCalendar events on {result.metadata.get('event_date')}:")
        for event in events:
            print(f"- {event['title']} ({event['start'].split('T')[1][:5]} - {event['end'].split('T')[1][:5]})")
            print(f"  Attendees: {', '.join(event['attendees'])}")
    else:
        print("\nNo calendar events found")

if "social_media_share" in result.metadata:
    if result.metadata["social_media_share"]:
        post = result.metadata["post"]
        print(f"\nShared on social media ({post['platform']}):")
        print(f"Content: {post['content']}")
        if post['hashtags']:
            print(f"Hashtags: {' '.join(post['hashtags'])}")
        print(f"URL: {result.metadata['post_url']}")
    else:
        print("\nNot shared on social media")
```

## Building a Memory Journal Application

This example shows how to build a simple journal application using the Memory SDK:

```python
import tkinter as tk
from tkinter import ttk, scrolledtext, messagebox
import threading
from datetime import datetime
from memory_sdk import MemorySDK
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

class MemoryJournalApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Memory Journal")
        self.root.geometry("900x600")
        
        # Initialize SDK
        self.memory = MemorySDK()
        
        # Create notebook for tabs
        self.notebook = ttk.Notebook(root)
        self.notebook.pack(fill='both', expand=True, padx=10, pady=10)
        
        # Create tabs
        self.create_journal_tab()
        self.create_memories_tab()
        self.create_analytics_tab()
        
        # Load memories
        self.load_memories()
    
    def create_journal_tab(self):
        journal_frame = ttk.Frame(self.notebook)
        self.notebook.add(journal_frame, text="Journal Entry")
        
        # Date field
        date_frame = ttk.Frame(journal_frame)
        date_frame.pack(fill='x', padx=10, pady=5)
        
        ttk.Label(date_frame, text="Date:").pack(side='left')
        self.date_var = tk.StringVar(value=datetime.now().strftime("%Y-%m-%d"))
        ttk.Entry(date_frame, textvariable=self.date_var, width=15).pack(side='left', padx=5)
        
        # Location field
        loc_frame = ttk.Frame(journal_frame)
        loc_frame.pack(fill='x', padx=10, pady=5)
        
        ttk.Label(loc_frame, text="Location:").pack(side='left')
        self.location_var = tk.StringVar()
        ttk.Entry(loc_frame, textvariable=self.location_var, width=30).pack(side='left', padx=5)
        
        # Memory content
        ttk.Label(journal_frame, text="Memory:").pack(anchor='w', padx=10, pady=5)
        self.memory_text = scrolledtext.ScrolledText(journal_frame, height=15)
        self.memory_text.pack(fill='both', expand=True, padx=10, pady=5)
        
        # Save button
        self.save_button = ttk.Button(journal_frame, text="Save Memory", command=self.save_memory)
        self.save_button.pack(pady=10)
        
        # Status message
        self.status_var = tk.StringVar()
        ttk.Label(journal_frame, textvariable=self.status_var).pack(pady=5)
    
    def create_memories_tab(self):
        memories_frame = ttk.Frame(self.notebook)
        self.notebook.add(memories_frame, text="My Memories")
        
        # Search frame
        search_frame = ttk.Frame(memories_frame)
        search_frame.pack(fill='x', padx=10, pady=5)
        
        ttk.Label(search_frame, text="Search:").pack(side='left')
        self.search_var = tk.StringVar()
        ttk.Entry(search_frame, textvariable=self.search_var, width=30).pack(side='left', padx=5)
        ttk.Button(search_frame, text="Search", command=self.search_memories).pack(side='left', padx=5)
        ttk.Button(search_frame, text="Refresh", command=self.load_memories).pack(side='left', padx=5)
        
        # Memories list
        columns = ('date', 'preview', 'emotion', 'tags')
        self.memory_tree = ttk.Treeview(memories_frame, columns=columns, show='headings')
        self.memory_tree.heading('date', text='Date')
        self.memory_tree.heading('preview', text='Preview')
        self.memory_tree.heading('emotion', text='Emotion')
        self.memory_tree.heading('tags', text='Tags')
        
        self.memory_tree.column('date', width=100)
        self.memory_tree.column('preview', width=400)
        self.memory_tree.column('emotion', width=100)
        self.memory_tree.column('tags', width=200)
        
        self.memory_tree.pack(fill='both', expand=True, padx=10, pady=5)
        self.memory_tree.bind('<Double-1>', self.view_memory)
        
        # Scrollbar for the treeview
        scrollbar = ttk.Scrollbar(memories_frame, orient=tk.VERTICAL, command=self.memory_tree.yview)
        self.memory_tree.configure(yscroll=scrollbar.set)
        scrollbar.place(relx=1, rely=0, relheight=1, anchor='ne')
    
    def create_analytics_tab(self):
        analytics_frame = ttk.Frame(self.notebook)
        self.notebook.add(analytics_frame, text="Analytics")
        
        # Controls frame
        controls_frame = ttk.Frame(analytics_frame)
        controls_frame.pack(fill='x', padx=10, pady=5)
        
        ttk.Label(controls_frame, text="Analysis Type:").pack(side='left')
        self.analysis_var = tk.StringVar(value="emotions")
        analysis_combo = ttk.Combobox(controls_frame, textvariable=self.analysis_var, 
                                     values=["emotions", "topics", "frequency"])
        analysis_combo.pack(side='left', padx=5)
        
        ttk.Button(controls_frame, text="Generate", command=self.generate_analytics).pack(side='left', padx=5)
        
        # Plot frame
        self.plot_frame = ttk.Frame(analytics_frame)
        self.plot_frame.pack(fill='both', expand=True, padx=10, pady=5)
    
    def save_memory(self):
        content = self.memory_text.get('1.0', 'end-1c')
        if not content.strip():
            messagebox.showerror("Error", "Memory content cannot be empty")
            return
        
        # Disable save button and show status
        self.save_button.config(state='disabled')
        self.status_var.set("Saving memory...")
        
        # Get optional metadata
        location = self.location_var.get().strip()
        
        # Create a dict for any additional fields
        additional = {}
        if location:
            additional['location'] = location
        
        # Run in a separate thread to keep UI responsive
        threading.Thread(target=self._save_memory_thread, 
                         args=(content, additional)).start()
    
    def _save_memory_thread(self, content, additional):
        try:
            # Save to SDK
            result = self.memory.save(content, **additional)
            
            # Update UI in the main thread
            self.root.after(0, lambda: self._memory_saved(result))
        except Exception as e:
            # Handle error in the main thread
            self.root.after(0, lambda: self._memory_save_error(str(e)))
    
    def _memory_saved(self, result):
        # Clear input fields
        self.memory_text.delete('1.0', 'end')
        self.location_var.set("")
        
        # Update status
        self.status_var.set(f"Memory saved! Emotional tone: {result.emotional_tone}")
        
        # Re-enable save button
        self.save_button.config(state='normal')
        
        # Refresh memories list
        self.load_memories()
    
    def _memory_save_error(self, error_msg):
        self.status_var.set(f"Error: {error_msg}")
        self.save_button.config(state='normal')
        messagebox.showerror("Error Saving Memory", error_msg)
    
    def load_memories(self):
        # Clear current items
        for item in self.memory_tree.get_children():
            self.memory_tree.delete(item)
        
        # Show loading status
        self.status_var.set("Loading memories...")
        
        # Load in a separate thread
        threading.Thread(target=self._load_memories_thread).start()
    
    def _load_memories_thread(self):
        try:
            # Get memories from SDK
            memories = self.memory.get_all(limit=100)
            
            # Update UI in the main thread
            self.root.after(0, lambda: self._display_memories(memories))
        except Exception as e:
            self.root.after(0, lambda: self._load_error(str(e)))
    
    def _display_memories(self, memories):
        # Sort by date, newest first
        memories.sort(key=lambda x: x.created_at, reverse=True)
        
        # Add to treeview
        for mem in memories:
            date = mem.created_at.split('T')[0] if hasattr(mem, 'created_at') else ""
            preview = mem.content[:50] + "..." if len(mem.content) > 50 else mem.content
            emotion = mem.emotional_tone if hasattr(mem, 'emotional_tone') else ""
            tags = ", ".join(mem.tags) if hasattr(mem, 'tags') and mem.tags else ""
            
            self.memory_tree.insert('', 'end', iid=mem.id, values=(date, preview, emotion, tags))
        
        self.status_var.set(f"Loaded {len(memories)} memories")
    
    def _load_error(self, error_msg):
        self.status_var.set(f"Error loading memories: {error_msg}")
        messagebox.showerror("Error", f"Failed to load memories: {error_msg}")
    
    def search_memories(self):
        query = self.search_var.get().strip()
        if not query:
            self.load_memories()
            return
        
        # Show searching status
        self.status_var.set(f"Searching for '{query}'...")
        
        # Search in a separate thread
        threading.Thread(target=self._search_thread, args=(query,)).start()
    
    def _search_thread(self, query):
        try:
            # Search using SDK
            results = self.memory.search(query, limit=100)
            
            # Update UI in main thread
            self.root.after(0, lambda: self._display_memories(results))
        except Exception as e:
            self.root.after(0, lambda: self._load_error(str(e)))
    
    def view_memory(self, event):
        # Get selected item
        item_id = self.memory_tree.selection()[0]
        memory_id = item_id
        
        # Show loading status
        self.status_var.set(f"Loading memory details...")
        
        # Load in separate thread
        threading.Thread(target=self._view_memory_thread, args=(memory_id,)).start()
    
    def _view_memory_thread(self, memory_id):
        try:
            # Get memory details
            mem = self.memory.get(memory_id)
            
            # Show in main thread
            self.root.after(0, lambda: self._show_memory_details(mem))
        except Exception as e:
            self.root.after(0, lambda: self._load_error(str(e)))
    
    def _show_memory_details(self, memory):
        # Create a new window to show details
        details_window = tk.Toplevel(self.root)
        details_window.title("Memory Details")
        details_window.geometry("600x500")
        
        # Memory date and metadata
        metadata_frame = ttk.Frame(details_window)
        metadata_frame.pack(fill='x', padx=10, pady=5)
        
        date = memory.created_at.split('T')[0] if hasattr(memory, 'created_at') else "Unknown"
        ttk.Label(metadata_frame, text=f"Date: {date}").pack(anchor='w')
        
        if hasattr(memory, 'emotional_tone') and memory.emotional_tone:
            ttk.Label(metadata_frame, text=f"Emotional Tone: {memory.emotional_tone}").pack(anchor='w')
        
        if hasattr(memory, 'location') and memory.location:
            ttk.Label(metadata_frame, text=f"Location: {memory.location}").pack(anchor='w')
        
        if hasattr(memory, 'tags') and memory.tags:
            ttk.Label(metadata_frame, text=f"Tags: {', '.join(memory.tags)}").pack(anchor='w')
        
        # Memory content
        ttk.Label(details_window, text="Memory:").pack(anchor='w', padx=10, pady=5)
        content_text = scrolledtext.ScrolledText(details_window, height=10, wrap=tk.WORD)
        content_text.pack(fill='both', expand=True, padx=10, pady=5)
        content_text.insert('1.0', memory.content)
        content_text.config(state='disabled')
        
        # Reflection (if available)
        if hasattr(memory, 'reflection') and memory.reflection:
            ttk.Label(details_window, text="Reflection:").pack(anchor='w', padx=10, pady=5)
            reflection_text = scrolledtext.ScrolledText(details_window, height=5, wrap=tk.WORD)
            reflection_text.pack(fill='both', expand=True, padx=10, pady=5)
            reflection_text.insert('1.0', memory.reflection)
            reflection_text.config(state='disabled')
    
    def generate_analytics(self):
        analysis_type = self.analysis_var.get()
        
        # Clear current plot
        for widget in self.plot_frame.winfo_children():
            widget.destroy()
        
        # Show loading status
        self.status_var.set(f"Generating {analysis_type} analysis...")
        
        # Generate in separate thread
        threading.Thread(target=self._analytics_thread, args=(analysis_type,)).start()
    
    def _analytics_thread(self, analysis_type):
        try:
            result = None
            
            if analysis_type == "emotions":
                result = self.memory.analyze_emotions()
            elif analysis_type == "topics":
                result = self.memory.analyze_topics(min_topic_count=1)
            elif analysis_type == "frequency":
                result = self.memory.analyze_frequency(group_by="month")
            
            # Update UI in main thread
            self.root.after(0, lambda: self._display_analytics(analysis_type, result))
        except Exception as e:
            self.root.after(0, lambda: self._analytics_error(str(e)))
    
    def _display_analytics(self, analysis_type, data):
        # Create figure and axis
        fig, ax = plt.subplots(figsize=(8, 5))
        
        if analysis_type == "emotions":
            # Sort by frequency
            emotions = {k: v for k, v in sorted(data.items(), key=lambda item: item[1], reverse=True)}
            ax.bar(emotions.keys(), emotions.values())
            ax.set_title("Emotional Distribution of Memories")
            ax.set_ylabel("Frequency")
            plt.xticks(rotation=45, ha='right')
            
        elif analysis_type == "topics":
            # Extract topic names and counts
            topics = [item["name"] for item in data]
            counts = [item["count"] for item in data]
            
            # Sort by count
            sorted_indices = sorted(range(len(counts)), key=lambda i: counts[i], reverse=True)
            topics = [topics[i] for i in sorted_indices]
            counts = [counts[i] for i in sorted_indices]
            
            ax.bar(topics, counts)
            ax.set_title("Topic Distribution")
            ax.set_ylabel("Frequency")
            plt.xticks(rotation=45, ha='right')
            
        elif analysis_type == "frequency":
            # Convert to dates and counts
            dates = list(data.keys())
            counts = list(data.values())
            
            ax.plot(dates, counts, marker='o')
            ax.set_title("Memory Frequency by Month")
            ax.set_xlabel("Month")
            ax.set_ylabel("Count")
            plt.xticks(rotation=45)
        
        plt.tight_layout()
        
        # Embed plot in UI
        canvas = FigureCanvasTkAgg(fig, master=self.plot_frame)
        canvas.draw()
        canvas.get_tk_widget().pack(fill='both', expand=True)
        
        self.status_var.set(f"{analysis_type.capitalize()} analysis complete")
    
    def _analytics_error(self, error_msg):
        self.status_var.set(f"Error generating analytics: {error_msg}")
        messagebox.showerror("Analytics Error", error_msg)

# Run the application
if __name__ == "__main__":
    root = tk.Tk()
    app = MemoryJournalApp(root)
    root.mainloop()
```

## Next Steps

- Explore [API Reference](../api/overview.md) for detailed method documentation
- Learn about [Best Practices](../guides/best-practices.md) for working with the SDK
- Check [Analytics Guide](../guides/analytics.md) for more information on memory analysis techniques 