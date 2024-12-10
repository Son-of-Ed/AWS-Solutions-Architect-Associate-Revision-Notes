## Rekognition
Used to find object, people, text, and scenes in images and videos using ML.
Creates a database of "familiar faces" to use in future comparisons.
Use cases include content moderation, face search/verification, text detection, and pathing for sport analysis.
Can moderate content by setting a minimum confidence threshold that will filter out items that are considered too inappropriate. You can flag sensitive content for manual review using Amazon Augmented AI (A2I) which is useful for regulation compliance.

## Transcribe
Convert audio to text via a deep learning algorithm called automatic speech recognition.
Can automatically redact personally identifiable information (PII) and has support for multi-lingual audio.
Used for generating closed captions, transcribing calls, and generating metadata that can be used as a searchable archive.

## Polly
Generate audio from text.
Can use pronunciation lexicons to pronounce styled words e.g. AWS => Amazon Web Services, 41ex => Alex.
Can take text from plain text documents or from files marked up with Speech Synthesis Markup Language (SSML) to add emphasis to certain words, use phonetic pronunciation, add breathing/whispering, or using the Newscaster speaking style.

## Translate
Localise content for users by translating text.

## Lex
Technology that powers Amazon Alexa and is used for building chatbots/call centre bots.
Uses Automatic Speech Recognition (ASR) and natural language understanding to recognise intent of text/speech.

## Connect
Virtual call centre used to receive phone calls and route the content of the calls to different services e.g. call about an appointment, Connect puts you through to correct service e.g. Lex, Lex invokes Lambda, can schedule appointment with a Customer Relationship Manager (CRM).

## Comprehend
Natural Language Processing (NLP) service to gain insight and find relationships within text or unstructured data.
#### Comprehend Medical
Detects and returns useful information in unstructured clinical text.
Use NLP to detect protected health information.
Stores generated docs in S3.
Can be combined with Kinesis Data Firehose for real-time analytics or with Transcribe to take patient narratives and convert them into text that can be analysed by Comprehend Medical.

## SageMaker
Service for developers/data scientists to build machine learning models.
Historical data is labelled and supplied to SageMaker which builds an ML model based on that data. You then train and tune the model to make it more accurate so that it can produce reliable results when new data analysed by the model.

## Forecast
Managed service that uses ML to create forecasts.
50% more accurate than looking at the data alone.
Helps with product demand/financial/resource planning.

## Kendra
Document search service that takes in documents from lots of different sources so that you can extract answers using machine learning.
Learns from user interactions to promote preferred results.

## Personalize
Service that helps you build personalised apps with personalised customer recommendations e.g. you buy lots of X, you get recommended X in future.
Build recommendations based on historic data in S3 and real-time data.

## Textract
Extracts data, text, and handwriting from documents by using AI and ML.

#aws