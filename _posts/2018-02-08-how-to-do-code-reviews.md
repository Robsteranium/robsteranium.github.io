---
title: How to do Code Reviews
layout: post
date: 2018-02-08
tags: code review
redirect_from: "/2018/02/08/how-to-do-code-reviews/"
---

Reviewing merge requests is a peculiar skill in and of itself. It's remarkably dissimilar to writing code! It also draws on your interpersonal skills as much as your technical knowledge - not something us geeks are reknowned for...

Based upon some research and practice, I've tried to distill some guidance for reviewers and for those submitting code for review.


## How to review code

First, some tips for the reviewer.

Don't try to understand everything in one pass, make multiple passes over the code. This might seem painstaking, but it should actually be quicker because the cognitive burden is lower.

Start with a definition of the purpose. This should be in a ticket but it may help to explore the context with the people creating the requirements too.

Use a first pass to get a handle on what's changed. Don't worry about the implementation. Instead look at which files and modules were affected. Maybe bear in mind common calls or data structures.

Start taking notes with a list of questions. If something peculiar catches your eye, take a note and move on. Resist the temptation to pursue a complete understanding or you'll find yourself exploring in 10 different directions at once.

You might then extend your reading to see what else is involved - exploring parts of the code that interact but haven't been changed directly. Again, you're looking to understand the structure, not the implementation detail at this stage. The questions you just wrote should help. You can expand on them here and maybe even find some answers already.

Now you should have the overview necessary to think about where to start your review and how much ground you need to cover. The application or domain may lead you to a particularly obvious starting point. You might like to start with the thorniest area, or perhaps you'd prefer to start the balling rolling typos or minor refactorings.

Try to imagine writing instructions for yourself, as if you were going to make the changes you're suggesting or as if you're writing to a future version of yourself who's got to come back and understand what happened.

Don't be like Torvalds. Take great pains to avoid any personal form of criticism. Be a teacher or a sympathetic friend not a drill sargeant. Your expertise is wasted if you try to beat people over the head with it. Refer to the code, not the author. Avoid words like "you" and say "we" instead - the author and reviewer are collaborators not opponents.

Be modest - it's often easier to spot mistakes from a distance. Don't let yourself believe that you wouldn't have made the same mistakes were you the code's author. Imagine that the author already knew about your suggestion, had considered that themselves and decided on an alternate course of action. Perhaps they know something you don't.

You're making suggestions not giving instructions. Substantiate your ideas with evidence if necessary. You need to convince the author to see this from your perspective, to believe these ideas too. Just because it's obvious to you, it doesn't follow that everyone will see that or even agree with it.


## How to submit code for review

The responsibility for a good code review doesn't just lie with the reviewer. Those authoring the code can also help the process along.

Provide some commentary, through commit messages or by putting comments against a line or code block (tools like Github/Gitlab allow you to do this outside of the source file in version control, so the comments exist only for the purpose of the code review).

he most valuable thing you can do is to help the reviewer decide where to focus their attention. This might be to direct them to a tricky part of the application logic, or a request for advice about a framework or API that the reviewer knows about. You can also use these channel of communication to explain your decision-making process or to present and discuss options for next steps to the reviewer.

Ensure that the amoung of code submitted for review hasn't gotten too long. Because of interactions, the complexity of even well factored and decoupled code can grow non-linearly with the time that goes into writing it. You the author have also benefitted from the increasing returns to scale that come from building up context. This means that it will get harder and harder for others to review your work at an accelerating rate. Small frequent changes are good for collaborators as well as the code base. This might mean you ought to request a review before a feature is fully implemented.

Don't take it personally. It can be difficult to take criticism on something you worked hard on. Particularly since your reviewer starts with the benefit of hindsight and freedom to reflect and pontificate. Indeed you may find comments based upon theoretical patterns or architectural ideas fail to work out in practice! You can help to keep things constructive by remaining objective - defend your decisions by all means, but try to remain open to alternative views. Talk about the code not the coders. Be your own worst critic.
