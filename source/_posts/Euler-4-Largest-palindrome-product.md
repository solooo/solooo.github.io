---
title: Euler-4-Largest-palindrome-product

date: 2017-03-17 13:46:01

categories:
- euler
tags:
- Euler
---
### Problem 4: Largest palindrome product

    A palindromic number reads the same both ways. The largest palindrome made from the product of two 2-digit numbers is 9009 = 91 × 99.

    Find the largest palindrome made from the product of two 3-digit numbers.

<!-- more -->
### Answer:

java code:
```java
public class Palindrome {

    /*
    Largest palindrome product
    Problem 4
    A palindromic number reads the same both ways. The largest palindrome made from the product of two 2-digit numbers is 9009 = 91 × 99.

    Find the largest palindrome made from the product of two 3-digit numbers.
     */

    public static void main(String[] args) {
        long time = System.currentTimeMillis();
        int n = 3;
        int a = (int) (Math.pow(10, n) -1);
        int max = 0;
        for (int i = a; i > 0; i--) {
            for (int j = a; j > 0; j--) {
                int product = i * j;
                String s = String.valueOf(product);
                String[] split = s.split("");
                String str = "";
                for (int m = split.length - 1; m >= 0; m--) {
                    str += split[m];
                }
                if (s.equals(str) && Integer.parseInt(s) > max) {
                    max = Integer.parseInt(s);
                }
            }
        }
        System.out.printf("Max Palindrome %d, cost %d ms", max, System.currentTimeMillis() - time);
    }
}

```