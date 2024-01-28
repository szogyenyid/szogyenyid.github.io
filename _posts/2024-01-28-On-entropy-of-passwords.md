---
layout: post
title: "On the entropy of passwords"
date: 2024-01-28 15:05:00 +0100
categories:
    - learning
    - security
    - privacy
tags:
    - security
    - password
    - passphrase
    - entropy
    - cybersecurity
    - diceware
    - randomness
description: "Explore password security with our dive into entropy. From basics to Diceware, grasp the importance of randomness in passwords. Learn about CSPRNG, passphrases, and the simplicity of Diceware for robust cybersecurity in the digital age."
#last_modified_at: 2024-01-28 15:05:00 +0100
author: Daniel Szogyenyi
readtime: 7
---

The inspiration for this article struck when I was in the midst of implementing a shell script for diceware. Additionally, working at LastPass as a Senior Software Engineer has deepened my understanding of password strength and the critical role it plays in securing sensitive information. In this article, we delve into the concept of entropy in passwords, beginning with a fundamental understanding of information entropy.

## Introduction

In the ever-evolving landscape of cybersecurity, the pivotal role of passwords in safeguarding sensitive information cannot be overstated. This article aims to clarify the concept of entropy in passwords, casting light on its significance in fortifying digital security.

## Entropy of information in general

Before delving into the specificities of password entropy, let's lay the groundwork by establishing an understanding of entropy within the broader context of information theory.

### What is entropy in information theory?

Entropy, within the domain of information theory, serves as a metric for quantifying uncertainty and randomness within a dataset. The higher the entropy, the greater the degree of unpredictability exhibited by the data.

## Entropy of simple events

To demonstrate the calculation of entropy, let me show you some trivial cases, which can be extended to general ones.

### Entropy of coin flips

Contemplate the simplicity of a coin flip – an ideal coin manifests a perfect `1/2` chance of landing on heads or tails. If we want to reflect this in bits, the entropy if this system (the coin) is 1 bit.

It's easy to calculate (if the chances are the same for every outcome): just get the base-2 logarithm of the number of elements in the universe of outcomes. In this case the universe has 2 elements (heads and tails), so the entropy is `log2(2)`, which is 1 bit. 

What if we have 3 coins to flip? Let's see a truth table to better demonstrate the universe of outcomes:

| Coin #1 | Coin #2 | Coin #3 |
| ------- | ------- | ------- |
|  Heads  |  Heads  |  Heads  | 
|  Heads  |  Heads  |  Tails  |
|  Heads  |  Tails  |  Heads  |
|  Heads  |  Tails  |  Tails  |
|  Tails  |  Heads  |  Heads  | 
|  Tails  |  Heads  |  Tails  |
|  Tails  |  Tails  |  Heads  |
|  Tails  |  Tails  |  Tails  |

As you may see now, with 3 coins, the universe expanded to `2^3`, where 2 is the number of outcomes of an individual event, and 3 is the number of events. So the entropy is `log2(2^3)`, which is - according to logarithmic identities - `3*log2(2)`, meaning 3 bits.

As you may see, the value of entropy indicates the size of the pool of the possible outcomes. So something with 3 bits of entropy indicates 8 possible outcomes, while having 12.34 bits of entropy means that the pool contains `2^12.34` (~5200) elements.

So we can say that the entropy of some true random events can be calculated with the formula `n*log2(o)`, where `n` is the number of individual events, and `o` is the number of possible outcomes of a single event.

Note: The above formula only works if the probability of all the outcomes is the exact same!
{: .info-yellow }

### Entropy of dice

According to the above formula, the entropy from a single throw of a 6-sided die is `log2(6)` (as there are six possible outcomes), which is approximately 2.58 bits.

Transitioning to multiple throws, the entropy is about `n*2.58`, where `n` if the number of throws.

### Why true random (or a CSPRNG) is important

While coins and dice provide a semblance of randomness, the quest for true randomness, especially in the context of password generation, surpasses the cognitive capabilities of the human brain. Humans are just not able to think of three __random__ numbers. Herein lies the significance of Cryptographically Secure Pseudo-Random Number Generators (CSPRNG), crucial for crafting sequences that exhibit true randomness and resist predictability. To preserve entropy, and the equal probability of each possible outcome, true random is a necessity.

