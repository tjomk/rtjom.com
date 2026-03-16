---
title: "Building a Personal Mnemonics Bot"
date: 2026-03-16T07:29:13+02:00
draft: false
---
When was the last time you memorized a phone number? When was the last time I memorized one?

The answer to this question is about 10 years ago when I changed my mobile phone number. At the dawn of the mobile phone era I used to remember lots of phone numbers.
There was no other way around it. I did have a small address book with a bunch of numbers of my friends, but most of them have been in my head. These days you just
write it down in your phone, it syncs to the cloud, and as a result it is forever with you. But what happens if you don't have a phone at hand?

We've been teaching our children to remember our phone numbers, but at the same time us adults have never made an effort to remember theirs. And so I decided to change that
and make an effort to remember family numbers.

## Anki and spaced repetition

For the past few years I've been using Anki on my phone to recall Japanese hiragana, katakana, and some basic kanji. Over time I also started adding some interesting words
I've been encountering from books, interviews, podcasts, and blog posts. Here's how it worked:

1. I encounter a word I haven't heard before
2. I go to [Wiktionary](https://en.wiktionary.org/wiki/Wiktionary:Main_Page) or [Wikipedia](https://www.wikipedia.org/) and look up this word
3. Open Anki and add new card. The front side would be the word, and the back side is its definition

Then Anki would remind me to check the new words. At first it would nag me daily. Then as I slowly memorized the new words, it would notify me with an ever-increasing delay
so that I do not forget the word.

However that approach did not work for me. Every interaction felt like a chore. Ugh, I need to spend 2 minutes on this task again. I didn't want to put any effort into remembering
the new words based on their dry definition. I didn't get any rewards after successfully completing the session. And so I started skipping reviews. Sometimes for weeks. Naturally
I didn't remember much of the words.

## The turning point

The turning point came after listening to a few different podcast episodes where participants easily threw various facts, dates, terms, and concepts to support their ideas.
Some would recall historical events with high precision. Others would explain the concepts using beautiful vocabulary. How can I be like them? How can I remember so many facts myself?
This is when I decided to put more effort into remembering information.

## Building a system

The first issue that needed a change was to stop using dry definitions and start putting a real effort into the description. It's like memorizing math and physics formulas. These
are just random set of characters that will disappear from your head the moment you pass the exam. Same with words. You start with the definition, then add few examples with this
word. Include familiar concepts, people, places in these examples to help you easier remember this concept. An example for a word `logorrhea` could be that a friend of yours
contracted logorrhea after visiting another country and eating expired food there. This resulted in them starting to uncontrollably talk and without being able to stop. Obviously
this is a very contrived example, but it is flashy enough to remember the word.

With the improved texts things were getting better. And now we're coming back to the phone numbers that I wanted to memorize. The most popular algorithm I found is called the
[major system](https://en.wikipedia.org/wiki/Mnemonic_major_system). Here's how it works:

1. Convert numbers to consonants (b, c, d, f, etc). Each digit can have one or more consonants. You can also assign pairs to bigger numbers, e.g. two or three digit numbers
2. Split the number into digits and map them to consonants
3. Come up with a word that contains these consonants, e.g. 314 becomes **m**e**t**eo**r**
4. Create some kind of association with the word: A bright meteor made a crater of size πr²

You can use existing mappings or create yours. In the end I went on to create mapping myself which felt easier to remember. But how do you memorize it? You add cards for each
digit. But what if I want to quickly see the whole major system? Tough luck in Anki. And this was the moment when I thought I'd spend few evenings using AI to create my own
spaced repetition system.

## Telegram Bot

I already wrote about using Telegram as the [RSS reader](blog/2021/03/telegram-is-the-new-rss-reader/). I also use it for various other tasks. Adding extra functionality felt
much easier than creating a separate phone application.

Getting the first working prototype was easy. It works the same way as Anki: create front of the card, then write the description on the back, and save. I have added a command
to enhance any individual card with AI. This performs 2 actions: create 1-2 sentences explaining the word etymology in the same language as the word, and then 1-2 more sentences
with mnemonics so it is easy to see how the word can be used.

I have also added option to memorize numbers, simple numbers, long numbers, and dates (which are just long numbers with added context). One can use the enhance command on
number cards too. And it is possible to quickly see the full mapping (major system) to refresh it all in memory.

## Expanding to more use-cases

The above system was already good enough to use. However in addition to numbers and words, my other goal was remembering events, facts, names, etc. There are many different
use-cases here, some that can be solved by spaced repetition, some that aren't. However the underlying algorithm for memorizing it is still the same: [the method of loci](https://en.wikipedia.org/wiki/Method_of_loci).
It involves creating/visualizing a place that you can visually walk around and attach different words to objects in there. Imagine your house, how you enter it. What objects are there
that you see first? What do you do next, where do you go? As you go, place different things on this route and link them to the words you want to remember. Derren Brown has
a great book called [Tricks of the mind](www.goodreads.com/book/show/945683.Tricks_of_the_Mind) which explain many of the memory techniques in detail.

After implementing this algorithm, now when adding new cards you go through a multi-step setup where you define type of the card, add description, enhance, etc. Along the way
you can use AI to create the visuals for you. I use a generated palace with many rooms for memorizing linked lists. Technical implementation is worth a separate blog post.

## Gamification

In the end creating this tool was more fun than actually using it. The moment I saw notifications to review words, I had the same feeling as with Anki: this is another chore.
And so I decided to try and gamify the experience a little:
- Add daily streak tracking to motivate myself to continue learning daily
- AI-generated streak messages. I split these messages into 2 groups: positive and negative. With positive being a cheerful message shown about 30% of the time, and negative being akin to "Do not lose that streak that you've already achieved"
- In 20% of cases after completing a review session I am shown a random Wikipedia article as a reward. I thought that a surprising Wikipedia fact would be a great reward, and knowing that would hook me to continue

These gamification features have been implemented only recently, so I don't have much to say if they work or not. Time will tell.

In the next blog post I will share more details on the Loci method and gamification implementation details.
