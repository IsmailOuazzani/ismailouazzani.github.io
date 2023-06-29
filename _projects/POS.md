---
layout: page
title: Part-of-Speech Tagger
description: Natural Language Processing, Part-of-Speech Tagging, Hidden Markov Models
img: assets/img/POS-Tagging-800x400.jpg
importance: 5
category: Projects
---

## Technologies Used
- Python
- Viterbi Algorithm
- Bayesian Inference
- Hidden Markov Models

<!-- Github Repository: <a href="https://github.com/IsmailOuazzani/AtomicAI">Atomic AI</a> -->

# Introduction

The focus of this project is to predict the categories of words in a sentence: the <a href="https://en.wikipedia.org/wiki/Part_of_speech">Part of Speech (POS)</a>. To do so, we use a hidden markov model, which contains the probabilities that each word belongs to each of the categories. I then use the Viterbi algorithm to classify the words, as it allows me to compute the sequence of POS tags that has the highest joint probability efficiently - that is the most likely.

## Personal Contributions
- Implementation of a Data Preprocessing script to extract the words and their POS tags 
- Training of the HMM model using dataset
- Implementation of the Viterbi algorithm to classify the words
- Implemented all functions with Numpy arrays to improve performance
- Implementation of a script to evaluate the accuracy of the model

