# Bigram
This project is an implementation of a character-level bigram language model from scratch. The model learns patterns in a text (from the given data set) by analyzing which characters are likely to appear next and uses those learned probabilities to generate new text. Basically- given a character, the model predicts the next possible character and uses these predictions to generate new words.
<br>

First let us clear out the concept and avoid any misconception:
- The model is generative. It does not memorize the words from the data set. What is does is it generates the next token or character based on what it has learned in the data set (the relationships and its probabilites).
- So, this distinction is very important that unlike a traditional program, which "retrieves" data from existing data set, our model is "generative".
Let us understand this through an illustration:<br>

Imagine our data set is:
```
cat
car
can
cap
dog
dot
door
```
Say we give the following characters: `do` and the model has to continue.
It can continue as `dog` or `dot` or `door`. But it might also generate `dor`, although `dor` is not a word in our data set.
<br>

Why would that happen? Because the model does not know that `door` is a thing and `dor` is wrong. It only knows that `r` sometimes follows `o` and other such relationships. So the generation is purely probabilistic and not retrieval or meaningful. The model does not memorize the words in the data set, it learns the patterns and the output is the combination of patterns it learns.

# The overall goal
The goal of this project is to build a program that can generate new names that look similar to the data set of the names the model is trained on.
<br>

The data set contains names. Those are the only things that the model sees, we never define spelling rules, grammer rules or instructions about what would make a name sound human. We only provide examples. So what we are trying to find out is: <i> Can a computer learn patterns from examples alone and use those patterns to create something new?</i>
<br>

At the beginning, we have nothing more than a text file:
```
emma
oliver
liam
sophia
..
```
A computer does not see names. It sees just a sequence of characters- it does not know that "emma" is a name. It only learns the patterns- after e, m often appears; after m, m or a often appears.
This project is about converting this raw sequence of characters into something mathematical that out machine can learn from.
<br>

## The core idea: prediction

The fundamental question is: given what I have seen so far, what is likely to come next?
For example: Given `em` what comes next?
<br>

Possible predictions:
- m → 70%
- i → 10%
- a → 5%
- ...

The model is not memorizing complete names. It is learning probabilities of what comes next.
<br>

We start with the question: What character comes next?
<br>

Example- Training data: `anna` becomes
```
a → n
n → n
n → a
a → end
```

Each pair teaches the model one small piece of the pattern. After seeing thousands of examples, these tiny patterns combine into something that can generate new names.
<br>

## How do we get there?
The project is divided into three main ideas:
<br>

<b>1. Represent language mathematically</b>
<br>

We need a way to represent: `a is followed by b 20% of the time, a is followed by c 10% of the time` as numbers.
<br>

<b>2. Use those probablities to generate names</b>
<br>

Once we begin, we can sample the probablities one after the other.
<br>

Example:
`Start(.) -> model predicts: a -> predicts: n -> predicts: d.....`
<br>

Eventually: `andrew`
<br>

The model creates a new sequence one character at a time.
<br>

<b>3. Measure how good the model is </b>
<br>

We need a way to measure: How wrong are the predictions?
<br>

Example: Model predicts the character after 'q':
```
After 'q':

u: 90%
a: 5%
x: 5%
```

The real answer should be: `u`  That would be considered as a good prediction.
<br>

But if the model predicts:`x` That would be a bad prediction.
<br>

We need a mathematical way to quantify this error. That would become the loss function.

# Step 1: Turning words into countable events

We want the model to stop thinking in terms of complete words, eg:`emma`
and start thinking in terms of tiny relationships:
```
e is followed by m
m is followed by m
m is followed by a
```

Why do we need to do this? Because the model does not have to memorize names, it has to learn what usually comes next.
<br>

So, the smallest useful unit is: (current character, next character). This pair is called a <b>bigram.</b>

## Boundary markers
We introduce a special character: `.` to denote the beginning and the end of the word.
<br>

Hence `emma` becomes `['.', 'e', 'm', 'm', 'a', '.']`. Now the word has boundaries and hence the model can learn that `e` (.,e) appears at the beginning and `a` (a,.) appears at the end of the word.

## Creating the events
Now we slide across the word two characters at a time. Look at neighboring pairs, these are the events.
<br>

The word `emma` becomes:
```
('.','e')
('e','m')
('m','m')
('m','a')
('a','.')
```
These becomes 5 learning examples for the model. This is done because every character needs to predict what comes after it.
<br>

Hence we have transformed a data set of words into a dataset of events (what character comes after what). Now we have something we can count. Once we have thousands of these events, we can ask:
"How many times does each character follow another character?"
