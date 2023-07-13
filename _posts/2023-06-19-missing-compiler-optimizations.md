---
layout: post
title:  "Automatically identifying excessive assembly bloat"
date:   2023-06-19 21:45:42 +0100
categories: compilers
---
<span style="color:darkgray">This project was performed in the context of the Automated Software Testing lecture held by Prof. Zhendong Su at ETH Zurich. I worked on it together with Marco Heiniger, under the supervision of Yann Girsberger.</span>

Being aware of the effects that different compiler flags have on the generated assembly from C code is crucial to 

{% highlight asm %}
main:
    cmp DWORD PTR i[rip], 6
jg .L2
    mov DWORD PTR i[rip], 7
.L2:
    xor eax, eax
    ret
{% endhighlight %}

The issue:

How we identify it

What we found

How can someone also try doing this