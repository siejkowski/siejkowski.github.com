---

layout: layout
title: "Should project structure be based on architecture or features?"
author: Krzysztof Siejkowski
date: 2016-03-15
keywords: project structure, architecture, functionalities, folders, names
description: An article describing why there's no definitive way of organizing the source code and what can be done to improve project structure readability.

---
# [{ page.title }]({ page.url })

Not a long time ago there was a discussion in the team I was working in concerning whether the project structure should be based on app's features or on its architecture.

The spark that ignited the argument was [the post on Medium](https://medium.com/the-engineering-team/package-by-features-not-layers-2d076df1964d#.p2cemmvb7) about the perceived absurdity of splitting packages in Android project into layers instead of functionalities. Please [go read the original work](https://medium.com/the-engineering-team/package-by-features-not-layers-2d076df1964d#.p2cemmvb7) by [César Ferreira](http://cesarferreira.com) if you're unfamiliar with the topic.

The discussion, although rooted in the Java package system, can be easily generalized to all of the programming languages that I know of. The usual structure of a project is determined either by the structure of filesystem (directories and files) or by how the build tools store their meta information. Nowadays, it's either XML, JSON, YAML or similar format (see Xcode `project.prxproj` file). Feel free to [tell me about exceptions](https://twitter.com/_siejkowski). 

## Build tools ❤️ tree-like data organization

One thing all of these forms of data organization (XML, filesystem, JSON etc.) have in common is that they are loosly based on the principles of [tree structures](https://en.wikipedia.org/wiki/Tree_(data_structure)). They share two characteristics:

* hierarchical: each connection between information has a direction,
* non-cyclical: each piece of information is accessible via distinct path.

I'm not going to dive into the graph theory for mathematical definitions and analysis. All we need to know is there's always a parent node the object belongs to (except when it's the root) and there's always only one parent node at the time.

Let's use the name *folder system* for such a tree-like structure. It's widely used to organize data in notes apps, mail clients and so on. The common alternative is *tag system*, such as labels in Gmail or contexts in GTD. Each object can have multiple labels at once. 

However, I've yet to see build tool that keeps the information based on tags. Other common way of working around limitations of *folder system* is linking without copying (such as symbolic linking in filesystems or hyperlinks inside JSON or XML). Again, I haven't come across build tool relying on links.

So we're mostly stuck with *folder system*, what's the problem? Can't we just organize any source code into directories? We can, and we do. But we cannot carry enough information about the inner workings of our application through the tree-like structure. It's fine for data organization, but has serious limitations when it comes to visualization of what the source contains.

## Why doesn't the code fit into *folders system*?

Tree structures are great for expressing the hierarchical data, such as text. Imagine this blog post as a tree: each character has a root that represents word, each word has a root that represents sentence, each sentence has a root that represents a paragraph and each paragraph has a root that represents the whole post. Pretty straightforward, isn't it? By analogy, we can easily represent the source code in the similar way. In fact, many compilers and runtimes use the abstract syntax trees as their inner data organization structure. 

However, while the text in source files is just as linear as text printed on a sheet of paper, the code execution is not. Each computer program is highly non-linear and hyperlinked. Just remember how many jumps are there each time you follow the debugger. In fact, multiple programming language constructs are designed for making the linear representations of non-linear execution. Think of switches, loops or conditionals as examples.

Because the execution is non linear, there is no definitive way of organizing the source code. The application code can be looked at from numerous perspectives. Just to name a few: 

* architecture: views, models, presenters, view models, aggregation roots, use cases, network clients, database clients, 
* functionalities: bounded contexts such as registration, login, 
* threads of execution: what is sync and what is async, what is foreground and what's in the background,
* scope: what is public and what is private, 
* purpose: belongs to tests and what belongs to production code, 

There are many more possible ways of looking at code. Some are valid only for particular types of projects or technology stacks. This is exactly what makes it hard to reason about computer programs.

Usually, it should be possible to fit each one of those perspectives into tree-like structure. There's even a benefit to that. If you're having some dificulties in identifying what architecture layer or bounded context particular function or class belongs to, great! More often than not it's a sign of lack of separation between components (tight coupling) or of hidden dependencies. *Folder system* can give us hints at what to improve in our app.

However, to fit all the possible perspectives at once into a single tree-like structure is simply impossible. The good analogy is light, which can be seen as either a wave or a particle. There's no definitive model: in some cases it's easier to use one or another, depending on what you're trying to describe. However, using the *folder system* for code organization limits the number of possible models to one at the time. It's as if a higher-dimensional structure is projected onto a lower-dimensional space. When you transform from a higher number of dimensions to a smaller number of dimensions, there are many possible projections. For each one some piece of the information is lost and some is preserved. You need to choose what to keep. 

## Does the project structure matter?

For an average build tool the naming of files and directories in your project doesn't matter. There might be some conventions about where the sources should be kept, but they're usually configurable. For the fellow programmer, however, the issue becomes important. Do you remember last time you've tried to navigate around the non-trivial application to apply a small change? Or when you've been starting to work on a legacy project? The good project structure might greatly improve the readability and maintainability of the source code. 

The more basic tools are used, the more important this becomes. Think of Github web interface which shows you only the list of clickable files and directories. Even then just by looking at the project structure one might get a lot of information: what's the app architecture? What are the building block? What are the main features? What is public and what is private? What belongs to production code and what belongs to tests? The problem with *folder system* is that it cannot carry all that information at once. It makes us pick and choose.

## Do we need to pick our poison?

So the tree-like structures are insufficient for code organization. But they are also unavoidable, because there's a number of tools based on *folder systems*. What can we do?

I see three possible solutions:

* pick your poison - choose one perspective that's best for your particular application,
* decouple the inner organization of the code from the project structure,
* use tools that let you challenge the shortcomings of tree-like structures.

As far as picking your poison goes, I believe there's no point in arguing what project structure is the proper one without sufficient context. Answer is, as almost always, "it depends". If you're using the architecture that is not widely known or is based on some guarantees that are not enforceable by compiler, linter or code review, let the project structure reflect the architecture. If you're working on a feature-heavy app or you want to make it obvious where to look in case of user-facing bugs, make the project structure based on features. There are more things to consider, so choose carefully.
 
The basic idea of decoupling the code organization from the project structure is to use an alternative tool for documenting your code. There's an abundance of possible ways, to name a few:

* writing extensive comments in the source code,
* providing documentation files along the source code (see [RxSwift documentation folder](https://github.com/ReactiveX/RxSwift/tree/master/Documentation)),
* creating wiki for your project (see [AFNetworking wiki](https://github.com/AFNetworking/AFNetworking/wiki)),
* making a separate website (see [akka.io](http://akka.io)),
* setting up a space for project inside your intranet (if you use intranet),
* pdfs, books, video tutorials etc.

While the actual choice will depend on the particularities of your project, there're two things that all these ways have in common. The good one is being able to provide either multiple *folder systems* or moving into *tag system* for code description. It'll let you provide extensive information for the newcomers to your project who're trying to grasp its workings. The bad one is that there's a need to keep the documentation and the source code in sync, which is an additional work. Project structure, with all its shortcomings, gives us that sync for free.

The last option is my favourite one, but at the same time it's the hardest to get. Wouldn't it be nice to use the tools that let us visualize the app from multiple perspective just by analyzing it's source code? Maybe there's gonna be some additional work needed - like special comment format, annotation or parsing configuration - but once it's done, the actual visualization would happen automatically. If you've used one of [JetBrains](https://www.jetbrains.com)' wonderful IDEs, there's a "Structure" view that show you the class hierarchy of the app. Could there be a tool that let's you do the same thing - providing alternative perspectives - but is configurable by the user? I haven't heard of any (if you know some, [please share with me!](https://twitter.com/_siejkowski)). I'd love to see more great code visualization tools, especially for Swift and Objective-C. However, the amount of work needed makes this option the least applicable for now.

I'd like to leave you with one thought: even if you're stuck with *folder systems* for code organization, here's a huge amount of possible structures, way more than just "architecture vs features". Each of them applicable in different context, so whatever choice you make, make it deliberate.

I'm more than happy to discuss the article or any other topic related to software development. Contact me [@\_siejkowski](https://twitter.com/_siejkowski). 

### { page.date | date: "%Y-%m-%d" }
