# YouTube Data Collection Tutorial

In this post, we will cover a quick guide on how to collect closed caption text from YouTube videos. With a dataset of texts like this, you can do sentiment analysis, NER, and more. More specifically, possible use cases might be NER on products, sentiment analysis from YouTube reviews, or text summarization from longer videos or podcasts.

## Getting Started

To get started, you will need to set up a GCP account and enable the YouTube API. Don't worry, this is completely free. Next install the youtube_transcript_api and googleapiclient packages. Then we can import our packages. Here are some helpful links:

 - [Link to GCP](https://www.googleadservices.com/pagead/aclk?sa=L&ai=DChcSEwja_5Wc-MCHAxXVNAgFHc9-BYUYABAAGgJtZA&co=1&gclid=CjwKCAjwzIK1BhAuEiwAHQmU3vk5uqNzHfodDgbSC_hrHwJQhVCTfaCFSc9_aJuobJzORCHHnbZjYRoC9KgQAvD_BwE&ohost=www.google.com&cid=CAESV-D2BvCv4Vg-5POaQmkHNct-fRrsasCKE_3oE-T523U2gB3MG2AU0qvCxqv7JrYuWQySy9gwCQMqspjBwHAGDclsNeD0IS3C8GpA1ZYiDBDhGo80UZsN1A&sig=AOD64_3WT4PhRUoMkElbjsYKflW_g5dIaQ&q&adurl&ved=2ahUKEwjf9JCc-MCHAxWFnokEHQorDj8Q0Qx6BAgQEAE)
 - [Link to youtube_transcript_api](https://pypi.org/project/youtube-transcript-api/)
 - [Link to googleapiclient](https://pypi.org/project/google-api-python-client/)


```python
# all the packages we will need
import os
from dotenv import load_dotenv
from youtube_transcript_api import YouTubeTranscriptApi
from googleapiclient.discovery import build
```

Next you'll need your GCP key. You can just paste it here, or create a .env file in the same directory and put it in there - up to you.


```python
load_dotenv()
youtube_api_key = os.getenv('GCP_YOUTUBE_API_KEY')
```

## Fetching Closed Captions Text

Now let's get to the YouTube API part. First we'll use the imported build function which let's us create an object to interact with specified Google APIs. Obviously we will put YouTube in as an arguement. We'll then use version 3 alongside your API key.


```python
youtube = build('youtube', 'v3', developerKey=youtube_api_key)
```

To use this youtube object, you'll actualy send in a query, as if you're searching youtube from the search bar as you normally would. We'll actually get the number of top results from this search query, where max_results will specify the exact number of results to get. Also, we'll set it to only search for videos (no playlists, channels, etc.), and also get the id and snippet (snippet contains info like title, description, etc.).


```python
search_query = "Top video games of 2023"
max_results = 5

search_response = youtube.search().list(
    q=search_query,
    type='video',
    part='id, snippet',
    maxResults=max_results
).execute()
```

Now let's explore when this result is. Naturally, it is a list of 5 items, 1 for each result (since we set max results to 5). So let's just look at the first item in the list.


```python
search_response_ex = search_response.get('items', [])[0]
search_response_ex
```




    {'kind': 'youtube#searchResult',
     'etag': 'cAWTfIz-k4IQLn33ru7_-03bFE8',
     'id': {'kind': 'youtube#video', 'videoId': 'sXnoQdA6cYM'},
     'snippet': {'publishedAt': '2023-11-23T00:00:32Z',
      'channelId': 'UCaWd5_7JhbQBe4dknZhsHJg',
      'title': 'Top 10 Best Video Games of 2023',
      'description': "2023 has been another great year for video games! Welcome to WatchMojo, and today we're counting down our picks for the ...",
      'thumbnails': {'default': {'url': 'https://i.ytimg.com/vi/sXnoQdA6cYM/default.jpg',
        'width': 120,
        'height': 90},
       'medium': {'url': 'https://i.ytimg.com/vi/sXnoQdA6cYM/mqdefault.jpg',
        'width': 320,
        'height': 180},
       'high': {'url': 'https://i.ytimg.com/vi/sXnoQdA6cYM/hqdefault.jpg',
        'width': 480,
        'height': 360}},
      'channelTitle': 'WatchMojo.com',
      'liveBroadcastContent': 'none',
      'publishTime': '2023-11-23T00:00:32Z'}}



So we have lot's of information here, and for your own purposes you can take whatever you need, but this tutorial is about getting the cc text. This means that the `id` is what we need, and more specifically, the `videoId` item inside this dictionary.


```python
video_id_ex = search_response_ex['id']['videoId']
video_id_ex
```




    'sXnoQdA6cYM'



Now that we have the videoId we can use the `YouTubeTranscriptApi` function to get the cc text as follows.


```python
transcript_ex = YouTubeTranscriptApi.get_transcript(video_id_ex)
transcript_ex[0:5] # this list is long, so just showing the first 5 elements to see what it's like
```




    [{'text': 'link you must find me welcome to watch',
      'start': 0.839,
      'duration': 6.241},
     {'text': "Mojo and today we're counting down our",
      'start': 4.52,
      'duration': 4.56},
     {'text': 'picks for the greatest video games to be',
      'start': 7.08,
      'duration': 4.88},
     {'text': 'released over the past year still angry',
      'start': 9.08,
      'duration': 4.84},
     {'text': 'after all these years Marco you should',
      'start': 11.96,
      'duration': 3.52}]



This json output might not be what you expected, but the function breaks up the cc text and couples it with the start time and duration. The use case here might be to help with subtitle generation, indexing, or analytics. In any case, we can easily put it back together though by concatenating everything.


```python
#set it to be an empty string
cc_text_ex = ""

#add each piece iteratively but with a space in between each element
for entry in transcript_ex:
    cc_text_ex += ' ' + entry['text']

#also let's remove new lines just in case
cc_text_ex.replace('\n', ' ')

#the above method results in an empty space as the first character, just removing it here
cc_text_ex = cc_text_ex[1:]
cc_text_ex
```




    "link you must find me welcome to watch Mojo and today we're counting down our picks for the greatest video games to be released over the past year still angry after ..."



There, that's it! We have our cc_text, but let's put this into a function.


```python
def fetch_captions(video_id):
    """
    Get the closed captions. 

    Parameters:
    - video_id (str): The video id which is obtained in search_videos.
    
    Returns:
    String: Closed caption text of a youtube video
    """
    try:
        # Retrieve the transcript for the video
        transcript = YouTubeTranscriptApi.get_transcript(video_id)

        cc_text = ""

        # Concatenate the transcript text
        for entry in transcript:
            cc_text += ' ' + entry['text']
            
        cc_text = cc_text.replace('\n', ' ')
        return cc_text[1:]

    except Exception as e:
        print(f"An error occurred: {str(e)}")
        return f"An error occurred: {str(e)}"
```

## Collecting a Full Dataset

We can go a step further still. Let's build a function to collect a dataset of most relevant information you might want from a youtube video. You can edit this to get what you like, but we'll just get the `video_id`, `title`, `channel_title`, `video_link`, and `cc_text`.


```python
video_id_ex = search_response_ex['id']['videoId']
title_ex = search_response_ex['snippet']['title']
video_link_ex = f'https://www.youtube.com/watch?v={video_id_ex}'
channel_title_ex = search_response_ex['snippet']['channelTitle']
cc_text_ex = fetch_captions(video_id_ex)

#excluding printing the cc_text here since we already saw it
for item in [video_id_ex, title_ex, video_link_ex, channel_title_ex]:
    print(item)
```

    sXnoQdA6cYM
    Top 10 Best Video Games of 2023
    https://www.youtube.com/watch?v=sXnoQdA6cYM
    WatchMojo.com
    

Now we can iterate over each item in our search results to get our full dataset.


```python
search_query_results = search_response.get('items', [])

videos_data = []
for item in search_query_results:
    video_id = item['id']['videoId']
    title= item['snippet']['title']
    video_link = f'https://www.youtube.com/watch?v={video_id_ex}'
    channel_title = item['snippet']['channelTitle']
    cc_text = fetch_captions(video_id)

    videos_data.append({
        'video_id': video_id,
        'title': title,
        'video_link': video_link,
        'channel_name': channel_title,
        'cc_text': cc_text
    })

#just showing the first couple of resulting items
videos_data[0:2]
```




    [{'video_id': 'sXnoQdA6cYM',
      'title': 'Top 10 Best Video Games of 2023',
      'video_link': 'https://www.youtube.com/watch?v=sXnoQdA6cYM',
      'channel_name': 'WatchMojo.com',
      'cc_text': "link you must find me welcome to watch Mojo and today we're counting down our picks for the greatest video games to be released over the past year still angry after ..."},
     {'video_id': '99kjo-MKmvU',
      'title': 'BEST GAMES OF 2023',
      'video_link': 'https://www.youtube.com/watch?v=sXnoQdA6cYM',
      'channel_name': 'gameranx',
      'cc_text': '(ambient music) - Hey folks, it\'s me, Jake Baldino, and no video game shirt today. We are all business here to talk about the Game of the Year Awards for ...'}]



And that's it, now you can make this into a dataset if you like, or edit it to get some other info as needed. As a bonus, let's turn this into a class.

## Bonus: YouTube Data Retriever Class

Before we create this class, I want to add one little thing: something I found useful was filtering youtube videos via their titles. So say you are searching for reviews of a product, like the latest MacBook Pro. Well you want reviews of just that video, but sometimes there might be videos in your query results that are comparison reviews, of the latest version against an older one. You can easily filter these out by disregarding videos with certain strings in their title, in this case "vs" is what you would ignore.  

Let's run through an example with the search query `Surface Laptop 7 Review`.


```python
search_query = "Surface Laptop 7 Review"
max_results = 12

search_response = youtube.search().list(
    q=search_query,
    type='video',
    part='id, snippet',
    maxResults=max_results
).execute()

search_query_results = search_response.get('items', [])
```

Let's check to see if there's any comparison reviews.


```python
for item in search_query_results:
    title = item['snippet']['title']
    print(title)
```

    Surface Laptop 7 X Elite Honest Review after 2 Weeks!
    Surface Laptop 7: Don’t Buy the WORST Model!
    Microsoft Surface Laptop 7 15 Inch Review
    Surface Laptop 7 vs MacBook Air M3 - The ULTIMATE Battle
    Surface Pro 11 vs Surface Laptop 7 Review &amp; Comparison
    ARMed &amp; Ready; but Not for All... Surface Laptop 7 Review
    Switching to Surface Laptop 7 From A MacBook User - The BEST &amp; WORST Parts
    1 month later: Surface Laptop 7 with Snapdragon X Elite
    Surface Pro 8 i7 Gaming - Once Human
    Surface Laptop 7 After 1 Week: Growing Pains!
    Surface Laptop 7 After 1 Week - Should Apple Worry!?
    Surface Laptop 7 Review: The New Daily Driver!
    

Indeed, you can this it in the title: `Surface Pro 11 vs Surface Laptop 7 Review &amp; Comparison`, which has both the vs and comparison words in it. So we can filter for this.


```python
#filter for variations of vs since it is case sensitive, we'll leave comparison out for now
filtered_strings = ["VS", "vs", "Vs"]

videos_data = []
for item in search_query_results:
    video_id = item['id']['videoId']
    title= item['snippet']['title']
    video_link = f'https://www.youtube.com/watch?v={video_id}'
    channel_title = item['snippet']['channelTitle']

    if not any(s in title for s in filtered_strings):
        #this function takes time to run so make sure not to use it, 
        #or anything with a lengthy runtime, before the filter 
        cc_text = fetch_captions(video_id)

        videos_data.append({
            'video_id': video_id,
            'title': title,
            'video_link': video_link,
            'channel_name': channel_title,
            'cc_text': cc_text
        })
```


```python
for vid in videos_data:
    print(vid["title"])
```

    Surface Laptop 7 X Elite Honest Review after 2 Weeks!
    Surface Laptop 7: Don’t Buy the WORST Model!
    Microsoft Surface Laptop 7 15 Inch Review
    ARMed &amp; Ready; but Not for All... Surface Laptop 7 Review
    Switching to Surface Laptop 7 From A MacBook User - The BEST &amp; WORST Parts
    1 month later: Surface Laptop 7 with Snapdragon X Elite
    Surface Pro 8 i7 Gaming - Once Human
    Surface Laptop 7 After 1 Week: Growing Pains!
    Surface Laptop 7 After 1 Week - Should Apple Worry!?
    Surface Laptop 7 Review: The New Daily Driver!
    

And there we have it, a simple filter for your consideration, should you use this function. Below is the class with all of the above implemented within it. You'll likely want to add some functions to it, but feel free to steal and modify as needed!


```python
class YouTubeAPIData:    
    def __init__(self, api_key):
        self.api_key = api_key
        self.youtube = build('youtube', 'v3', developerKey=api_key)
        
    def fetch_youtube_data(self, search_query, filtered_strings=[], max_results=5):
        """
        Search for YouTube videos based on a given query and retrieve additional information including closed captions.

        Parameters:
        - search_query (str): The search query used to find relevant videos on YouTube.
        - filtered_strings (list): Strings to filter video results by i needed. Defaults to an empty list. 
        - max_results (int): The maximum number of videos to retrieve. Defaults to 5.

        Returns:
        List[dict]: A list of dictionaries, each containing information about a video, including:
            - 'video_id' (str): The unique identifier for the video.
            - 'title' (str): The title of the video.
            - 'video_link' (str): The YouTube link to the video.
            - 'channel_name' (str): The name of the channel that uploaded the video.
            - 'cc_text' (str): The closed captions text for the video.

        Note:
        - The 'cc_text' field may contain an empty string if closed captions are not available. Mend that as needed.
        """       
        
        search_response = self.youtube.search().list(
            q=search_query,
            type='video',
            part='id, snippet',
            maxResults=max_results
        ).execute()
        
        videos_data = []
        for result in search_response.get('items', []):
            video_id = result['id']['videoId']
            title = result['snippet']['title']
            video_link = f'https://www.youtube.com/watch?v={video_id}'
            channel_name = result['snippet']['channelTitle']

            # Check and remove unwanted titles
            if not any(s in title for s in filtered_strings):
                cc_text = self.fetch_captions(video_id)
                videos_data.append({
                    'video_id': video_id,
                    'title': title,
                    'video_link': video_link,
                    'channel_name': channel_name,
                    'cc_text': cc_text
                })

        return videos_data
    
    def fetch_captions(self, video_id):
        """
        Get the closed captions. 

        Parameters:
        - video_id (str): The video id which is obtained in search_videos.
        
        Returns:
        String: Closed caption text of a youtube video
        """
        try:
            # Retrieve the transcript for the video
            transcript = YouTubeTranscriptApi.get_transcript(video_id)

            cc_text = ""

            # Concatenate the transcript text
            for entry in transcript:
                cc_text += ' ' + entry['text']
                
            cc_text = cc_text.replace('\n', ' ')
            return cc_text

        except Exception as e:
            print(f"An error occurred: {str(e)}")
            return f"An error occurred: {str(e)}"

```


```python
youtube_api_data = YouTubeAPIData(youtube_api_key)
#just showing first 3 values
youtube_api_data.fetch_youtube_data("Surface Laptop 7 Review", filtered_strings=["VS", "vs", "Vs"], max_results=12)[0:3]
```




    [{'video_id': '_BFF-arkGWY',
      'title': 'Surface Laptop 7 X Elite Honest Review after 2 Weeks!',
      'video_link': 'https://www.youtube.com/watch?v=_BFF-arkGWY',
      'channel_name': 'Max Tech',
      'cc_text': " I've been testing and using the new Surface laptop 7 with qualcomm's X Elite chip for the past 2 weeks now and I have to say that it is my favorite X Elite laptop and ..."},
     {'video_id': '1l6xqZexZVg',
      'title': 'Surface Laptop 7: Don’t Buy the WORST Model!',
      'video_link': 'https://www.youtube.com/watch?v=1l6xqZexZVg',
      'channel_name': 'Just Josh',
      'cc_text': " if you're planning to buy the surface laptop 7 don't buy the wrong model we are neurotic laptop enthusiasts here so we bought all three of them with the ..."},
     {'video_id': '8PTexfv50e4',
      'title': 'Microsoft Surface Laptop 7 15 Inch Review',
      'video_link': 'https://www.youtube.com/watch?v=8PTexfv50e4',
      'channel_name': 'Alex Kidman',
      'cc_text': " [Music] Hey there Alex Kidman here with my quick video review of the Microsoft Surface laptop 7 so this is the co-pilot plus version of the surface laptop running on arm I've been ..."}]
