# hackernews-py

----

# Hacker News Search Tool: Stay Updated with the Latest Discussions  
   
Are you looking to keep up with the most recent conversations on Hacker News about topics that matter to you? This simple Python script allows you to search through Hacker News stories and comments based on your query and a specified time window. Whether you're interested in the latest trends in technology, programming, startups, or any other subject, this tool helps you find relevant discussions quickly and efficiently.  
   
## Key Features  
   
- **Real-Time Search**: Fetches the latest stories and comments matching your query from Hacker News.  
- **Time Window Filtering**: Allows you to specify a time window (in hours) to focus on recent content.  
- **Detailed Results**: Provides essential information like title, author, creation time, points, number of comments, and a text snippet.  
- **Direct Access**: Includes direct links to the Hacker News posts or comments.  
- **Optional Content Retrieval**: Offers the option to retrieve and display content from URLs using Jina AI (functionality to be implemented).  
   
## Why Use This Tool?  
   
Staying informed can be time-consuming. This script simplifies the process by:  
   
- **Saving Time**: Quickly fetching and displaying relevant discussions without manual browsing.  
- **Customizable Searches**: Tailoring results based on your interests and a specific timeframe.  
- **Enhancing Engagement**: Providing quick access to discussions allows you to engage with the community promptly.  
   
## How to Use  
   
1. **Run the Script**: Execute the Python script in your terminal or command prompt:  
  
   ```bash  
   python hackernews.py  
   ```  
   
2. **Enter Your Query**: When prompted, input the keyword or phrase you're interested in (e.g., `python`, `startup`, `AI`).  
   
3. **Specify the Time Window**: Input the number of hours to define how far back the search should go (e.g., `24` for the past day).  
  
   ```  
   Enter your search query (e.g., 'python' or 'ai'): python  
   Enter the time window in hours (e.g., '24' for the last day): 24  
   ```  
   
4. **View Results**: The script will display the results, including:  
  
   - **Type**: Whether it's a story or a comment.  
   - **Title**: The title of the story or associated story for comments.  
   - **Author**: The username of the poster.  
   - **Created At**: When it was posted, along with how much time has passed since.  
   - **Points and Comments**: The number of points and comments it has received.  
   - **Text Snippet**: A brief excerpt from the story or comment.  
   - **Links**: Direct URLs to the Hacker News post or the content if available.  
   
5. **Optional Content Retrieval**: For items with an external URL, you can choose to fetch and display the content using Jina AI (note that this feature needs to be implemented).  
  
   ```  
   This item has a URL. Fetch content with Jina AI? (y/n): n  
   ```  
   
## Benefits  
   
- **Stay Informed**: Keep up with the latest discussions and trends in your areas of interest.  
- **Quick Access**: Access summaries and key information without navigating away from your terminal.  
- **Community Engagement**: Join conversations early by finding new posts and comments shortly after they're made.  
   
## Note on Content Retrieval  
   
The script includes a placeholder for a function called `get_url_text_with_jina`, intended to fetch and process content from URLs using Jina AI. This functionality is not currently implemented (`pass` is used as a placeholder). To enable this feature:  
   
- Implement the `get_url_text_with_jina` function.  
- Ensure you have any necessary dependencies installed.  
- Handle exceptions and errors appropriately to maintain the script's stability.

---------

```python
import html  
import os  
import re  
import urllib.parse  
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
  
def get_url_text_with_jina(url):  
    """  
    Fetches the main textual content of the given URL using the Jina AI API.  
    """  
    # Hardcoded Jina AI API key  
    api_key = "jina_2897436cf38044a6b4601f69eebb5237ypcyC_CdmbZABOEoE7QclK-urrPr"  
  
    # Encode the URL  
    encoded_url = urllib.parse.quote(url, safe='')  
    endpoint = f"https://r.jina.ai/{encoded_url}"  
    headers = {  
        'Authorization': f'Bearer {api_key}'  
    }  
  
    try:  
        response = requests.get(endpoint, headers=headers)  
        response.raise_for_status()  
        content = response.text  
        return content  
    except requests.exceptions.HTTPError as http_err:  
        print(f"HTTP error occurred: {http_err}")  
        print(f"Response content: {response.text}")  
        return None  
    except Exception as err:  
        print(f"An error occurred: {err}")  
        return None  
  
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
                        print("\n  --- Extracted Content ---")  
                        print(content)  
                        print("  --- End of Content ---\n")  
                    else:  
                        print("  [Content Extraction] No content retrieved.")  
                print("\n")  
    else:  
        print("No results found.")  
  
if __name__ == "__main__":  
    main()

```
```
