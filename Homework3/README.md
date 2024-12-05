# Cloud Computing Analysis - Solving Language Barriers with AWS

In this blog post, I would like to explore a real-world topic. In my experience, many Hungarians don't speak English, so when they try to stay informed about international news, they have to rely on Hungarian journalists' translation skills and objectivity. Additionally, there are English articles that include videos, which are often not transcribed. I think this is a significant problem, as the lack of language proficiency makes people more vulnerable to influence. With the help of Amazon services, I would like to demonstrate how this problem could be solved.

## Video Analysis: Global News - "Trump Suggests Canada Could Become 51st State"

For the demonstration of the tools, I used a [video](https://www.youtube.com/watch?v=UUJqLwxp-vE) published on the 3rd of December by Global News. The backstory of this video is that recently Donald Trump was elected president of the United States, and during a dinner at Mar-a-Lago over the weekend, he suggested that Canada could become the United States' 51st state and that Prime Minister Justin Trudeau would become its governor. In the video, journalists ask Canadian politicians for their opinions about this statement.

### Steps of the Analysis

#### 1. Transcription of the Video
First, I converted the YouTube video to mp3 format and uploaded it to my GitHub workspace, then to my Amazon bucket. For the conversion, I used this [website](https://y2mate.nu/en-lZM2/). After that, I imported all the necessary libraries and set the AWS region for the services. You can see the code for transcribing below. I also saved the text into a file called `speech_text.txt`.

**Fun fact:** I thought it would be interesting to do the same project with songs. I tried to transcribe *Rolling in the Deep* by Adele, but unfortunately, the program couldn’t hear out the words, probably due to the music. It only transcribed 2 sentences, which weren’t even good.


```python
# Import required libraries and set up our environment
import os
import pprint
import random
import string
import boto3
import time

s3 = boto3.client(
    "s3",
    region_name="eu-west-1",  # Ireland region
)

# Transcribing the text
import requests
transcribe = boto3.client('transcribe')
# Define the transcription job details
job_name = 'speech_job_2'  
s3_bucket = 'name'  
s3_key = 'speech.mp3'  
media_uri = f's3://{s3_bucket}/{s3_key}'
output_file = "speech_text.txt"

try:
    response = transcribe.start_transcription_job(
        TranscriptionJobName=job_name,
        Media={'MediaFileUri': media_uri},
        MediaFormat='mp3',
        LanguageCode='en-US'  # Specify language code
    )
    print("Transcription job started:", response['TranscriptionJob']['TranscriptionJobName'])

    # Wait for the job to complete
    while True:
        status = transcribe.get_transcription_job(TranscriptionJobName=job_name)
        if status['TranscriptionJob']['TranscriptionJobStatus'] in ['COMPLETED', 'FAILED']:
            break
        print("Waiting for transcription to complete...")
        time.sleep(5)

    if status['TranscriptionJob']['TranscriptionJobStatus'] == 'COMPLETED':
        transcript_uri = status['TranscriptionJob']['Transcript']['TranscriptFileUri']
        print("Transcription completed! Fetching the transcript...")

        # Fetch the transcript JSON file
        response = requests.get(transcript_uri)
        transcription_data = response.json()

        # Append the transcription to the file
        with open(output_file, "a") as f:
            for segment in transcription_data["results"]["audio_segments"]:
                segment_text = segment["transcript"]
                f.write(segment_text + "\n")  # Append each transcript segment
                print(f"Appended: {segment_text}")

        print(f"Full transcription saved to {output_file}")
    else:
        print("Transcription failed:", status)

except Exception as e:
    print("Error occurred:", e)
```

#### 2. Sentiment Analysis on the Text
It wasn’t strictly necessary for the goal of the analysis, but I thought it would be interesting to check what AI thinks about the mood of the text. I originally wanted to do it on the translated text, but sentiment analysis is not yet available in Hungarian. You can see the code below as well as the output of the sentiment analysis. In my opinion, the analysis was mostly good. I consider the interviews to be positive as well, with a slight tone of defensiveness.

💭 Sentiment analysis:
Overall sentiment: POSITIVE

Sentiment scores:
- Positive: 38.66%
- Negative: 24.84%
- Neutral: 10.31%
- Mixed: 26.19%

![Pie Chart](https://github.com/gretazsikla/Data_Engineering2_Cloud_computing/raw/main/Homework3/pie.PNG)


```python
# making sentiment analysis on the original text
S3_BUCKET = 'name' 
S3_KEY = "speech_text.txt"

comprehend = boto3.client(service_name="comprehend", region_name="eu-west-1")
try:
    s3_response = s3.get_object(Bucket=S3_BUCKET, Key=S3_KEY)
    text_to_analyse = s3_response['Body'].read().decode('utf-8')
    response = comprehend.detect_sentiment(Text=text_to_analyse, LanguageCode="en")

    print("\n📝 Input text:")
    print("-" * 40)
    print("-" * 40)

    print("\n💭 Sentiment analysis:")
    print(f"Overall sentiment: {response['Sentiment']}")
    print("\nSentiment scores:")
    for sentiment, score in response["SentimentScore"].items():
        print(f"- {sentiment}: {score:.2%}")

except Exception as e:
    print(f"❌ Error: {str(e)}")
```

#### 3. Translating the Text
You can see the translated text through the [link](https://raw.githubusercontent.com/gretazsikla/Data_Engineering2_Cloud_computing/refs/heads/main/Homework3/translated_text.txt)
. I do speak both English and Hungarian, so I checked the correctness of the translation. As the people in the video are not giving a prepared speech, the translation is a little fragmented. Sometimes it combines into one sentence what the interviewers say and what the politicians respond to. As there are many interviewers, the questions are not understandable quite often. It also makes it a little bit hard to read the text because sometimes the interviewers don’t let the politicians complete their sentences. However, even apart from these mistakes, which are due to the quality of the video, there are some clear translation mistakes. For example, in the video, they say that “Trump was teasing us,” and it was translated as “scared us.” Another mistranslation was “No one had pillows.” He either meant that “no one had pens for taking notes” or he meant “no one had pads” (like iPads). To be fair, I am not sure that someone only reading the text below would fully understand the context.

```python
translate = boto3.client("translate")
S3_BUCKET = 'name' 
S3_KEY = "speech_text.txt"

print("\n📂 Text location:")
print(f"- Bucket: {S3_BUCKET}")
print(f"- Key: {S3_KEY}")

try:
    s3_response = s3.get_object(Bucket=S3_BUCKET, Key=S3_KEY)
    text_to_translate = s3_response['Body'].read().decode('utf-8')
    translate_response = translate.translate_text(
        Text=text_to_translate, SourceLanguageCode="en",
        TargetLanguageCode="hu"
    )
    translated_text = translate_response['TranslatedText']

    # Save the translated text to a file
    with open("translated_text.txt", "w", encoding="utf-8") as file:
        file.write(translated_text)

    print("\n✅ Translate analysis complete!")

    print("\n📦 Raw response from AWS:")
    pp.pprint(translate_response)

except Exception as e:
    print(f"❌ Error translating: {str(e)}")
```

#### 4. Celebrity Recognition Through Pictures
At first, I didn’t see that there is an option for video celebrity recognition, so I took pictures of the people in the video and made celebrity recognition based on the pictures. Truth be told, in the video, there is some text explaining who the politicians are, but I think there could be cases when it is not accurate. Out of 4 people, it recognized 3 people well, which is 75% accuracy. The program didn’t recognize Arif Rivani, the Minister of Justice and Attorney General of Canada. The program said it is Carlos Mena, a Spanish singer, with 76% confidence. The program recognized everyone else with 99-100% confidence. The program also gave information about their facial experssion. You can see the main results below.

```python
# Analyze image for celebrity detection
print("🔍 Analyzing image for celebrities...")
import boto3
rekognition = boto3.client("rekognition")

S3_BUCKET = "name"
S3_KEY = "pol1.PNG"

print("\n📂 Image location:")
print(f"- Bucket: {S3_BUCKET}")
print(f"- Key: {S3_KEY}")

try:
    celebrity_response = rekognition.recognize_celebrities(
        Image={
            "S3Object": {
                "Bucket": S3_BUCKET,
                "Name": S3_KEY,
            }
        }
    )

    print("\n✅ Image analysis complete!")
    print(f"Found {len(celebrity_response['CelebrityFaces'])} celebrities")

    print("\n📦 Raw response from AWS:")
    pp.pprint(celebrity_response)
    celebrity_name = celebrity_response['CelebrityFaces'][0]['Celebrity']['Name']
    print(f"Recognized celebrity: {celebrity_name}")

except Exception as e:
    print(f"❌ Error analyzing image: {str(e)}")

try:
    if not celebrity_response["CelebrityFaces"]:
        print("No celebrities detected in the image.")
    else:
        for i, celebrity in enumerate(celebrity_response["CelebrityFaces"], 1):
            print(f"\n🌟 Celebrity {i}:")
            print(f"Name: {celebrity['Name']}")
            print(f"Confidence: {celebrity['MatchConfidence']:.2f}%")
            print("\nReference URLs:")
            for url in celebrity["Urls"]:
                print(f"- https://{url}")

            # Print face details if available
            if "Face" in celebrity:
                print("\nFace Details:")
                face = celebrity["Face"]
                emotions = face.get("Emotions", [])
                if emotions:
                    top_emotion = emotions[0]  # Emotions are returned sorted by confidence
                    print(f"- Top emotion: {top_emotion['Type']} ({top_emotion['Confidence']:.1f}%)")

            print("-" * 40)

except Exception as e:
    print(f"❌ Error processing results: {str(e)}")
```

| Name                     | Confidence   | Top Emotion | Top Emotion confidence % |
|--------------------------|--------------|-------------|---------------|
| Dominic LeBlanc           | 99.99%       | CALM        | 58.9%         |
| Carlos Mena               | 76.62%       | CALM        | 56.3%         |
| François-Philippe Champagne | 100.00%      | SURPRISED   | 94.1%         |
| Jean-Yves Duclos           | 100.00%      | CALM        | 94.5%         |



#### 5. Celebrity Recognition Through Video
I had the preconception that through the video, it would give a better result for Arif Rivani. It gave a different answer, but still not the correct answer. The program thought it was Xavier García Albiol. At different moments of the video, the program had different confidence levels, but it was between 77-90%. The program was right about the other politicians.

```python
try:
    # Start celebrity recognition job
    response = rekognition.start_celebrity_recognition(
        Video={
            "S3Object": {
                "Bucket": S3_BUCKET,
                "Name": S3_KEY,
            }
        }
    )

    job_id = response['JobId']
    print(f"\n🚀 Job started! Job ID: {job_id}")

    # Wait for job completion
    print("⏳ Waiting for the celebrity recognition job to complete...")
    while True:
        status_response = rekognition.get_celebrity_recognition(JobId=job_id)
        status = status_response['JobStatus']
        if status in ['SUCCEEDED', 'FAILED']:
            break
        time.sleep(10)

    if status == 'SUCCEEDED':
        print("\n✅ Video analysis complete!")

        print("\n📦 Raw response from AWS:")
        pp.pprint(status_response)

        print("\n📋 Recognized celebrities:")
        for celeb in status_response['Celebrities']:
            name = celeb['Celebrity']['Name']
            timestamp = celeb['Timestamp']
            print(f"- {name} at {timestamp}ms")
    else:
        print("\n❌ Video analysis failed.")

except Exception as e:
    print(f"❌ Error analyzing video: {str(e)}")

try:
    if not status_response["Celebrities"]:
        print("No celebrities detected in the video.")
    else:
        print("\n🌟 Detected Celebrities:")
        for i, celebrity_info in enumerate(status_response["Celebrities"], 1):
            celebrity = celebrity_info["Celebrity"]
            timestamp = celebrity_info["Timestamp"]

            print(f"\n🌟 Celebrity {i}:")
            print(f"Name: {celebrity['Name']}")
            print(f"Confidence: {celebrity['Confidence']:.2f}%")
            print(f"Detected at: {timestamp} ms")

            # Print reference URLs
            print("\nReference URLs:")
            for url in celebrity.get("Urls", []):
                print(f"- {url}")

            # Print additional face details if available
            if "Face" in celebrity:
                face = celebrity["Face"]
                emotions = face.get("Emotions", [])
                if emotions:
                    top_emotion = emotions[0]  # Emotions are returned sorted by confidence
                    print(f"\nFace Details:")
                    print(f"- Top emotion: {top_emotion['Type']} ({top_emotion['Confidence']:.1f}%)")

            print("-" * 40)

except Exception as e:
    print(f"❌ Error processing results: {str(e)}")
```

### Cost Analysis

#### Overall Cost, Including Trials
- Initially, I tried to transcribe the Adele song. I attempted this 3 times using different codes, as well as using the website. The song is 4 minutes long.
    - **Cost:** 3 * 4 minutes * $0.024 = $0.288
- At one point, I mistakenly selected "Transcribe Medical" for the Adele song, which is more expensive.
    - **Cost:** 4 minutes * $0.075 = $0.3
- The interview was 5 minutes long, and I transcribed it 3 times due to error messages.
    - **Cost:** 3 * 5 minutes * $0.024 = $0.36
- The transcribed text contains 4,813 characters (including spaces), which is equivalent to 481.3 units for sentiment analysis.
    - **Cost:** 481.3 units * $0.0001 = $0.04813
- I wanted to translate 4,813 characters. Since the cost is $15 per million characters:
    - **Cost:** 0.004813 * $15 = $0.0722
- For celebrity recognition, I processed 4 pictures:
    - **Cost:** 4 * $0.001 = $0.004
- For celebrity recognition with video (5 minutes long):
    - **Cost:** 5 minutes * $0.1 = $0.5

**Overall Total:** $1.57233

#### Cost of Actual Work
- Transcription: 5 * $0.024 = $0.12
- Sentiment Analysis: $0.04813
- Translation: $0.0722
- Celebrity Recognition with Pictures: $0.004
- Celebrity Recognition with Video: $0.5
