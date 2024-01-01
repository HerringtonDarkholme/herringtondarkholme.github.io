title: How I build a chatbot for my OSS project, for free, without code!
date: 2024-01-01
tags: AI
---

Hi all! Happy New Year! I am Herrington, the author of [ast-grep](https://github.com/ast-grep/ast-grep).
Today I am going to show you how I build a [discord bot](https://discord.com/invite/4YZjf6htSQ) for ast-grep that helps people to understand the tool and help them write patterns or YAML rules.

And I get it done free, without code! You can try the bot via [ast-grep's discord channel](https://discord.com/invite/4YZjf6htSQ)!

# What is ast-grep?

Before we dive into the details, let me give you a brief introduction of ast-grep. It helps you to understand why I need a chatbot and what will the bot do.

[ast-grep](https://ast-grep.github.io/) is a command-line tool that lets you search and transform code written in many programming languages using abstract syntax trees (ASTs). ASTs are data structures that capture the syntactic and semantic structure of source code. With ast-grep, you can write patterns as if you are writing ordinary code, and it will match all code that has the same syntactical structure. And if you need more power, you can use YAML, a rule system that allows you to write more sophisticated linting rules or code modifications.

# Why is it hard for users to learn?

Sounds cool, right? But there's a catch. ast-grep is a new tool, so users have no prior knowledge and are less exposed to it compared to other established tools. Understanding ASTs requires some knowledge of programming languages. ast-grep is also feature-rich, so users need some time to find the relevant documentation. Writing YAML rules requires knowledge of the rule system. People don't know whether they should opt for patterns or YAML to complete their tasks.

And, finally, we are all hasty nowadays and have no patience to read the documentation of a tool. We want to get started quickly!

# How can a discord chatbot help?

That's why I decided to build a discord chatbot for ast-grep. A chatbot that can help users understand the basic concepts about ASTs, get people started quickly, guide them to the most relevant documentation, and even help them to write patterns or YAML rules! It's like having a personal tutor, available 24/7!

How did I do it? Well, I used [coze](https://coze.com/), a free gpt-4, no-code workflow system that lets you build chatbots for any purpose. And the best part is, I did it for free, without code!

coze has four main features like GPTs, except that it is free:

* Custom prompting: you can write your own prompts to control the flow of the conversation, ask for user input, validate the input, and generate responses.
* Knowledge base retrieval: you can upload text files as a knowledge base, and coze will automatically segment it and index it for retrieval. The bummer is that you can upload at most ten files. :(
* Function call: you can invoke any external API from your prompts, and use the results in your responses.
* Database access: you can store and retrieve data from a built-in database, and use it in your prompts and responses.

With these features, I was able to build a chatbot for ast-grep in a few hours. Here’s how I did it:

## Curating the Knowledge Base

To build the knowledge base, I took a simple approach. I concatenated all the markdown documentation from ast-grep's official website! And voila, coze did the segmentation for me. It was like having a personal librarian organizing my tech library!

## Building the Routing Prompt

The next step was to build a workflow to instruct GPT to write a pattern or a YAML rule for the users.

However, adding all the details about patterns and YAML rules to one single prompt is too much for GPT4 to learn and handle.

So I used a workflow to break this task down into smaller tasks and use a routing prompt to decide whether the task can be done with a pattern or a YAML rule.

The workflow looks like this:

{%img /images/ast-grep-bot.png%}

And the routing prompt is something like

```
You're an ast-grep assistant who specializes in guiding users in and about the ast-grep tool.
ast-grep has three kinds of usage: using pattern, using YAML rule and using API. Your task is to identify the most proper
use case.
* Pattern can match simple and straightforward expression/constructs in programming language.
* YAML can match complicate expressions and take the surrounding AST around the expression.
* API is the most flexible and powerful tool for users to implement their own matching logic. Custom matching should use API.

# Example
Q: "How can I match console.log in JavaScript"
A: "pattern"
Explanation: `console.log` can be searched trivially with pattern

// ... more examples omitted

Please decide which usage is the most proper for the following user question:
User Quesiont {{input}}
```


## Teach the bot to write patterns or YAML rules

The next step was to teach the bot to write patterns or YAML rules for the user. For this, I used a specific and in-context-learning approach, where I provided the bot background knowledge on rule and pattern in the prompt with examples.

```
You're an ast-grep assistant. Your task is to help user to write a patter based on users' request.

# ast-grep's pattern syntax
## Pattern Basic
// explaining the basic syntax of ast-grep pattern...
## Meta Variable
// explaining the basic syntax of ast-grep pattern...
* Example: `console.log($GREETING)` can match `console.log(123)`, but not `console.log()` nor `console.log(1, 2, 3)` because GREETING does not match one single node.

# Example
Q: "Write a pattern for console.log in JavaScript"
A: console.log($$$A)

================

Please answer user's question:
{{input}}
```

# Create a bot avatar!

The last step was to use Bing to generate a bot avatar. The default avatar placeholder is boring and I wanted to create an attractive avatar that represents ast-grep.

So I used Bing image creator! I uploaded the ast-grep logo to Bing. I also added its description and some extra keywords to make it more interesting. Here is what I wrote:

```
Can you draw me a avatar image of the ast-grep bot, an assistant bot to answer users' question about ast-grep and write pattern/rules for them, with the element of ast-grep's logo, a yellow lightning bolt and dark blue letters SG. The bot should be cute, cartoon sketch style.
```

And here is the result I picked up!

{%img /images/bot.jpeg%}


# Link it to discord!

To link my chatbot to discord, I followed the instructions [here](https://www.coze.com/docs/publish/discord.html).
It required some setup but not too hard. I created a discord bot account, granted it authorization, invited it to my server, and connected it to my coze bot. Then I tested it with some questions, and it worked like a charm!

![ast-grep screenshot](https://pbs.twimg.com/media/GCaGKMQbIAAts1y?format=jpg&name=large)

# That's it!
And there you have it, folks! A chatbot ready to guide users through the ast-grep world.

Building this bot was a fun and enlightening experience, and I hope you found this journey equally enjoyable.
Though the bot creation does not require any coding, it still requires some knowledge of the tool. And analyzing your use-cases and using the workflow strategically will greatly improve bots' responses!

Until next time, keep exploring the magic of tech!
