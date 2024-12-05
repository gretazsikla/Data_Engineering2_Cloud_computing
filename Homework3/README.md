# Cloud Computing Analysis - Solving Language Barriers with AWS

In this blog post, I would like to explore a real-world topic. In my experience, many Hungarians don't speak English, so when they try to stay informed about international news, they have to rely on Hungarian journalists' translation skills and objectivity. Additionally, there are English articles that include videos, which are often not transcribed. I think this is a significant problem, as the lack of language proficiency makes people more vulnerable to influence. With the help of Amazon services, I would like to demonstrate how this problem could be solved.

## Video Analysis: Global News - "Trump Suggests Canada Could Become 51st State"

For the demonstration of the tools, I used a video (https://www.youtube.com/watch?v=UUJqLwxp-vE) published on the 3rd of December by Global News. The backstory of this video is that recently Donald Trump was elected president of the United States, and during a dinner at Mar-a-Lago over the weekend, he suggested that Canada could become the United States' 51st state and that Prime Minister Justin Trudeau would become its governor. In the video, journalists ask Canadian politicians for their opinions about this statement.

### Steps of the Analysis

#### 1. Transcription of the Video
First, I converted the YouTube video to mp3 format and uploaded it to my GitHub workspace, then to my Amazon bucket. For the conversion, I used this website: https://y2mate.nu/en-lZM2/. After that, I imported all the necessary libraries and set the AWS region for the services. You can see the code for transcribing below. I also saved the text into a file called `speech_text.txt`.

**Fun fact:** I thought it would be interesting to do the same project with songs. I tried to transcribe *Rolling in the Deep* by Adele, but unfortunately, the program couldn’t hear out the words, probably due to the music. It only transcribed 2 sentences, which weren’t even good.

#### 2. Sentiment Analysis on the Text
It wasn’t strictly necessary for the goal of the analysis, but I thought it would be interesting to check what AI thinks about the mood of the text. I originally wanted to do it on the translated text, but sentiment analysis is not yet available in Hungarian. You can see the code below as well as the output of the sentiment analysis. In my opinion, the analysis was mostly good. I consider the interviews to be positive as well, with a slight tone of defensiveness.

#### 3. Translating the Text
You can see the translated text below. I do speak both English and Hungarian, so I checked the correctness of the translation. As the people in the video are not giving a prepared speech, the translation is a little fragmented. Sometimes it combines into one sentence what the interviewers say and what the politicians respond to. As there are many interviewers, the questions are not understandable quite often. It also makes it a little bit hard to read the text because sometimes the interviewers don’t let the politicians complete their sentences. However, even apart from these mistakes, which are due to the quality of the video, there are some clear translation mistakes. For example, in the video, they say that “Trump was teasing us,” and it was translated as “scared us.” Another mistranslation was “No one had pillows.” He either meant that “no one had pens for taking notes” or he meant “no one had pads” (like iPads). To be fair, I am not sure that someone only reading the text below would fully understand the context.

#### 4. Celebrity Recognition Through Pictures
At first, I didn’t see that there is an option for video celebrity recognition, so I took pictures of the people in the video and made celebrity recognition based on the pictures. Truth be told, in the video, there is some text explaining who the politicians are, but I think there could be cases when it is not accurate. Out of 4 people, it recognized 3 people well, which is 75% accuracy. The program didn’t recognize Arif Rivani, the Minister of Justice and Attorney General of Canada. The program said it is Carlos Mena, a Spanish singer, with 76% confidence. The program recognized everyone else with 99-100% confidence.

#### 5. Celebrity Recognition Through Video
I had the preconception that through the video, it would give a better result for Arif Rivani. It gave a different answer, but still not the correct answer. The program thought it was Xavier García Albiol. At different moments of the video, the program had different confidence levels, but it was between 77-90%. The program was right about the other politicians.

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
