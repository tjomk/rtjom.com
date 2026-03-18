---
title: "Building a Personal Mnemonics Bot"
date: 2026-03-18T07:29:13+02:00
draft: false
---
Every now and then I encounter a new word that I've never heard before. Or need to memorize the shopping list, a phone number, a date,
or a fact. In school I didn't have any system to do that other than just read the information as many times as needed for it to stay in
my head until it is no longer needed. But as I grow older, I want to retain the information for much longer.

Whenever I hear people like Naval Ravikant talk on podcasts, I am always amazed by how much they know, how many dates, facts they are
able to recall during the conversation. There are many ways of achieving this, from thoughtfully reading books, discussing them, taking notes,
and then revisiting those notes, to organizing your personal knowledge graph using tools like Obsidian.

Below I will describe one of such journeys I am currently undertaking in order to retain small bits of information like names,
phone numbers, dates, facts, and words, and be able to recall them any time I need.

## Anki and spaced repetition

At the university I studied Japanese. When you rarely use the language, if at all, it tends to quickly escape from your memory.
I didn't want to completely forget it, so I looked into the different language learning apps. That didn't stick as I kept forgetting the
basic characters. This is when I discovered [Anki](https://apps.ankiweb.net/) - an app built for spaced-repetition.

The way Anki works is you create a card with two sides: the front and the back. On the front you write a word that you want to remember,
and on the back you create a translation. Then you group these cards into decks, e.g. hiragana, katakana, basic kanji, and practice these
decks. After practicing the card you can specify how well you remembered it, and whether you want to practice it again in the same session,
or later. Depending on your response, Anki would schedule this card accordingly: today, tomorrow, next week, or even next year. The more
you practice the card, the longer you can retain it in the memory, the less you need to revisit it in the app. This is how spaced repetition
works.


Over time I also started adding some interesting words I've been encountering from books, interviews, podcasts, and blog posts. Here's how it worked:

1. I encounter a word I haven't heard before
2. I go to [Wiktionary](https://en.wiktionary.org/wiki/Wiktionary:Main_Page) or [Wikipedia](https://www.wikipedia.org/) and look up this word
3. Open Anki and add a new card. The front side would be the word, and the back side is its definition

This worked with a handful of words, but as I kept adding more information like dates, numbers, facts, the system started showing cracks.
To begin with, all of the explanations were very "dry": a definition from Wikipedia is hard to remember, especially if it is not
followed by personalized examples. All of the cards are completely independent of each other: one mentions the date when the World War 2
started, and the other explains what "Melee" means. And then all of a sudden there is a phone number on the front of the card and the name
on the back. How do you remember that?

## Improving the system

The first issue that needed a change was to stop using dry definitions and start putting a real effort into the description. It's like memorizing math and physics formulas. These are just a random set of characters that will disappear from your head the moment you pass the exam. Same with words. You start with the definition, then add a few examples with this word. Include familiar concepts, people, places in these examples to help you more easily remember this concept. With the improved texts things were getting better when it came to remembering words. My next stop was the numbers. Specifically phone numbers.

The only two numbers I remember are my own and my mum's. I did want to also memorize numbers of my partner and my children.
How do you do that? The answer is the [major system](https://en.wikipedia.org/wiki/Mnemonic_major_system). Here's how it works:

1. Convert numbers to consonants (b, c, d, f, etc). Each digit can have one or more consonants. You can also assign pairs to bigger numbers, e.g. two or three digit numbers
2. Split the number into digits and map them to consonants
3. Come up with a word that contains these consonants, e.g. 314 becomes **m**e**t**eo**r**
4. Create some kind of association with the word: A bright meteor made a crater of size πr²

You can use existing mappings or create yours. In the end I went on to create the mapping myself which felt easier to remember. With that in
place, I now had to memorize the mapping.

Having added all of the necessary cards into the app, memorizing them felt like a chore. At times instead of going through digits one
by one I wanted to look at the full picture and review them together. At the same time I wanted a quicker way to add new words to my decks
and use AI to generate examples. This is where I had the crazy idea of building my own system.

## Telegram Bot

I already wrote about using Telegram as the [RSS reader](blog/2021/03/telegram-is-the-new-rss-reader/). I also use it for various other tasks. Adding extra functionality felt much easier than creating a separate phone application.

Getting the first working prototype was easy. It works the same way as Anki: create front of the card, then write the description on the back, and save. I have added a command to enhance any individual card with AI. This performs 2 actions: create 1-2 sentences explaining the word etymology in the same language as the word, and then 1-2 more sentences with mnemonics so it is easy to see how the word can be used. The bot supports the full workflow: adding cards, reviewing them, editing individual ones, and summoning the major system mapping at any moment.

As I was developing this system, I wanted to learn more about the different memory techniques and see if I could use them too. This is
where I found out about [the method of loci](https://en.wikipedia.org/wiki/Method_of_loci).

## The memory palace

The method of loci involves creating or visualizing a place that you can visually walk around and attach different words to objects in there. Imagine the house you live in. Visualize how you enter it. Open the door and enter into the hallway. What objects are there
that you see first? Put your clothes on the hanger and into the wardrobe. Leave your shoes on the deck. Enter the living room. Look around. Where can you go next? What objects are there in the living room? As you go, place different things on this route and link them to the words you want to remember. Now that you have this familiar place in mind, take a list of items, e.g. a shopping list, and start
placing these objects as you walk through your house. Morph your shopping items into the house objects: an entrance door made of broccoli
and the handle made of an apple. As you enter the hallway, you step on a carpet made of bread. You turn right to hang your coat, and the
hanger is made of two cucumbers. The more crazy these associations, the easier it is to recall these items. Derren Brown has a great book called [Tricks of the mind](www.goodreads.com/book/show/945683.Tricks_of_the_Mind) which explains many of the memory techniques in detail.

A simple example of a list would be the names of the planets in the Solar system starting from Mercury. A vivid example of the palace could start with the following illustration:

> On the left bridge, shrouded in crimson mist, where each step glimmers with a mercurial sheen, stands Mercury—a miniature statue of the winged messenger, cast from liquid silver, whose droplet-eyes flow into one another like a planet forever hastening along its orbit.

As you move, you encounter other planets with their description that not only help you remember the order as you memorize the path, but also describe how these planets look.

## Gamification

As an engineer with a new set of tools I enjoyed building this tool more than using it. Even with new features like the memory palace, using the application felt like a chore after several weeks. I made a full circle and came back to where I was with Anki, i.e. skipping sessions. After some thinking I came up with an idea of adding gamification to the process. After all, I know what brings me dopamine, I know a little bit of how online platforms hold my attention. So why not implement similar techniques here?

Reflecting on what motivates me, I came up with the following:

- Tracking and keeping the daily streak. When I used to journal, my initial goal was to do it daily so that exactly a year later I could read what happened on this day last year. Going forward I didn't want to break the writing streak that kept just growing. It's thrilling to see this number getting bigger. And it would definitely work in this case too
- AI-generated streak messages. This has the benefit of being different messages every time. I have also split these messages into 2 groups: positive and negative. With positive being a cheerful message shown about 30% of the time, and negative being akin to "Do not lose that streak that you've already achieved"
- A variable reward after completing a session. I spent some time trying to come up with a reward that would feel like an actual reward. There must be some randomness not only in the frequency when this is awarded (think of occasional wins in casino or lottery and the hope that the next bet will bring the victory), but also the reward itself. In the end I settled on a random Wikipedia page. I love learning new facts, and this sounded like something that could genuinely excite me.

These gamification features have been implemented only recently, so I don't have much data to say whether these worked or not. However reading the research on gamification was both exciting and terrifying to realize how social networks use these hooks to keep us engaged. What I can say for sure though is that I got the excitement to learn new things once again. I keep opening my Telegram bot and sending the `/review` command. I try to memorize all these words, and I try to use them thanks to better examples and explanations.

In the next blog post I will write about the technical implementation and the algorithms used.