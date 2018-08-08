<div align="center">
    <img src="adversary_name.png"></img>
</div>

<br>

[![Build Status](https://travis-ci.com/airbnb/artificial-adversary.svg?branch=master)](https://travis-ci.com/airbnb/artificial-adversary)
[![MIT License](https://img.shields.io/dub/l/vibe-d.svg)](https://github.com/100/Cranium/blob/master/LICENSE)

This repo is primarily maintained by [Devin Soni](https://github.com/100/) and [Philbert Lin](https://github.com/philin).

## Introduction

When classifying user-generated text, there are many ways that users can modify their content to avoid detection. These methods are typically cosmetic modifications to the texts that change the raw characters or words used, but leave the original meaning visilbe enough for human readers to understand. Such methods include replacing characters with similar looking ones, removing or adding puncutation and spacing, and swapping letters in words. For example `please wire me 10,000 US DOLLARS to bank of scamland` is probably an obvious scam message, but `pl3@se.wire me 10000 US DoLars to,BANK of ScamIand` would fool many classifiers.

This library allows you to generate texts using these methods, and simulate these kind of attacks on your machine learning models. By exposing your model to these texts offline, you will be able to better prepare for them when you encounter them in an online setting. Compared to other libaries, this one differs in that it treats the model as a black box and uses only generic attacks that do not depend on knowledge of the model itself.

## Installation

```
pip install Adversary
```

## Usage

See [`Example.ipynb`](https://github.com/airbnb/artificial-adversary/blob/master/Example.ipynb) for a quick illustrative example.

```python
from Adversary import Adversary
gen = Adversary(verbose=True, output='Output/')
texts_original = ['tell me awful things']
texts_generated = gen.generate(texts_original)
metrics_single, metrics_group = gen.attack(texts_original, texts_generated, lambda x: 1)
```

### Use cases:
**1) For data-set augmentation:** In order to prepare for these attacks in the wild, an obvious method is to train on examples that are close in nature to the expected attacks. Training on adversarial examples has become a standard technique, and it has been shown to produce more robust classifiers. Using the texts generated by this library will allow you to build resilient models that can handle obfuscation of input text. 

**2) For performance bounds:** If you do not want to alter an existing model, this library will allow you to obtain performance expectations under each possible type of attack.

### Included attacks:
- Text-level:
    - Adding generic words to mask the suspicious parts (`good_word_attack`)
    - Swapping words (`swap_words`)
    - Removing spacing between words (`remove_spacing`)
- Word-level:
    - Replacing words with synonyms (`synonym`)
    - Replacing letters with similar-looking symbols (`letter_to_symbol`)
    - Swapping letters (`swap_letters`)
    - Inserting punctuation (`insert_punctuation`)
    - Inserting duplicate characters (`insert_duplicate_characters`)
    - Deleting characters (`delete_characters`)
    - Changing case (`change_case`)
    - Replacing digits with words (`num_to_word`)

### Interface:

**Constructor**
```
Adversary(
    verbose=False, 
    output=None
):
```
- **verbose:** If verbose, prints output while generating texts and while conducting attack
- **output:** If output, pickles generated texts and metrics DataFrames to folder at `output` path

**Returns:** None

---

#### Note: only provide instances of the positive class in the below functions.

**Generate attacked texts**
```
generate(
    texts,
    text_sample_rate=1.0,
    word_sample_rate=0.3,
    attacks='all',
    max_attacks=2,
    random_seed=None,
    save=False
):
```
- **texts:** List of original strings
- **text_sample_rate:** P(individual text is attacked) if in [0, 1], else, number of copies of each text to use
- **word_sample_rate:** P(word_i is sampled in a given word attack | word's text is sampled)
- **attacks:** Description of attack configuration - either 'all', `list` of `str` corresponding to attack names, or `dict` of attack name to probability
- **max_attacks:** Maximum number of attacks that can be applied to a single text
- **random_seed:** Seed for calls to random module functions
- **save:** Whether the generated texts should be pickled as output

**Returns:** List of tuples of generated strings in format (attacked text, list of attacks, index of original text). 

Due to the probabilistic sampling and length heuristics used in certain attacks, some of the generated texts may not differ from the original.

---

**Simulate attack on texts**
```
attack(
    texts_original, 
    texts_generated, 
    predict_function, 
    save=False
):
```
- **texts_original:** List of original texts
- **texts_generated:** List of generated texts (output of generate function)
- **predict_function:** Function that maps strings to classification label (0 or 1)
- **save:** Whether the generated metrics DataFrames should be pickled as output

**Returns:** Tuple of two DataFrames containing performance metrics (single attacks, and grouped attacks, respectively)

## Contributing

Check the `issues` tab on GitHub for outstanding issues. 
Otherwise, feel free to add new attacks in `attacks.py` or other features in a pull request and the maintainers will look through them.
Please make sure you pass the CI checks and add tests if applicable.

#### Acknowledgments

Credits to [Airbnb](https://airbnb.io/) for giving me the freedom to create this tool during my internship, and [Jack Dai](https://github.com/jdai8) for the (obvious in hindsight) name for the project.
