---
title: Pretty Printing Interactive List Haskell
author: arpiku 
date: 2024-08-05 20:45:00 +0530
categories: [Programming, Haskell , Funtional Programming]
tags: [Haskell, Functional Programming]
pin: false 
---


The standard Haskell installation with ghci actually brings a lot of goodies alongside, one of these
libraries in ANSI console library, which provides tools to sylize the terminal output. Following are a few simple example.

[img1] (/assets/ar1/img1.png)

Let's start with some simple code that would let us provide a set of settings and a string to
print to the terminal

```haskell
prettyPrint sgrs str = setSGR sgrs >> putStr str >> setSGR [Reset] >> putStrLn ""
```

[img2] (/assets/ar1/img2.png)

This works for a single string, but what about a list of strings?

Let's first write a function that will give us some formats to map over string, depending on the
length of the string

```haskell
getSGRList n
            | n <= 0 = []
            | n == 1 = [selected, alt1]
            | otherwise = selected : concat (replicate (n `div` 2) [alt1, alt2])
```

and then we can simple write a function as follows

```haskell
printSelectableList sgrs xs = zipWithM_ prettyPrint sgrs xs
```
where sgrs can be generated using getSGRList by just passing the length of the string.
Combining the above ideas we get something like this

[img3] (/assets/ar1/img3.png)

We can use and modify the code from `https://stackoverflow.com/questions/23068218/haskell-read-raw-keyboard-input/38553473#38553473`
to handle inputs alongside adding two funtions to rotate the currently selected row

```haskell
getKey :: IO [Char]
getKey = reverse <$> getKey' ""
  where getKey' chars = do
          char <- getChar
          more <- hReady stdin
          (if more then getKey' else return) (char:chars)


rotateUp [x] = [x]
rotateUp (x:xs) = xs ++ [x]


rotateDown [x] = [x]
rotateDown xs = [last xs] ++ init xs

menuController (sgrs, strs) = do
  clearScreen
  hSetBuffering stdin NoBuffering
  hSetEcho stdin False
  printSelectableList sgrs strs
  key <- getKey
  when (key /= "\ESC") $ do
    case key of
      "\ESC[A" -> menuController (rotateUp sgrs, strs)
      "\ESC[B" -> menuController (rotateDown sgrs, strs)
      _        -> return ()
    menuController (sgrs, strs)
```

[img4] (/assets/ar1/img4.png)
[img5] (/assets/ar1/img5.png)

Given this, we can modify the above code to result in particular behaviour, like returning the
number of the currently 'green' row and using that information to perform some other task, hence
we got ourselves a rather simple interactive interface.
