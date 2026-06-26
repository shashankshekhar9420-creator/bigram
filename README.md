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
