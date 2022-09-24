---
title: "Understanding Large Unfamiliar Codebases"
date: 2022-09-23T15:51:58+05:00
summary: "There are several things you can do to ease your way into an unfamiliar codebase:"
draft: false
---

# The Problem
Right off the bat let me say that when you join a new company and enter its codebase for the first time, it is bound to be difficult. No matter what you do, it won't be as easy as working on a project that you built from scratch.

# The Solutions
There are several things you can do to ease your way into an unfamiliar codebase:

## Understanding the Product
If you don't know what a piece of code intends to do, you will have a hard time understanding how it does it. Trying to understand code without first understanding what it does is like trying to put together a 1000-piece puzzle without first knowing what the end result should look like.

The following are some pro-tips regarding this:

- If the codebase has **selenium tests**, run them and watch. You will learn how a feature works end-to-end.
- When trying to understand a single unit, such as a React Component, reading its **unit tests** may give you some useful hints.
- You can also **write tests** on the feature you are trying to understand. It helps you deeply understand behavior.

## Removing Things
A great way to see what a line of code does is to remove it and see what changes. This way, by removing lines one at a time, you can understand what many of them do.

## Taking up tasks
No matter how much you don't want to hear this, true understanding will only come when you work with and extend the code you're trying to understand. Try to take up tasks regarding the code you want to understand, such as fixing bugs in it, adding new features, or improving its performance.

## White Boards and Markers
This works well especially when trying to understand full stack applications or complex React component trees. In React, for example, you can draw all relevant react components and their state, then try to visualize props being passed to children, and changes bubbling up. This helps you see the bigger picture and not get lost in implementation details.

## Browser DevTools & Extensions
These are especially helpful when it comes to understanding network requests & responses, React components & state, and Redux stores. The Redux devtools extension even has a time-travel feature where you can replay state changes over time.

## Reading GitHub
Read Pull Requests that created (or made changes to) the component you are trying to understand. Inside the PR, try to find the commit history. There, hopefully, you will see the component being built little by little incrementally and that will be easier to understand with the aid of commit statements. The PR may also contain information about why the component was written in the first place.

## Documentations & Pagers
Any good organization will definitely have documented their code or written pagers on at least the major parts of the codebase, such as database models, cloud deployment, etc. Reading them can help.

## Asking Questions
Finally, as a last resort, you can always ask someone what a piece of code does. I call this a last resort as this doesn't result in a deep understanding of the code.