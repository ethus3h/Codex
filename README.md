Codex - Source code language classification
===========================================

This small library is an attempt to classify source code using a support vector machine (SVM) classifier.

It doesn't work very well currently but is still in development.

Motivation
----------

The motivation for writing this library was to develop a [Discord](https://discordapp.com/) bot that detects when someone posts code, automatically formats it with backticks and sets the proper language. This requires that the bot be able to:

1) distinguish code snippets from legitimate conversations (most important)
2) recognize which language a code snippet is written in (less important)

The most obvious way to find out if text is of such and such programming language is to just attempt to parse it according to the language grammar. However, this approach quickly runs into limitations once one realizes that the input may not be a complete block of code (for instance just a small snippet of a larger source code file) and may not necessarily be valid code according to the rules of the language, especially for languages with many different dialects.

This library takes a different approach; instead of rigorously parsing the input text, it attempts to tokenize it according to a chosen set of patterns, called **markers**, which are likely to occur in different programming languages (regular expressions are used for pattern matching). An example of a marker is the following regex

```
^\s*#\s*include\s+("|<)[^">]+("|>)$
```

which matches C/C++ `#include` statements. The basic algorithm then goes as follows:

1) For each marker, find all non-overlapping matches of the pattern in the input, calculate how many characters of the input were matched, and divide this number by the length of the text in characters; this gives the *weight* of the input text for that marker.

2) Sum up all the marker weights to obtain the text's *score*; if this score is below some configurable threshold - which in some vague sense describes how much (of) the input text looks like code - then reject the input text as not code.

3) Otherwise, put all the marker weights in a vector and determine the closest language using a linear support vector machine (SVM) classifier.

Some preprocessing takes place on the input text prior to the algorithm above. In particular, free-form syntactic elements such as comments and string literals are *completely* removed and replaced with a single space; these are not particularly useful for classification, as many languages share identical comment and string literal syntax.

There is a special case where such constructs can lead to severe ambiguity between languages. An example is the Delphi curly brace comment construct, which represents a compound statement in the C family of languages. These are removed **only** if the matched text **doesn't** look like code according to steps (1) and (2) above. Fortunately most cases are not ambiguous, the only known ambiguous cases are given below:

 * `{...}`: Block comment in Delphi, compound statement in C-like languages

As a general rule, misclassifying the input as the wrong language is preferable to mistakenly detecting normal text as code and vice versa, whenever possible.

Usage
-----

TODO: dataset stuff

For now please use the pre-trained dataset in `dataset/dataset.bin`.


To initialize the classifier:

```python
>>> import codex
>>> classifier = codex.Classifier('dataset/dataset.bin')
```

To classify a piece of code:

```python
>>> classifier.classify("""
... #include <stdio.h>
... 
... int main(void) {
...   printf("hello\n");
... }
... """)
<Language.C: 'C'>
```

The `classify` method will return `None` or a best guess if it cannot ascertain the language. In particular, it will return `None` if the input isn't code but is ordinary text.

The classifier can be retrained on additional data with `cls.train`.