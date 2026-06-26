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

Hence we have transformed a data set of words into a dataset of events (what character comes after what). Now we have something we can count.
<br>

But this is only one name. Our model needs to learn from the entire dataset. So now we need to find: Across all names, how often does each character follow another character? That is what the matrix N stores.

# Step 2: Counting the bigrams (the matrix N)
Suppose our data set is:
```
emma
anna
amy
```

After step 1, these names are converted into:<br>

emma: `(., e) (e, m) (m, m) (m, a) (a, .)` <br>

anna: `(., a) (a, n) (n, n) (n, a) (a, .)`<br>

amy: `(., a) (a, m) (m, y) (y, .)` <br>

Now we need to build the frequency table- How many times did the following occur:
```
m → a =1
a → . =2
```

## The idea of matrix N
We create a grid: Rows = current character and Columns = next character. For example-
```
        .   a   b   c   d   ...
     -------------------------
.   |   0   2   0   0   0
a   |   2   0   1   0   0
b   |   0   0   0   4   0
c   |   0   0   0   0   2
...
```

- This is our matrix: N
- The meaning of a single cell: N[i,j] means how many times character i was followed by character j?
- Example: N[m,a] means how many times did m appear before a?
- Why 27 × 27? Our data set has only lower case alphabets `(a-z)` which means 26 characters and one for the special boundary token `.`.
- Since every character can have any other character as the next character, each character has 27 ways. So, total possible transitions are 27 x 27.

## But computers don't understand letters
The matrix cannot have:
```
        a b c
a       0 3 1
b       2 0 4
```

Computers need numbers, so we create mappings:
```
stoi = {
'a':1,
'b':2,
'c':3,
...
'.':0
}
```

- stoi means string to integer
- example: `m` becomes 13
- hence: N[13,1] means count of `m` followed by `a`
## How counting happens:
```
for w in words:
  chs = ['.'] + list(w) + ['.']
  for ch1, ch2 in zip(chs, chs[1:]):
    ix1 = stoi[ch1]
    ix2 = stoi[ch2]
    N[ix1, ix2] += 1
```

The following steps will happen for every word, for example: emma:
- emma is converted into .emma.
- then pairs are taken: `(., e) (e, m) (m, m) (m, a) (a, .)`
- then characters are convered into numbers, for example: .->0, e->5
- then count is increased, for example: N[0,5] += 1 (meaning: Increase the count of . → e by one)

The numbers in N are raw counts and not probabilites. Next we turn counts into probabilities.
So up till here we have answered: "How many times did each transition happen?" <br>

Now we answer: "Given a character, what are the chances of each possible next character?"

# Step 3: Turning counts into probabilities (the matrix P)
Now, we want to convert:
```
m → a happened 50 times
m → e happened 20 times
m → i happened 10 times
```
into:
```
If I see m:

a → 62.5%
e → 25%
i → 12.5%
```

This would create a probability distribution from which we can sample from.
We have already built: `N` the count matrix.
Example:
```
        a   e   i   .
m       50  20  10  0
```

Meaning: After m-
- a appeared 50 times
- e appeared 20 times
- i appeared 10 times

Total: 50+20+10=80 <br>

But these numbers are not probabilities, they're just frequencies. Probablities must add up to 1. Hence we normalize.

## Normalization
P (next∣current)= count(current,next)​ / total counts after current
<br>

For example: for `m`-
- P(m→a)=50/80= 0.625
- P(m→e)=20/80= 0.25
- P(m→i)=10/80= 0.125

Now these terms add up to 1: 0.625+0.25+0.125=1

## Matrix P
Now we create a matrix P, which can be considered as the probability version of matrix N.
For example:
<br>

N:
```
        a   e   i

m       50  20  10
```

becomes- P:
```
        a      e      i

m       .625   .25    .125
```

Now every row is a probability distribution. <br>

<b>Why rows?</b> As we discussed, rows represent the current character and the columns represent the next character.
<br>

So: P[m] means: show the probability of coming after m for all charachters.

## Code explanation
```
P = (N+1).float()
```
- creates a copy of N and converts counts into floating point numbers.
- Why +1? This is called smoothing. +1 adds a count of exactly 1 to every single element in the tensor. This eliminates any zeros in our data (Laplace smoothing). This means that every possible transition gets a tiny probability.

Example:
```
before-
a: 0
b: 0
c: 5

after-
a: 1
b: 1
c: 6
```

