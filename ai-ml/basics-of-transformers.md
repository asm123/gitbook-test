# Basics of Transformers

References:

* [Transformers, explained: Understand the model behind GPT, BERT, and T5](https://www.youtube.com/watch?v=SZorAJ4I-sA)
* [Transformers, Explained: Understand the Model Behind GPT-3, BERT, and T5](https://daleonai.com/transformers-explained)

### History

Convolutional Neural Network (CNN) - for images

Recurrent Neural Network (RNN) - for translation

* Look at one word at a time sequentially
* Could not retain long context
* Susceptible to vanishing / exploding gradient problem
* Hard to parallelize

### Transformers

* Initially designed for translation
* Can be trained in parallel as opposed to sequential
* 3 main innovations / pillars:
  * Positional encoding
    * Store information about the order of word in the data itself instead of in the structure of the network
    * Learns the importance of the order of the words from the data itself
  * Attention mechanism
    * Neural network structure that allows a text model to look at every single word in the original sentence
    * Which words to attend to - learned from data
  * Self-attention
    * Building a model that understand the underlying meaning and patterns in a language for different language tasks
    * Allows a neural network to understand the context of the words around it
    * Helps in disambiguation, e.g., server in computing vs server in restaurant

### Understanding each pillar

#### Positional encoding

[Visual Guide to Transformer Neural Networks - (Episode 1) Position Embeddings](https://www.youtube.com/watch?v=dichIcUZfOw) - to understand the intuition\
[The Illustrated Transformer – Jay Alammar](https://jalammar.github.io/illustrated-transformer/)\
[Transformers Illustrated!. I was greatly inspired by Jay Alammar’s… | by Tamoghna Saha](https://tamoghnasaha-22.medium.com/transformers-illustrated-5c9205a6c70f)
