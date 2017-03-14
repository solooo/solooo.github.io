---
title: Euler 3 Largest prime factor
date: 2017-03-14 13:41:09
categories:
tags:

- Euler
---
### Problem 3: Largest prime factor

    The prime factors of 13195 are 5, 7, 13 and 29.

    What is the largest prime factor of the number 600851475143 ?

<!-- more -->
### Answer:

java code:
```java
public class PrimeFactor {
    private static Long maxPrimeFactor(Long n) {
        if (n < 2) {
            return n;
        }
        Long i = 2L;
        while (n > i) {

            if (n%i == 0) {
                n = n/i;
            }
            i++;
        }
        return i;
    }

    public static void main(String[] args) {
        Long n = 6008514751430L;
        System.out.println(maxPrimeFactor(n));
    }
}
```
