---
layout: post
title: "Back to First Principles"
tags: [strings]
---

I had my first ever technical interview with Google today. It was a humbling experience.<!--excerpt-->

The problem I had to solve was fairly simple. It was far from complete in its specification but I was prepared for this curveball. I asked my interviewer all the *right* questions which led to exciting discussions on things like developer user experience. Something as seemingly trivial as the return value of a string-examining function can require considerable deliberation on whether to return a pointer versus an index to a character in a string, for example.

After running through the asymptotic analysis of space-time complexity of the algorithm I had designed to solve the problem, my interviewer asked me if there was anything else I wanted to "design" before writing code. I took this opportunity to enumerate test cases covering corner cases and the likes. This led to a number of  interesting tangential discussions. Then we returned back to the "design" question, and I was asked one more time if there was anything else I wanted to explore.

In hindsight, that was the biggest yet most subtle hint that the interviewer had given me. For there was one last feature I had missed to write a test case for and consider in my algorithm design. **Case-insensitive character comparison.**

Feeling confident about my algorithm design and test cases, I moved forward and belted out some solid C++ code that implemented the algorithm. Finishing my work, I plugged in to my algorithm each of my test cases and saw that they all passed. I happily declared my job done.

My interviewer disagreed. That was a good enough hint for me to re-evaluate the correctness of my algorithm's cornerstone piece — a character comparison routine. I immediately recognized my mistake — **my comparison was not case-insensitive!** My interviewer sighed in relief and asked me to fix it.

To my horror, I blanked out. *How on Earth do you compare two characters such that they evaluate as alphabetically equal, i.e. regardless of their case?*

I recalled that I had been using case-insensitive regular expressions all my life (`/[a-zA-Z]/i`). Now, I knew that it would be stupid to compare each pair of characters using such a regular expression but I thought I would throw it out there anyway. My interviewer promptly agreed with my analysis and encouraged me to explore alternative approaches.

The next thing I remembered is that I had always performed such comparisons by comparing a pair of strings after *converting them both to either upper or lower case*. Consider this piece of JS code:
``` javascript
if (str1.toLowerCase() == str2.toLowerCase()) {
	// do something
}
```
So I proposed to compare each pair of characters in my algorithm using this approach. I would convert them to upper case and then compare them. My interviewer received my idea well and asked me to implement a `toUpper` function.

I blanked out again. In all of my 10 years of writing code, from way back during my naive undergraduate days until my present day-job, *I had never bothered to think about how my trusty `toLowerCase` function is actually implemented*. Or maybe I had thought about this at some point but I clearly had forgotten the details.

I started by reasoning that assuming these are ASCII characters (which I was graciously granted), I would have to know the offset that separates the 26 lower case alphabet characters from their upper case counterparts.

"You can't rely on specifically knowing this offset". *True*; not if I was wanting to write portable code, anyway!

The remaining 15 minutes or so (it felt like an eternity) of my interview comprised of me trying to figure out under tremendous pressure how to convert a ruddy character to upper case with the constraint of being disallowed from making assumptions on the specifities of ASCII encoding except that a) the 26 lower case and 26 upper case alphabets are contiguous but are not necessarily aligned back-to-back; and b) I was working with a classic unsigned 8-bit integer representation of characters.

I am sad to say that I never ended up figuring it out during the interview despite all the hints I was given. After we ran out of time, my interviewer showed me the solution. I was still experiencing intense pressure and so didn't really understand the code but I got the gist of it. After the interview concluded and we hung up on the phone, I decided to complete my solution based on my vague recollection of the final solution giveaway.

The redeeming news is that I finally understood it. The idea is that while you can't rely on a hard-coded offset if you want your function to be portable, you *can* defer this encoding knowledge to the compiler. For example, the encoding `65` may or may not refer to the upper case alphabet 'A'; however the character literal `'A'` can be used as a number without the programmer needing to know what it actually is encoded by, except that it is some value between 0 and 255 inclusive. We can then do things like `'z' - 'a'` in order to determine different kinds of offsets and use this knowledge to our advantage.

Keeping this in mind along with the contiguous arrangement of the characters `'A'` to `'Z'`, the solution is roughly as follows.

``` c++
char toUpper (char const c) {
	if (c >= 'A' && c <= 'Z') {
		// the character is already upper case, so return it as-is
		return c;
	} else if (c >= 'a' && c <= 'z') {
		// we've got a lower case character, so jump to the equivalent
		// upper case character by computing the offset between lower
		// case and upper case characters in the encoding scheme
		return c + 'A' - 'a';
	} else {
		// not an alphabet character; return as-is
		return c;
	}
}

// Can be shortened to:
return c + (c >= 'a' && c <= 'z' ? 'A' - 'a' : 0);
```

## Retrospective

I thought *hard* in retrospect about why I struggled with such a fundamental coding problem. What is the key insight here? I recognized it as a two-part phenomenon.
1. *Mystery*. I just had never encountered a problem of this *class* before. I am able to identify graph traversal problems in disguise and implement them relatively at ease. On the other hand, I couldn't perform the trivial task of converting an ASCII character to upper case in a portable manner because I hadn't ever bothered to stop and think about it so far in my CS journey.
2. *Incompetence*. I was not equipped with the tools I needed to derive the upper case conversion algorithm myself from first principles. Returning to the graph comparison, I have developed by now an intuition for the structure of graphs and so do not need to memorize breadth-first and depth-first traversal algorithms because I can derive them myself. Yet for some reason, I was lost in the darkness of character encoding schemes groping around hopelessly for some salvific insight that would help me solve the problem and impress my interviewer.

I view my ordeal today as a wake-up call for me to regain a solid grasp on the fundamentals; ***first principles***. During my interview preparation, I worked my royal behind off studying all the hard problems in CS: graph theory, recurrences, backtracking, memoization, search trees, sorting. But I didn't bother to go over the bread and butter of all digital information: *strings*!

"But why should I care about knowing the innards of case-insensitive string comparison? Who on Earth in their right mind stops at the `toLowerCase()` function and ponders the implementation thereof? There are way more pressing and interesting problems out there to be solved!"

I'm not wrong. *Unless I want to push the boundaries of human knowledge*. Thinktanks like Google are in the business of pushing the frontiers of technology. How could they trust me to join them in transforming the fundamentals of the field if I am not solidly grounded in these extant truths myself?

> What right does a man have to talk authoritatively about matters of which he is ignorant?
>
> <cite>Paraphrase, Anonymous.</cite>

My Lord God has spoken clearly to me. The Holy Spirit is inspiring me to take many steps back in my Calling before I can move forward...to go back to humble beginnings and start again from *first principles*.
