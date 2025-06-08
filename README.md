# hackernews-py

```python

import html  
import os  
import re  
from datetime import datetime, timedelta, timezone  
  
import requests  
  
def search_hackernews(query: str, window_size_hours: int):  
    """  
    Searches Hacker News comments and stories for a given query within the last N hours.  
    """  
    # Calculate the threshold timestamp  
    threshold = datetime.now(timezone.utc) - timedelta(hours=window_size_hours)  
    params = {  
        "tags": "(story,comment)",  
        "query": query,  
        "numericFilters": f"created_at_i>{int(threshold.timestamp())}",  
    }  
  
    # Make a request to the Hacker News Search API  
    response = requests.get("http://hn.algolia.com/api/v1/search_by_date", params=params)  
    response.raise_for_status()  
    data = response.json()  
  
    hits = []  
    for hit in data.get("hits", []):  
        item = {  
            "id": hit.get("objectID", ""),  
            "author": hit.get("author", "N/A"),  
            "title": hit.get("title") or hit.get("story_title", "No Title"),  
            "text": hit.get("story_text") or hit.get("comment_text", ""),  
            "url": hit.get("url") or hit.get("story_url", ""),  
            "created_at": hit.get("created_at", ""),  
            "points": hit.get("points", 0),  
            "num_comments": hit.get("num_comments", 0),  
            "_tags": hit.get("_tags", []),  
        }  
        hits.append(item)  
    return hits  
  
def main():  
    """  
    Main function to interactively search Hacker News.  
    """  
    query = input("Enter your search query (e.g., 'python' or 'ai'): ")  
    window_size_hours = int(input("Enter the time window in hours (e.g., '24' for the last day): "))  
    print(f"\nSearching Hacker News for '{query}' in the last {window_size_hours} hour(s)...\n")  
  
    results = search_hackernews(query, window_size_hours)  
    print(f"Found {len(results)} result(s).\n")  
  
    if results:  
        for i, item in enumerate(results):  
            print(f"Result {i + 1}:")  
            item_type = 'Story' if 'story' in item['_tags'] else 'Comment'  
            print(f"Type: {item_type}")  
            print(f"Title: {item['title']}")  
            print(f"Author: {item['author']}")  
              
            # Parse and format the created_at timestamp  
            created_at_str = item['created_at']  
            try:  
                created_at_datetime = datetime.strptime(created_at_str, "%Y-%m-%dT%H:%M:%S.%fZ")  
                created_at_datetime = created_at_datetime.replace(tzinfo=timezone.utc)  
                time_since = datetime.now(timezone.utc) - created_at_datetime  
                hours_ago = int(time_since.total_seconds() // 3600)  
                minutes_ago = int((time_since.total_seconds() % 3600) // 60)  
                time_ago_str = f"{hours_ago}h {minutes_ago}m ago"  
                formatted_created_at = created_at_datetime.strftime('%Y-%m-%d %H:%M:%S UTC')  
            except ValueError:  
                time_ago_str = "Unknown"  
                formatted_created_at = created_at_str  
  
            print(f"Created At: {formatted_created_at} ({time_ago_str})")  
            print(f"Points: {item['points']}")  
            print(f"Number of Comments: {item['num_comments']}")  
  
            # Clean and display a snippet of the text  
            text_content = item['text']  
            if text_content:  
                text_content = html.unescape(re.sub('<[^<]+?>', '', text_content))  
                text_snippet = ' '.join(text_content.split())  
                snippet_length = 300  
                if len(text_snippet) > snippet_length:  
                    text_snippet = text_snippet[:snippet_length].rsplit(' ', 1)[0] + '...'  
                print(f"Text Snippet: {text_snippet}")  
            else:  
                print("Text Snippet: N/A")  
  
            if item_type == 'Comment':  
                # Provide a direct link to the comment  
                comment_url = f"https://news.ycombinator.com/item?id={item['id']}"  
                print(f"Hacker News Link: {comment_url}")  
            else:  
                # For stories, show the URL if available  
                print(f"URL: {item['url'] if item['url'] else 'N/A'}")  
  
            print("-" * 80)  
  
            if item['url']:  
                choice = input("  This item has a URL. Fetch content with Jina AI? (y/n): ").lower()  
                if choice == 'y':  
                    content = get_url_text_with_jina(item['url'])  
                    if content:  
                        print("\n  --- Jina AI Content ---")  
                        print(content)  
                        print("  --- End of Content ---\n")  
                    else:  
                        print("  [Jina AI] No content retrieved.")  
            print("\n")  
    else:  
        print("No results found.")  
  
# Ensure get_url_text_with_jina function is present  
def get_url_text_with_jina(url):  
    # ... (same as before)  
    pass  
  
if __name__ == "__main__":  
    main()  

```
