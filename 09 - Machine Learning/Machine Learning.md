---
tags:
  - aws/ml
  - machine-learning
category: Machine Learning
---

# 🤖 AWS Machine Learning (Overview)

> [!INFO] Overview
> AWS provides a vast array of Machine Learning (ML) services. For the **AWS Certified Solutions Architect Associate (SAA-C03)** exam, you generally do not need deep, hands-on ML expertise or data science knowledge. Instead, you only need to know **which service to use for which use case**.

This is a high-level cheat sheet of the ML services that frequently appear as distractors or specific answers in the exam.

---

## 🧠 Core ML Services

### Amazon Comprehend
* **What it does**: Natural Language Processing (NLP) service that finds insights and relationships in text.
* **Key Use Case**: Sentiment analysis (positive, negative, neutral), keyword extraction, and topic modeling from customer feedback, product reviews, or social media.

### Amazon Kendra
* **What it does**: Highly accurate, ML-powered enterprise search service.
* **Key Use Case**: Allowing users to search across various disparate document repositories (S3, SharePoint, internal wikis) using natural language questions (e.g., "Where is the IT help desk?").

### Amazon Lex
* **What it does**: Conversational AI for building voice and text chatbots. (This is the same engine that powers Amazon Alexa).
* **Key Use Case**: Creating chat interfaces for customer service routing, ordering a pizza via text, or booking appointments.

### Amazon Polly
* **What it does**: Text-to-Speech (TTS) service. Turns text into lifelike speech.
* **Key Use Case**: Creating audio versions of blog posts, news articles, or generating speech for automated contact centre responses.

### Amazon Rekognition
* **What it does**: Image and video analysis service.
* **Key Use Case**: Facial recognition, identifying objects/people/text/scenes in images, detecting inappropriate/explicit content in user uploads, and tracking people across video frames.

### Amazon Textract
* **What it does**: Extracts text, handwriting, and data from scanned documents.
* **Key Use Case**: Automating data entry from highly structured physical documents like tax forms, receipts, invoices, or medical records. (Much more advanced than basic OCR).

### Amazon Transcribe
* **What it does**: Speech-to-Text service. Automatically converts speech into text.
* **Key Use Case**: Generating subtitles for videos, transcribing customer service calls, or capturing meeting notes for later analysis.

### Amazon Translate
* **What it does**: Neural machine translation service.
* **Key Use Case**: Translating text from one language to another quickly and accurately, such as translating a website, app content, or customer support tickets into an agent's native language.

### Amazon Forecast
* **What it does**: Time-series forecasting service based on ML.
* **Key Use Case**: Predicting future business outcomes like inventory demand, financial planning, or computing resource requirements using historical data.

### Amazon Fraud Detector
* **What it does**: Fully managed service that identifies potentially fraudulent online activities.
* **Key Use Case**: Catching fake account creations, detecting fraudulent online payments, or preventing abuse of free-tier services.

### Amazon SageMaker
* **What it does**: A fully managed service for developers and data scientists to build, train, deploy, and manage custom ML models at scale.
* **Key Use Case**: Used when you need to write custom algorithms, train your *own* models, or manage the ML lifecycle yourself (rather than using the pre-trained, managed AI services listed above).

---

## 🎯 Exam PowerUP: Quick Associations

> [!IMPORTANT]
> When you see these keywords in a question, immediately think of the corresponding service:
> * **Search internal documents / Enterprise Search** → **Kendra**
> * **Sentiment analysis / NLP** → **Comprehend**
> * **Chatbots / Voice bots** → **Lex**
> * **Text-to-Speech** → **Polly**
> * **Speech-to-Text** → **Transcribe**
> * **Image/Video analysis or Facial Recognition** → **Rekognition**
> * **Extract data from forms/invoices/PDFs** → **Textract**
> * **Translation** → **Translate**
> * **Time-series predictions / Demand prediction** → **Forecast**
> * **Build/Train/Deploy custom ML models** → **SageMaker**
