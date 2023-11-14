# Hallucination Leaderboard

Public LLM leaderboard computed using Vectara's Hallucination Evaluation Model. This evaluates how often an LLM introduces hallucinations when summarizing a document. We plan to update this regularly as our model and the LLMs get updated over time.

Last updated on November 1st, 2023

|Model|Accuracy|Hallucination Rate|Answer Rate|Average Summary Length (Words)|
|----|----:|----:|----:|----:|
|GPT 4|97.0 %|3.0 %|100.0 %|81.1|
|GPT 3.5|96.5 %|3.5 %|99.6 %|84.1|
|Llama 2 70B|94.9 %|5.1 %|99.9 %|84.9|
|Llama 2 7B|94.4 %|5.6 %|99.6 %|119.9|
|Llama 2 13B|94.1 %|5.9 %|99.8 %|82.1|
|Cohere-Chat|92.5 %|7.5 %|98.0 %|74.4|
|Cohere |91.5 % |8.5 % |99.8 % |59.8 |
|Anthropic Claude 2 |91.5 % |8.5 % |99.3 % |87.5 |
|Mistral 7B |90.6 % |9.4 % |98.7 % |96.1 |
|Google Palm|87.9 % |12.1 % |92.4 % |36.2|
|Google Palm-Chat|72.8 % |27.2 % |88.8 % |221.1|

## Model
You can find the model used to compute this leaderboard open sourced for commercial use on hugging face: https://huggingface.co/vectara/hallucination_evaluation_model along with instructions how to use the model.

## Data
See [leaderboard-summaries.csv](https://github.com/vectara/hallucination-leaderboard/blob/main/leaderboard_summaries.csv) for the generated summaries we used to evaluate the models with.

## Methodology
To determine this leaderboard, we trained a model to detect hallucinations in LLM outputs, using various open source datasets from the factual consistency research into summarization models. Using a model that is competitive with the best state of the art models, we then fed 1000 short documents to each of the LLMs above via their public APIs and asked them to summarize each short document, using only the facts presented in the document. Of these 1000 documents, only 831 document were summarized by every model, the remaining documents were rejected by at least one model due to content restrictions. Using these 831 documents, we then computed the overall accuracy (no hallucinations) and hallucination rate (100 - accuracy) for each model. The rate at which each model refuses to respond to the prompt is detailed in the 'Answer Rate' column. None of the content sent to the models contained illicit or 'not safe for work' content but the present of trigger words was enough to trigger some of the content filters. The documents were taken primarily from the [CNN / Daily Mail Corpus](https://huggingface.co/datasets/cnn_dailymail/viewer/1.0.0/test). 

We evaluate summarization accuracy instead of overall factual accuracy because it allows us to compare the model's response to the provided information. In other words, is the summary provided 'factualy consistent' with the source document. Determining halucinations is impossible to do for any ad hoc question as it's not known precisely what data every LLM is trained on. In addition, having a model that can determine whether any response was hallucinated without a reference source requires solving the hallucination problem and presumably training a model as large or larger than these LLMs being evaluated. So we instead chose to look at the hallucination rate within the summarization task as this is a good analogue to determine how truthful the models are overall. In addition, LLMs are increasingly used in RAG (Retrieval Augmented Generation) pipelines to answer user queries, such as in Bing Chat and Google's chat integration. In a RAG system, the model is being deployed as a summarizer of the search results, so this leaderboard is also a good indicator for the accuracy of the models when used in RAG systems.

## Prompt Used
> You are a chat bot answering questions using data. You must stick to the answers provided solely by the text in the passage provided. You are asked the question 'Provide a concise summary of the following passage, covering the core pieces of information described.'  &lt;PASSAGE&gt;'

When calling the API, the &lt;PASSAGE&gt; token was then replaced with the source document (see the 'source' column in [leaderboard-summaries.csv](https://github.com/vectara/hallucination-leaderboard/blob/main/leaderboard_summaries.csv) ). 

## API Details
For GPT 3.5 we used the model name ```gpt-3.5-turbo``` in their API, and ```gpt-4``` for GPT4, and we used the ```ChatCompletion``` endpoint from the python client library. For the 3 Llama models, we used the Anyscale hosted endpoints for each model. For the Cohere models, we used their ```/generate``` endpoint for *Cohere*, and ```/chat``` for *Cohere-Chat*. For Anthropic, we used the largest ```claude 2``` model they offer through their API. For the Miustral 7B model, we used the  [Mistral-7B-Instruct-v0.1](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.1) model, hosted via Hugging Face's API. For Google Palm we used the ```text-bison-001``` model, and for Google Palm Chat we used ```chat-bison-001```.

## Frequently Asked Questions
* **Qu.** Why are you are using a model to evaluate a model?
* **Answer** There are several reasons we chose to do this over a human evaluation. While we could have crowdsourced a large human scale evaluation, that's a one time thing, it does not scale in a way that allows us to constantly update the leaderboard as new APIs come online or models get updated. We work in a fast moving field so any such process would be out of data as soon as it published. Secondly, we wanted a repeatable process that we can share with others so they can use it themselves, something not possible for a human annotation process, where the only things that could be shared are the process and the labels acquired from that annotation run. By sharing it with others they can use it to evalaute and improve their own LLMs, where for commercial use, research or for self education. It's also worth pointing out that building a model for detecting hallucinations is much easier than building a model that is free of hallucinations. So long as the halucination evaluation model is highly correlated with human raters' judgements, it can stand in as a good proxy for human judgements. As we are targetting summarization and not general 'closed book' question answering, the LLM we trained does not need to have memorized a large proportion of human knowledge, it just needs to have a solid grasp and understanding of the languages it support (currently just english, but we plan to expand language coverage over time).

* **Qu.** What version of model XYZ did you use?
* **Answer** Please see the API details section for specifics about the model versions used and how they were called, as well as the date the leaderboard was last updated. Please contact us (create an issue in the repo) if you need more clarity.

## Coming Soon
* GPT4 Turbo results
* We will also be adding a leaderboard on citation accuracy. As a builder of RAG systems, we have noticed that LLMs tend to mis-attribute sources sometimes when answering a question based on supplied search results. We'd like to be able to measure this so we can help mitigate it within our platform.
* We also plan to cover more languages than just english. Our current platform covers over 100 languages, and we want to develop hallucination detectors with comparable multi-lingual coverage.
