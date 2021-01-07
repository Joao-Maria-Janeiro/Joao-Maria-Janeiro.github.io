---
layout: post
title: "Translation with Attention"
author: "João Maria Janeiro"
categories: posts
tags: [Machine Learning]
image: translate.jpg
---

# Goal
The goal of this project is to create a translator from english to portuguese.

## Tools
How will we go about doing this? There are several ways to tackle this issue but a really good one is to build a sequence to sequence (Seq2Seq) model with an attention mechanism! This will be done using Tensorflow.

### Introuction to RNN
If you already know about RNN feel free to jump to the **introduction to attention**. 

There are lots of data which are inherently sequential, like speech, sensor data, weather, finance, video and text, among others. Recurrent Neural Networks (RNN) are a family of neural networks designed specifically to deal with sequential data. In a traditional neural network we assume all inputs and outputs are independent of each other. This is not such a great idea for many tasks! If you want to, for instance, predict the next word in a sentece, you surely need to know the words that came before it. Like we base most of our understanding on context, this is the idea behind this architecture. Given a sentece like, NY is a city in ___, it's easy to know that the sentence would be filled with "the United States". 
RNN are called *recurrent* because they operate in a recurrent way, which means that they perform the same operations in every element of the sequence, with the output dependent on the previous elements. 

