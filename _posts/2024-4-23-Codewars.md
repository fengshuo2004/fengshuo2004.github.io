---
layout: post
title: Codewars C语言算法题解集锦
---

本文集合了我在 [Codewars](https://www.codewars.com/dashboard) 上刷到的题目（Kata）以及我自己的C语言解法。

# Find The Parity Outlier

## 描述

You are given an array (which will have a length of at least 3, but could be very large) containing integers. The array is either entirely comprised of odd integers or entirely comprised of even integers except for a single integer `N`. Write a method that takes the array as an argument and returns this "outlier" `N`.

### Examples

```
[2, 4, 0, 100, 4, 11, 2602, 36] -->  11 (the only odd number)

[160, 3, 1719, 19, 11, 13, -21] --> 160 (the only even number)
```

[原题链接](https://www.codewars.com/kata/5526fc09a1bbd946250002dc)

## 解

```c
#include <stddef.h>

int find_outlier(const int values[/* count */], size_t count)
{
    int i;
    if ((values[0] & 1) ^ (values[1] & 1)) { // are values[0] and values[1] of different parity?
        // YES: look at the values[2], determine outlier to be values[0] or values[1]
        i = (values[1] & 1) ^ (values[2] & 1);
        return values[i];
    } else {
        // NO: the outlier is in the rest of the array
        for (i = 2; i < count; ++i){
            if ((values[0] & 1) ^ (values[i] & 1)){ // compare each against values[0]
                return values[i];
            }
        }
    }
}
```

# Convert string to camel case

## 描述

Complete the method/function so that it converts dash/underscore delimited words into camel casing. The first word within the output should be capitalized **only** if the original word was capitalized (known as Upper Camel Case, also often referred to as Pascal case). The next words should be always capitalized.

### Examples

`"the-stealth-warrior"` gets converted to `"theStealthWarrior"`

`"The_Stealth_Warrior"` gets converted to `"TheStealthWarrior"`

`"The_Stealth-Warrior"` gets converted to `"TheStealthWarrior"`

[原题链接](https://www.codewars.com/kata/517abf86da9663f1d2000003)

## 解

```c
//  do not allocate memory for the result
//  write to pre-allocated pointer *camel

void copyChar(char* src, char* dest, char cap){
    if (*src == '\0'){ // we've reached end of string?
        *dest = '\0';
        return;
    }
    if (*src == '-' || *src == '_'){
        src++;
        copyChar(src, dest, 1); // copy next char, set Capitalize flag
        return;
    }
    if (cap && *src >= 'a' && *src <= 'z') {
        *dest = *src - 32; // 'a' -> 'A' etc.
    } else {
        *dest = *src; // copy as-is
    }
    src++;
    dest++;
    copyChar(src, dest, 0); // copy next char
    return;
}

void to_camel_case(const char *text, char *camel) {
    copyChar(text, camel, 0);
}
```

# Multiples of 3 or 5

## 描述

If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.

Finish the solution so that it returns the sum of all the multiples of 3 or 5 **below** the number passed in.

Additionally, if the number is negative, return 0.

**Note:** If the number is a multiple of **both** 3 and 5, only count it *once*.

*Courtesy of projecteuler.net (Problem 1)*

[原题链接](https://www.codewars.com/kata/514b92a657cdc65150000006)

## 解

```c
int solution(int number) {

    int temp = number - 1; // not to include number itself
    long maxDiv3 = temp / 3;
    long maxDiv5 = temp / 5;
    long maxDiv15 = temp / 15;
    long sum;
    // sum multiple-of-3s: 3 * (1 + 2 + 3 + ... + maxDiv3)
    sum = 3 * (1 + maxDiv3) * maxDiv3 / 2;
    // sum multiple-of-5s
    sum += 5 * (1 + maxDiv5) * maxDiv5 / 2;
    // de-dupe multiple-of-15s
    sum -= 15 * (1 + maxDiv15) * maxDiv15 / 2;

    return sum;

}
```

# Take a Ten Minutes Walk

## 描述

You live in the city of Cartesia where all roads are laid out in a perfect grid. You arrived ten minutes too early to an appointment, so you decided to take the opportunity to go for a short walk. The city provides its citizens with a Walk Generating App on their phones -- everytime you press the button it sends you an array of one-letter strings representing directions to walk (eg. ['n', 's', 'w', 'e']). You always walk only a single block for each letter (direction) and you know it takes you one minute to traverse one city block, so create a function that will return true if the walk the app gives you will take you exactly ten minutes (you don't want to be early or late!) and will, of course, return you to your starting point. Return false otherwise.

> Note: you will always receive a valid array containing a random assortment of direction letters ('n', 's', 'e', or 'w' only). It will never give you an empty array (that's not a walk, that's standing still!).

[原题链接](https://www.codewars.com/kata/54da539698b8a2ad76000228)

## 解

```c
#include <stdbool.h>

// input is a null-terminated string

bool isValidWalk(const char *walk) {

    int i;
    // we start at coordinate 0,0
    int x = 0;
    int y = 0;
    // iterate the first 10 moves
    for (i = 0; i < 10; ++i){
        switch (walk[i]){
            case 'n': ++y; break;
            case 's': --y; break;
            case 'w': --x; break;
            case 'e': ++x; break;
            case '\0': return false; // walk ended early
        }
    }
    // return whether walk ended at 0,0 AND at 10 mins
    return (x == 0 && y == 0 && walk[i] == '\0');

}
```
