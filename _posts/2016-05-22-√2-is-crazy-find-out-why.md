---
layout: post
title: √2 is crazy find out why
categories: musings, Maths
tags:
- musings
- Maths
---

Back in my school days, my mathematics professor always used to praise maths as **_Natural Science_**. It is indeed. However during school / college days, I used to wonder why the hell I’m learning the differentiation / integration / ordinary differential equations / Fourier transforms bla bla bla.
However, after spending 7 odd years in IT career and filling by mind with arrays, structures, sockets, RPCs, observer design patters, memory leaks, core dumps, GDB, conference calls, proposals, delivery **_Huhhhhh_**  😦 I wanted to go back to my 10 std and immerse myself in _trigonometry_ & _algebra_.

My friends used to mock me being a nerd as I always use to tell speed of the light is not $$ 3 \times 10^8 m/sec $$ , it is $$ 2.99792458 \times 10^6 m/sec $$ .  :) [ Now I know, why those people called me a nerd  :) ]

Mathematics & Physics were my most sought-after subject. It still intrigues me when I see movies like A beautiful mind Apollo 13 Inter-Stellar Martian etc. Not just for sci-fi scenes ( could anyone forget Back to the future series wow… ), but for the subtle physical & mathematical facts that is buried deep inside story and dialogues.

Of all the mathematical figures, $$ \sqrt{2} $$ is my favorite number. It is because, it a number, however it is categorized as *irrational*.

> Irrational means, not being rational. i.e going [bonkers](http://www.thesaurus.com/browse/going%20bonkers)

Well let me explain how.

Any rational number can be explained in fractional form. So let us say  $$ \sqrt{2} $$ can be expressed as by 2 numbers $$ p $$ & $$ q $$, such that, $$ \frac {p}{q} = \sqrt{2} $$

But we should also remember that, P or Q should be ODD number. Because, if both P & Q are even number, then should get cancelled until, they become odd. Just think about it, $$ \frac {128}{48} $$ will eventually become $$ \frac {8}{3} $$ . So our first _premise_ is this.

> **_$$ \sqrt{2} $$ is a rational number expressed by $$ \frac {p}{q} = \sqrt{2} $$, such that P or Q are odd numbers._**


Now let us try to prove our premise and try to add support for our argument. 

$$
\begin{align}
\sqrt{2} & = \frac{p}{q} \\
& = \frac{2n}{q}  \text{p = 2n} \\ 
& = 2 = \frac{4n^2}{q^2}  \text{squaring on both sides} \\
& = 2q^2 = 4n^2  \text{re-arranging} \\
& = q^2 = 2n^2   \text{Taking out the common factor}
& = \approx q^2 \approx \text{q is even}
\end{align}
$$


_Eq-1_ <center> $$ \frac {p}{q} = \sqrt{2} $$ </center> [ premise ]

_Eq-2_ <center> $$ \frac {p^2}{q^2} = 2 $$    </center> [ squaring on both sides ]

_Eq-3_ <center> $$ p^2 = 2q^2 $$           </center> [ re-arranging ]

Since anything multiplied by $$ 2 $$ is _even number_, we can term that, $$ p^2 $$ is _even_ . If $$ p^2 $$ is even, then even $$ p $$ is even, since square of an odd number is odd and square of even number is even.

Okay, so now we had deduced $$ p $$ is even, there can be only one thing possible to prove the above premise. i.e $$ q $$ is _Odd_.
Let us try to find out, whether $$ q $$ is odd or not.

**Eq-4** $$ \frac {2n}{q} = \sqrt{2} $$  [ Given p is even, any even number p can be represented as $$ p = 2n $$, where n being odd or even ] 
