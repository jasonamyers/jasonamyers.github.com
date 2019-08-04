---
layout: post
title: "SICP with Dave Beasley"
description: "My thoughts after a week long look at SICP"
tags: [Beasley, class, SICP]
comments: true
share: true
---

He creates a space and invites you to explore it with him.  That's how I describe taking a class from Dave Beasley.  I view Dave as an inquisitive soul with a devilishly sharp mind and the heart of an explorer.

## Act I: Recursive Iteration Accumulation

His SICP class gives him the space to do just that, and take you along for the ride.  The course opens with an exploration of computing and programming in scheme (racket lang); however, this class isn't about that. This class is about exploring the composition of programs, how you can design them, and ultimately how you can create an interpreter.

This class has quite a bit of math in it, but not in an overwhelming way. It provides the basis for examples and allows for reviewing different solutions to a similar problem. You'll learn how to solve the cases recursively and iteratively. All of this is building over the first two days leading towards creating a scheme interpreter in Python. This exercise ties together the first couple of days and is such a treat to see the way all the prior efforts can be composed to build a language. 

## The Second Act: Owning the staging

Next, there is a lovely trip through lambda calculus (don't worry about the math, you got this.) and a peek at the y combinator. There is a discussion of church numerals and functions generating functions. Here is where things can get interesting. Once you can generate functions with other functions, you are at the foundation of this course.  

Building a scheme interpreter in scheme, and here, Dave shines more brilliantly than at any other point in this class. (To be clear, Dave does a fantastic job throughout the course, but he steps it up here.) He began with a simple evaluator function, moved into a beautiful message passing evaluator, and then it was time for lunch. At lunch, he asked us how far we wanted to descend into madness. This class asked him to take us off the deep end and after a coffee Dave 100% delivered. 

## The Third Act: A decent into madness

We began by exploring lazy evaluation in our interpreter and metacircular evaluation. Then Dave took us way off the deep end by exploring Nondeterministic computing and the Amb evaluator. This exploration of Amb was fantastic, first off I've always been fascinated by nondeterministic computing, and Dave explained that he hadn't done this part of the class before. Dave continued to explore this area with us. It was clear he was thinking through the problem, and he spoke out loud as he pondered the solution. Watching Dave chew on something that is still not fully known to him is a rare glimpse into the deep parts of how Dave attacks problems.  For me this was the absolute best part of the class. 

The class ended with us lost in the middle of a problematic part of the evaluator looking at the lambda special form. It wasn't a clean break, and honestly I'm not sure what would have been one with our demand for taking us off the deep end. Dave always delivers, and we awoke the next morning to a message from Dave with a solution to the Amb evaluator problem. Truly the close to the week that Dave deserved. (I also think this reveals a part of Dave's true self. He had to finish the problem. He had to share what he found.)

## Epilogue

I'm always amazed at how Dave runs a class. They are small classes, and he humors each person' questions in the class. He is playfully batting some of them away, giving them space to talk along through some, and diving into the details of others. It takes a true educator to meet people on their terms. We had a mix of people with a wide array of backgrounds, feelings about functional languages,  and in my opinion Dave gave all of us a robust set of take aways and a wonderful experience. 

This class isn't something you immediately take back to work; nonetheless, it is a class that will affect your work. For me, I'm going to be thinking a lot more about different ways I can explore and understand a problem. Additionally, this class solidified some concepts I had been mulling over since I took Dave's compilers class. I always ask myself after each one of the classes I pay for I ask myself "would I do this again?" And for the fourth time after one of Dave's classes, I've had a resounding yes. It's time to save up to take the rafting trip class.
