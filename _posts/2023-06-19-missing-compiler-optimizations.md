---
layout: post
title:  "Automatically identifying excessive assembly bloat"
date:   2023-06-19 21:45:42 +0100
categories: compilers
---
<span style="color:darkgray">This project was performed in the context of the Automated Software Testing lecture held by Prof. Zhendong Su at ETH Zurich. I worked on it together with Marco Heiniger, under the supervision of Yann Girsberger.</span>

Compiler flags allow programmers to optimize generated binaries for their specific use-case. Generating fast, easy to debug or short assembly files can be accomplished with flags such as `-O3`, `-Og`, `-Os`. In this project we have focused on finding minimal instances of C code that, while using the space-optimizing flag `-Os` result in larger binary size than with the use of different compilers or compilation flags.

This blog post will give a short outline of the process used to find these bloated instances. A detailed analysis as well as source code can be found on our [Gitlab page][ast-gitlab]
### Assembly bloat example

How GCC bloats an empty printf:  
<pre><code class="c">
#include &lt;stdio.h&gt;
int a;
int main() { printf("", a); }
</code></pre>


GCC 13.1 [-Os]
<pre><code class="x86asm">
.LC0:
    .string ""
main:
    push    rax
    mov     esi, DWORD PTR a[rip]
    mov     edi, OFFSET FLAT:.LC0
    xor     eax, eax
    call    printf
    xor     eax, eax
    pop     rdx
    ret
a:
    .zero   4
</code></pre>

Clang 16.0.0 -Os
<pre><code class="x86asm">
main:
    xor     eax, eax
    ret
a:
    .long   0

</code></pre>

More examples, as well as a short analysis can be found in our [artifacts branch][arti-branch].

### The method
We have conducted our analysis using [Diopter][diopter], a python wrapper around the following tools:
1. `csmith`: This tool has been used to automatically generate C code for our analysis. By passing different flags we can specify the kinds of structures that the generated code should contain (such as arrays, strings, ...). 
2. `C-Reduce`: This is the main tool used in our analysis. It performs the process of delta debugging. More specifically it takes as inputs C source code as well as a True/False test function. Creduce then attempts to remove lines from the initial C code, while ensuring that the test function always returns True. This test funciton is an "interestingness" test. In general it often checks if a specific bug gets triggered, while in our instance it ensures that the size of the assembly generated using -Os is larger than that generated via other flags such as -O3 or with other compilers (CLANG).
3. `c sanitization checker`: We use this after each creduce step, to ensure that we have generated valid C code.


### Example of a test function

Following is an example of one of the test functions used. We noticed that one of the best metrics to quantify the amount of remaining C code was obtained by counting the number of nodes in its Abstract Syntax Tree, instead of the number of characters or lines left in the C file.
<pre><code class="python">
# Compute the ratio using AST to quantify number of lines
def test_4(self, program: SourceProgram) -> bool:
    root = ast_parser.get_ast_tree(program.code)
    ratio = helper.get_tree_ratio(program,self.Os,root)

    return ratio > self.bestRatio
</code></pre>

This test function ensures that the ratio between -O3 and -Os increases in each succesful C-Reduce iteration.


[ast-gitlab]: https://gitlab.ethz.ch/tibaldoc/ast_largebinaries_smallcode
[arti-branch]: https://gitlab.ethz.ch/tibaldoc/ast_largebinaries_smallcode/-/tree/artifacts/
[diopter]: https://github.com/DeadCodeProductions/diopter/tree/main