Smoothing makes a model less confident (which means the model's predictions are less extreme and closer to a 50/50 guess)
```
P /= P.sum(1, keepdims=True)
```

This means: divide every row by its row total.
Example:
```
Before:
[50,20,10]
sum: 80

After:
[50/80,20/80,10/80]
=> [0.625,0.25,0.125]
```

Hence, now we can generate. Earlier we only knew- m happened 50 times before a, etc etc.<br>

Now we know- after m:
- a has probability 62%
- e has probability 25%
- i has probability 13%

## Important Note: No need for training loop and how it is different from neural network.
We are not guessing any parameters or optimizing anything. The data set already tells us the answer.
So, the whole process of guessing weights, calculating error, backpropogation and optimizing weights is not occuring here.
<br>

The best probability estimate is simply: count/total
<br>
​
If: In 1000 names after q, u appears 900 times, then the model should learn:
```
P(u∣q)=0.9
```

There is nothing to optimize.
The calculation itself gives the optimal table.

# Step 4: Using P to generate
First let us understand what do we want? We want our model to create a new name. Not copy (retrieve) a name from the dataser. The goal is not to pick the name `emma` from the list, it is to generate a new sequence that follows same patterns which the model learnt in the data set.
<br>

We now have a probability matrix P. A row tells that if it is currently at this character what is the probability of all the other characters to come next.<br>

For example: lets take `m`, what we might have is-
```
P[m]

a : 60%
i : 25%
o : 10%
. : 5%
```

meaning if the current character is `m`:
- choose a most often
- sometimes choose i
- sometimes finish the word

## How generation starts
Every word starts from the boundary symbol: `.` The model does not know "start", so we represent beginning of the word as `.`
<br>

So we will have: P[.]
example:
```
P[.]

a: 20%
e: 30%
m: 10%
l: 40%
```

Hence for the model the first letter would probably be one of these.

## Weighted random choice
This is an important concept. We donot choose the character with the highest probability.
Meaning we donot always do 40% → l therefore choose 1. Because then every generation would be identical.
<br>

By using sampling methods like torch.multinomial(), we introduce controlled randomness.
A good analogy to understand this is-
<br>

"Instead of automatically picking "l", torch.multinomial() acts like spinning a roulette wheel where the slots match those percentages.40% of the wheel is "l" 30% of the wheel is "e" And so on...If you spin it 10 times, "l" will likely win about 4 times, but "a" or "m" will still win occasionally. This is how AI generates creative, diverse, and different responses to the same input."
<br>

I found this analogy really helpful to understand the use of torch.multinomial().
Hence: <b>torch.multinomial() draws random samples from a given probability distribution<b>
<br>

Example generation:
- At thr start we have `.`.
- Model looks up at P[.] and say it picks `e`
- Next it looks up P[e] and say it picks `m`
- Next it looks up P[m] and say it picks `m`
- Next it looks up P[m] and say it picks `a`
- Next it looks up P[a] and say it picks `.`
- The dot means stop

Removing the boundary symbols, the output is: `emma`

# Step 5: Using P to measure quality
Until now we built a model that can generate. But now we want one number that summarizes: "How well does this model explain the real data?"
<br>
We will measure loss. Low loss means the model is good whereas high loss means the model is bad.
We have two things- the probability matrix and the actual data, which has the events that actually happend. 
<br>

Now we compare:
"What probability did the model give to the thing that really happened?"

## 1. Look up the real transition
Lets take `emma` for example. One real bigram is: e → m. Now we go our probability table: P[e,m] say it is equal to 0.125. Which means that the model says: given e, there is a 12.5% chance that the next character is m.
<br>

Important rule: We take only this one value, not all probablities after `e`, only the probability of what actually happened.
## 2. Take the log
We have: P(e→m)=0.125, now we will take log: log(0.125)
<br>

Why? Because probabilities would multiply.
<br>

For the word: `emma` <br>

The probability is: P(.e)×P(em)×P(mm)×P(ma)×P(a.)  ->This would be a very small number. Log turns multiplication into addition. 
```
log(ab)=log(a)+log(b)
```

## 3. Add all log probabilities
We start: `log_likelihood = 0`
<br>

Then for every bigram we add (example):
```
. → e
e → m
m → m
m → a
a → .

log(P[. , e])
+
log(P[e , m])
+
log(P[m , m])
+
log(P[m , a])
+
log(P[a , .])
```

At the end: `log_likelihood` is a big number.

## What does this number mean?
A good model: assigns high probabilities.
Example:
```
Real event: e → m
Model says: 90%
Log: log(0.9)=> which is close to: 0=> Good.
```
```
Bad model:
Real event: e → m
Model says: 1%
Log: log(0.01)=> which is a large negative number.=> Bad.
```

## 4. Negative log likelihood
The problem is log likelihood is negative. We prefer lower loss (better).
Example:
```
Good: -20
Bad: -200
```
So we flip the sign:
<br>

NLL=−log likelihood
```
Now:
Good model: 20
Bad model: 200
```

Now lower NLL is better, just like loss.

## 5. Average NLL
Suppose:
```
Dataset A: 100 transitions
Dataset B: 10000 transitions
```
The bigger dataset naturally creates a bigger number. So we take an average, where n is the number of bigrams.
<br>

Average NLL = NLL/n

# Where this is heading
We want a model that can learn patterns without us explicitly building the probability table.
The bigram model works because the problem is tiny. We only look for the probability of the next character by looking at the current character.
<br>

Hence for: current character → next character, there are only 27 X 27 possible relationships. So we can literally count them.
<br>

But imagine we want: previous 10 characters → next character
For previous one character 27 possibilites were there. For ten: 27 raised to the power 10.
Such a large matrix would be impossible to store and count, hence we need to build something smarter.
<br>

One thing we have is very important and that is the average negative log likelihood. This is the loss, a good model will have lower loss and a bad model will have larger loss. The loss tells us how far away is the model from the data.
