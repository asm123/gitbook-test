# AI Benchmarks

_These are notes from the podcast with some added external references. The podcast is wonderful, highly recommend listening to it in full for a high-level idea of benchmarking, interesting backstories and observations._

***

Hosts: swyx, Alessio Fanelli

Links:

* [Podcast on Latent Space](https://www.latent.space/p/benchmarks-101)

Synopsis: What does it really mean for an AI model to do well at X benchmark vs Y benchmark? How do you judge one model vs another?

***

#### Traditional metrics

Three main measures for benchmark metrics:

1. Accurancy = how many successful predictions the model does = `(TP + TN) / (TP + TN + FP + FN)`
2. Precision = how many of them are good compared to the overall amount of predictions made = `TP / (TP + FP)`
3. Recall = what proportion of positives were identified = `TP / (TP + FN)`

Translating to real-world application:

1. Precision: How many songs in a Spotify playlist did you like?
2. Recall: Of all the Spotify songs that like, how many were put into the playlist?

Most benchmarks use F1 score = harmonic mean of precision and recoall = `2 * precision * recall / (precision + recall)`

#### Recent metrics - especially for LLMs

[Stanford HELM](https://crfm.stanford.edu/2022/11/17/helm.html) - Rishi Bommasani, Percy Liang, Tony Lee

* 7 metrics: accuracy, calibration, robustness, fairness, efficiency, general information bias, toxicity

#### Benchmarking methodology

1. Zero-shot fashion: Do not provide any examples, just test how good the model is at generalizing.
2. Few shot: Provide `K` examples, see from there how good the model is.
3. Fine tune models: Take a bunch of data and fine-tune the model for a specific task and test it.

#### History of benchmarking

[WordNet](https://wordnet.princeton.edu/) - Princeton University - George Miller, Christiane Fellbaum

* 155k words organized in 175k synsets. Synsets are pairings of nouns, verbs adjectives, adverbs that go together.
* Useful for understanding semantic similarity, sentiment analysis, machine translation.
* Expanded to 4.5 million words of text.

[MNIST](https://en.wikipedia.org/wiki/MNIST_database)

* 60k training images of handwritten numbers.
* First visual dataset.
* Extended to EMNIST for handwritten letters and Fashion MNIST for fashion images.

[Enron email dataset](https://www.cs.cmu.edu/~enron/)

* 600k emails.
* Useful for email classification, email summarization, entity recognition, language modelling.

[ImageNet](https://www.image-net.org/) - Fei Fei Li

* Big for deep learning

[AlexNet](https://en.wikipedia.org/wiki/AlexNet)

* Error rate of 15%, first to use deep learning.

[CIFAR 10 and CIFAFR 100](https://www.cs.toronto.edu/~kriz/cifar.html)

[GLUE, SuperGLUE](https://gluebenchmark.com/)

* General Language Understanding Evaluation
* Has 9 tasks: single sentence tasks, similarity and paraphrase tasks, inference tasks.

[SQUAD](https://rajpurkar.github.io/SQuAD-explorer/) - Stanford Question Answering Dataset

* Dataset of `<question, answer>` pairs.
* One of the sentences of the paragraph drawn from Wikipedia contains the answer to the corresponding question.

[Swag, HellaSwag](https://rowanzellers.com/hellaswag/) - Situations with Adversarial Generations.

* 100k multiple choice questions.
* Common sense inference.
  * Contextual inference.
  * Conversational.
  * Plausibility - cause and effect kind of connections.
* Whether the language model can correctly pick the most likely answer based on the context.

[MMLU](https://paperswithcode.com/dataset/mmlu) - Massive Multitask Language Understanding

* 57 tasks - elementary math, US history, computer science, law, etc.
* Models require extensive world knowledge and problem solving to attain high accuracy on these tasks.

[HumanEval](https://github.com/openai/human-eval)

* Notable benchmark for code generation

[XTREME](https://sites.research.google/xtreme) - Cross-lingual Transfer Evaluation of Multilingual Encoders

* Multilingual benchmarks - extension of monolingual benchmarks.
* Accurately predicting the conversion of one word or one part of word to another part of the word.

[BIG-Bench](https://paperswithcode.com/dataset/big-bench)

* 204 tasks - linguistics, childhood development, math, common sense reasoning, biology, physics, social bias, software development, etc.

#### How to design benchmarks

Baseline = the score for randomly guessing all the questions.

* Then run each of the language models, human evaluations, median evaluations, expert evaluations.

#### Issues with benchmarks

* Data contamination and memorization
  * Language models are trained on data scraped from Internet. After a while, models may be memorizing the results of the tests instead of deriving them from first principles.
    * Overfitting.
  * Benchmarks might only include data until certain time cut-off, can be bad at generalizing.
* Benchmark data quality - inherent bias in the data, task specificity, reproducibility, resource requirements, calibrating confidence in the model's answers (hallucinating).
* Tradeoffs
  * Latency - how quickly can I get an answer
  * Inference cost - how much does each call to the model cost
  * Throughput - scaling the model to a lot of questions