## Password strength and entropy

Now, that you understand what entropy is, let's see how does it influence the strength of passwords and their susceptibility to unauthorized access. 

### Entropy and password breaking

The interplay between entropy and password security becomes apparent when considering the formidable challenge of guessing a genuinely random password. As we delve into the dynamics of password-breaking attempts, the central role of entropy emerges as a key determinant of resilience.

As discussed in the chapter on coin flips, entropy of a generated password indicates the total possible outcomes of the generation, eg. 77 bits of entropy means `2^77` possible passwords.

Let's assume a malicious actor can try 1 quadrillion passwords each second (this may be the rate at NSA). One quadrillion is about `2^50`.

So if you generate a password with 77 bits of entropy, NSA needs about `(2^77) / (2^50)` so `2^27` seconds to try all your possible passwords. That's more than 4 years of constant work. Keep in mind, after trying 50% of possible passwords, it's more and more likely to get to yours, so `2^26` seconds is a better estimate, but that's still more than 2 years.

Note: there's a really low chance that they manage to guess your password on the first try. It has the same probability as guessing the password on the last try. As they try more and more unsuccessfully, the less untried remains, so the chance of getting to your pass increases, that's why we estimate with the average: your password is the one in the middle.
{: .info-blue }

### Password vs Passphrase

The creation of passwords involves the careful selection of elements from a predefined universe, be it individual characters or entire words.

#### Selecting Individual Characters

Traditional passwords encompass a universe of lowercase and uppercase letters (26+26 elements), numbers (10 elements), and special characters (~20 elements). The entropy of a single element from this universe is somewhere near 6.4 bits. So if you have a password of 12 __true random__ characters, its entropy is ~76 bits. Now imagine memorizing this: `W8J=0)rM$DRX`. Sounds painful, right? It's possible to memorize, but it isn't the most comfortable task.

#### Bunch of words: entropy of passphrases

In contrast, passphrases involve the selection of words from a vastly expanded universe. Assuming 7776 words (which is the length of the Diceware list provided by EFF), a single random word's entropy is 12.92 (`log2(7776)`) bits. To exceed the entropy of a 12 random character password, you have to memorize 6 randomly chosen English words, like `absolute gray pecan flatbed directive partner`. As these are words with meaning, it's not easier to imagine something and keep these 6 words in mind. No uppercases and special characters, just words, what a breeze!

#### Why passphrases are better

Passphrases having about the same entropy of a password have the added benefit of being more memorizable, present a compelling case for bolstering password security. The high entropy renders passphrases resilient against an array of password-breaking techniques, while their user-friendly nature enhances the overall authentication experience. But do not forget the importance of choosing your passphrases at random!

[Related XKCD.](https://imgs.xkcd.com/comics/password_strength.png)

### Diceware

[Diceware](https://theworld.com/~reinhold/diceware.html) is a method of generating true random passphrases. It's all about having a list of words (most commonly 7776 words), throw dice, and select the words accordingly. All the words have a number assigned to them (5 digits, all being in the range [1;6]), and as we consider the results of dice throws true random, the entropy of getting a single word from this list is `log2(7776)`, which is about 12.9 bits. Using this method, it's easy to generate true random passphrases.

If you would like to try a secure Diceware on your computer, feel free to try [my CLI tool](https://github.com/szogyenyid/diceware/).

## Conclusion

In cybersecurity, entropy is key to password security. From fundamental concepts to practical applications like Diceware, our exploration highlights the vital link between randomness and password strength.

Cryptographically Secure Pseudo-Random Number Generators (CSPRNG) play a crucial role in crafting truly random passwords. The exponential challenge for malicious actors attempting to crack passwords with higher entropy emphasizes the effectiveness of this approach.

Comparing traditional passwords to passphrases, we find that passphrases offer both high entropy and memorability. Diceware, leveraging true randomness, is a simple yet effective method for generating secure passphrases.

Understanding and prioritizing entropy in password creation is essential for robust cybersecurity. Whether adopting passphrases or embracing cryptographic advancements, focusing on entropy ensures resilient digital defenses in today's evolving digital landscape.