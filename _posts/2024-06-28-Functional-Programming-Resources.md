---
title: Awesome Functional Programming Resources
author: arpiku 
date: 2024-06-28 20:45:00 +0530
categories: [Programming, Haskell , Funtional Programming, Category Theory]
tags: [Category Theory, Haskell, Functional Programming]
pin: false 
---



# Awesome Functional Programming Stuff

So somewhere along the line I started using nixOS that resulted me having a serious look into functional programming and before I knew it
I was deep into reading research papers on type theory 😮‍💨.  Welp! here are some the cool corners of internet I have found on the journey that can
help you learn functional programming too or at least help you get lost in theory and mathematics of it all. Enjoy!

## Lambda Calculus

https://en.wikipedia.org/wiki/Lambda_calculus
This wikipedia pages introduction is good enough, it might seem a bit confusing at first, therefore below are some links to practices exercies to understand
the concept a bit better.

- https://www.cs.umd.edu/class/fall2017/cmsc330/tests/prac8-soln-fall13.pdf
- https://www.cs.rochester.edu/u/brown/173/exercises/functional/lambda/lambda_ex1.pdf

I would also suggest to give the Wikipedia page for combinatorial logic a glance as having these mental contructs can help later on in understanding 
a lot of jargon that may be thrown around while discussing functional programming.

https://en.wikipedia.org/wiki/Combinatory_logic


## Category Theory

- https://www.youtube.com/watch?v=I8LbkfSSR58&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_
This is an awesome lecture series on the topic, there are 2 more levels availble on Mr.Bartosz Milewski channel, but I defenetly would suggest that go
through the first part atleast, the other two basically continue the ideas introduced into the first part to higher levels though if you are trying to
code in Haskell the first series should give you enough of a primer to get more intutive feeling around types and how to use them.

- https://ncatlab.org/nlab/show/HomePage
A rather general site that covers many areas from mathematics and philosophy and such, they have a wide collection of articles on topics of Category theory, even if I hadn't mentioned this website here, you probably would come across this website multiple times trying to understand category theory and
Haskell.

Following are wikipedia pages on Monad, Monoid, Product and Sum types in Category theory, I found myself coming back to them to see the diagram chases to
develop a better understanding around Category theory.
- Sum Type (Co-Product) - https://en.wikipedia.org/wiki/Coproduct
- Product Type - https://en.wikipedia.org/wiki/Product_type
- Monad - https://en.wikipedia.org/wiki/Monad_(category_theory)
- Monoid - https://en.wikipedia.org/wiki/Monoid_(category_theory)

## Research Papers
Some classics of functional programming world that you should read, as they are the direct sources of a lot of functional programming ideas, and as it 
is with a lot of things going directly to the source can really result in some quality dense information gathering and learning.

- Generalising Monads to Arrows - https://www.cse.chalmers.se/~rjmh/Papers/arrows.pdf
- The Essence of the Iterator Pattern - https://www.cs.ox.ac.uk/jeremy.gibbons/publications/iterator.pdf
- Fun with type Functions - https://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/typefun.pdf?from=https://research.microsoft.com/~simonpj/papers/assoc-types/fun-with-type-funs/typefun.pdf&type=exact

The following are "Lambda the Ultimate Papers", I found about them by watchin this awesome video - https://www.youtube.com/watch?v=uR_VzYxvbxg&t=2155s
which discusses how simple the core of Haskell actually is.
Anyways here are the papers:

- Scheme: An Interpreter For Extended Lambda Calculus - https://dspace.mit.edu/bitstream/handle/1721.1/5794/AIM-349.pdf
- Lambda : The Ultimate Imperative - https://apps.dtic.mil/sti/pdfs/ADA030751.pdf
- Lambda : The Ultimate Declarative - https://apps.dtic.mil/sti/pdfs/ADA034090.pdf
- Lambda : The Ultimate GOTO - https://apps.dtic.mil/sti/pdfs/ADA062381.pdf
- The Art of the Interpreter or, the Modularity Complex - https://dspace.mit.edu/bitstream/handle/1721.1/6094/AIM-453.pdf
- Rabbit: A compiler for Scheme - https://dspace.mit.edu/bitstream/handle/1721.1/6913/AITR-474.pdf

- Software Transaction Memory - https://www.microsoft.com/en-us/research/wp-content/uploads/2005/01/2005-ppopp-composable.pdf

## Some Awesome Haskell Resources:

- https://learnyouahaskell.com/chapters
This is the first book that I read on Haskell, I cannot appreiciate enough the simplicity with which the author has approached the topic, especially with
all the cute doodles, this is defenitely a book that shows how Haskell is actually a lanugage for kids due to it's inherent simplicity and is infact not
as complicated as people belive it to be.

- https://book.realworldhaskell.org
Another awesome book made freely available by the authors, some really smart code snippets are preseneted here that show the compactness and power of Haskell.


- https://wiki.haskell.org/Category:Idioms
These are various topics related to Haskell, I like them because exploring these pages introduces me to a lot of new theortical concepts and their
direct implementation in Haskell, again some really awesome code snippets are presented that can really help you get a feel of the possiblities with Haskell code.

- https://www.haskell.org/onlinereport/haskell2010/
This is the original documentation of Haskel2010 report, it may seem a lot to include it, but actually the report is actually fairly simple, and the
technical details can easily be understood, infact it reads more like a simple book that anyone can sort of glance through and achieve a rather deep look in Haskell under the hood.


And that's pretty much it, I hopefully will update this in future as there is still some stuff I perusing through.

