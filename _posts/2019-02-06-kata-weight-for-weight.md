---
layout: post
title: Weight for Weight
excerpt: "Codewars Kata Weight for Weight: 5kyu"
categories: [kata]
tags: [5kyu, c++]
kata_url: https://www.codewars.com/kata/weight-for-weight
use_math: true
---

<!-- TODO: Insert kata series preamble include -->

## Brief

> My friend John and I are members of the "Fat to Fit Club (FFC)". John is worried because each month a list with the weights of members is published and each month he is the last on the list which means he is the heaviest.
>
> I am the one who establishes the list so I told him: "Don't worry any more, I will modify the order of the list". It was decided to attribute a "weight" to numbers. The weight of a number will be from now on the sum of its digits.
>
> For example `99` will have "weight" `18`, `100` will have "weight" `1` so in the list `100` will come before `99`. Given a string with the weights of FFC members in normal order can you give this string ordered by "weights" of these numbers?
>
> -- <cite>[Source, complete description]({{page.kata_url}})</cite>

## Analysis

We have here a complex sorting problem. Given a string of whitespace-delimited numbers (the "weights"), we need to output a stringified list of these numbers that is sorted in ascending order using the following criteria:
1. Sum of the individual digits of a weight, in ascending order;
2. If the sum of two weights is equal, then use the weight number as a string, in lexicographical order.

Some stipulations and assumptions:
1. All numbers in the list are positive numbers.
2. The list can be empty.
3. The input string can have any number of leading, trailing and delimiting whitespace characters. (All kinds of whitespace characters are possible; not just 'space' `' '`).
4. The input parameters to function `orderWeight` should not be modified.
5. Though not specified in the problem description, we will safely assume that the numbers won't be bigger than `INT_MAX` given the domain of the problem (human body weights).

## Strategy

We start by splitting the input string into a `std::vector` of tokens that contain just the numbers. **Regular expressions** will serve this purpose quite well.

It is then just a matter of writing a comparison function/functor (which fulfills the STL [`Compare`](https://en.cppreference.com/w/cpp/named_req/Compare) concept) that will compare a pair of numbers *first* based on the sums of their individual digits and *then* by lexicographical order of the stringified numbers *iff* the digit sums are equal.

We can then use `std::sort` with the above comparison object on the `std::vector` of tokens. Finally, we stringify the sorted `std::vector` into a string and return this value.

### Compute the Sum of the Digits of a Positive Integer

The functional but naive approach to calculating the sum of the digits of a number is:
1. Stringify the number;
2. Convert each character of this string into a number (perhaps using `std::stringstream`), yielding the individual digits;
3. Add up all of the digits.

**Boring!** Why don't we have some number-theoretic fun instead?

It is actually quite simple. Divide the number by 10. The remainder is the least-significant digit (LSD). Divide the quotient by 10. The remainder is LSD + 1. Divide the quotient by 10...etc. until we obtain a quotient of `0`, meaning that we have extracted all the digits, then add up all of them. We will take care of this using integral division and the modulo `%` operator.

#### Explanation

*I was initially stumped on this part as I had forgotten the algorithm for digit extraction from my undergrad days. Fortunately, I was able to refresh my memory thanks to [some](https://www.electronics-tutorials.ws/binary/bin_2.html) [help](https://stackoverflow.com/questions/3614103/how-to-extract-each-digit-from-a-number) from the Internet.*

A non-negative integer $$N$$ can be expressed as the sum of the place values of each of its digits $$d$$. Generally, given a base $$b$$,
$$
N_b = (d_i \times b^i) + (d_{i-1} \times b^{i-1}) + ... + (d_0 \times b^0)
$$

where $$i$$ corresponds to the [index of the respective digit](https://en.wikipedia.org/wiki/Positional_notation).

For example, we can express the decimal number `456` as:
$$
456_{10} = (4 \times 10^2) + (5 \times 10^1) + (6 \times 10^0)
$$

Now how does continually dividing the number by 10 give us its digits? To be frank, I do not understand the proper mathematics behind this phenomenon. Nonetheless, we can intuit the mechanics via an example.

1. Divide `456` by `10`: Quotient = `45`; Remainder = `6`. $$456 \div 10 = (45 \times 10) + 6$$
2. Divide `45` by `10`: Quotient = `4`; Remainder = `5`. $$45 \div 10 = (4 \times 10) + 5$$
3. Divide `4` by `10`: Quotient = `0`; Remainder = `4`. $$4 \div 10 = (0 \times 10) + 4$$

The result is the array of digits starting from the least-significant digit (LSD) to the most-significant digit (MSD). Observe that arranging this array in reverse order gives us the original number.

**In general, digits of a number $$N$$ in arbitrary base $$b$$ can be extracted by continually dividing by $$b$$, obtaining the quotient and the remainder, then using the quotient as the next dividend until we obtain a quotient of 0. The remainder of each division corresponds to the individual digits of the number from the LSD to the MSD.**

### Output

Easy: stringify the final list using a single space `' '` character as the delimiter.

### Edge Cases

1. Empty list.
  * Simply return the empty string.
2. Singleton list.
  * Return the single number as a string.

## Final Solution

``` c++
#include <cstdlib>
#include <iostream>

#include <vector>
#include <regex>
#include <sstream>
#include <algorithm>

typedef unsigned long long num_t; // handle huge numbers

num_t sumOfDigits (num_t const n, num_t const base=10)
{
  num_t remainder = n % base;
  
  // Base case: we have reached the final digit since the quotient `n` is 0
  if (n == 0)
  {
    return remainder;
  }

  return remainder + sumOfDigits(n / base, base);
}

class WeightSort
{  
  static bool compare (num_t const left, num_t const right)
  {
    num_t sum_left = sumOfDigits(left);
    num_t sum_right = sumOfDigits(right);
    
    if (sum_left == sum_right)
    {
      std::stringstream ss_left;
      ss_left << left;
      std::stringstream ss_right;
      ss_right << right;
      return ss_left.str() < ss_right.str(); // lexicographical order
    }
    else
    {
      return sum_left < sum_right;
    }    
  }

public:
  static std::string orderWeight(const std::string &str)
  {
    if (str.empty())
    {
      return "";
    }
    
    // Split into tokens delimited by any combination of whitespace
    std::vector<num_t> numbers;
    {
      std::regex re("\\d+");
      std::sregex_iterator next(str.begin(), str.end(), re);
      std::sregex_iterator end;
      for (; next != end; ++next)
      {
        numbers.push_back(strtoll(next->str().c_str(), NULL, 10));
      }
    }        
    
    // Sort according to problem criteria
    std::sort(numbers.begin(), numbers.end(), compare);
    
    // Output as a string
    size_t n = numbers.size();
    std::stringstream result;
    for (num_t i = 0; i < n; ++i)
    {
      result << numbers[i];
      if (i != n - 1 )
      {
        result << " ";
      }
    }
    return result.str();
  }
};
```

<cite>[Check it out @ CodeWars](https://www.codewars.com/kata/reviews/57872ea6fdd4261d260000d2/groups/5c5aaf1c6a8a380001e73e8e)</cite>
