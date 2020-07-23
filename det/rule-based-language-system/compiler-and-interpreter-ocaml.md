# Compiler and Interpreter \(OCaml\)

## Introduction

In this mini-project, we fulfilled an earlier anticipation for the module: to build an interpreter and compiler for arithmetic expressions. In what follows we will explain our implementations and share some interesting observations.

## Task 1: Implement the source language interpreter

Implement an interpreter for the source language as an OCaml function that satisfies the specification in the project descriptions.

The first task is to implement an interpreter for the source language. In this process, we make use of the match expressions to implement the given specifications of the interpreter. Note that our implementation mirrors the specification \(almost exactly\) and each of the induction steps have the same structure. Concretely, we evaluate the input expression `e` by matching it with the following cases:

* **Base case**: for any integer n \(noting its syntactic representation as n\), evaluating Literal n yields the integer n. And our implementation is:

  ```ocaml
  | Literal n -> Expressible_int n
  ```

* **Induction step for additions:** for any arithmetic expression e1 that was evaluated as the expressible value ev1 \(which is the first induction hypothesis\) and any arithmetic expression e2 that was evaluated as the expressible value ev2 \(which is the second induction hypothesis\). For this our implementation is:

  ```ocaml
  | Plus (e1, e2) ->
     (match evaluate e1 with
      | Expressible_int n1 ->
         (match evaluate e2 with
          | Expressible_int n2 -> Expressible_int (n1 + n2)
          | Expressible_msg s -> Expressible_msg s)
      | Expressible_msg s -> Expressible_msg s)
  ```

* **Induction step for subtractions:** mirroring the structure for the induction step for additions, we have

  ```ocaml
  | Minus (e1, e2) ->
     (match evaluate e1 with
      | Expressible_int n1 ->
         (match evaluate e2 with
          | Expressible_int n2 -> Expressible_int (n1 - n2)
          | Expressible_msg s -> Expressible_msg s)
      | Expressible_msg s -> Expressible_msg s)
  ```

* **Induction step for quotients:** the implementation, again, mirrors the structure for additions and subtractions, except an additional step to determine if there is an error of division by zero \(if there is, we will generate an error message; otherwise we will proceed with the usual computation\).

```text
| Quotient (e1, e2) ->
       (match evaluate e1 with
        | Expressible_int n1 ->
           (match evaluate e2 with
            | Expressible_int n2 ->
               if n2 = 0 then
                 (Expressible_msg ("quotient of " ^ string_of_int n1 ^ " over 0"))
               else Expressible_int (n1 / n2)
            | Expressible_msg s -> Expressible_msg s)
        | Expressible_msg s -> Expressible_msg s)
```

* **Induction step for remainders:** this induction step mirrors that of the quotient, in which we will generate the error message of division by zero.

Finally, we verified that our function passed the unit-test function:

**Subsidiary questions:**

1. Does your interpreter evaluate from left to right or from right to left?

   We experimented with Ocaml and discovered that our interpreter evaluates from left to right:

   In the above we need to make a distinction between the evaluation order for arithmetic operations in OCaml \(which is from right to left\) and the evaluation order of our interpreter \(which is from left to right\). This is made clear by the second example, where the message is printed for the first instance of division by zero, which means it evaluates the expression on the left `(Quotient(Literal (an_int 10), Literal (an\_int 0)))` before the expression on the right `Quotient(Literal (an_int 100), Literal (an_int 0))`.

2. Does it make any observable difference whether your interpreter evaluates from left to right or from right to left?

   As illustrated by the first example, when the expression is pure \(i.e., does not produce any side effects\), the evaluation order of the interpreter does not make any observable difference, for instance, the case where the expression evaluates to an output of the type `Expressible_int`. As illustrated by the second example, when the expression is impure \(i.e., produces any side effects\), the evaluation order of the interpreter will make observable difference, for instance, the case where the expression evaluates to an output of the type `Expressible_msg`.

## Task 2: Implement a one byte-code instruction processor

Implement a processor for one byte-code instruction as an OCaml function that satisfies the specification in the project descriptions.

In this task, we implemented a function that can process the byte-code instructions. Our problem-solving approach was the same as the previous exercise: follow the instructions, that is, the structure of our implementation mirrored that of the given specifications. The function was specified by cases, following the structure of byte-code instructions:

* **Push n**, where n represents the integer n: for any data stack ds \(represented as ds\), processing the byte-code instruction Push n consists in pushing n on top of ds and returning the resulting stack, i.e., n :: ds. Our implementation is as follows:

  ocaml \| Push n -&gt; OK \( n:: ds\)

* **Add.** The implementation for this case follows from the byte-code instructions for addition.
* **Sub.** The implementation for this case follows from the byte-code instructions for subtraction.
* **Quo.** The implementation for this case follows from the byte-code instructions for quotient.
* **Rem.** The implementation for this case follows from the byte-code instructions for remainder.

Finally, we verified that our function passed the unit-test function:

## Task 3: Implement the target language interpreter

Implement an interpreter for the target language \(i.e., a virtual machine\) as an OCaml function that satisfies the specification just above.

\(Solution is given for this task.\)

## Task 4: Implement a compiler

Implement a compiler from the source language to the target language as an OCaml function that satisfies the specification in the project descriptions.

In this exercise, we used list concatenation to implement each case in the function.

* **Base case:** for any integer n \(noting its syntactic representation as n\), translating Literal n yields the singleton list containing the byte-code instruction**Push n**. Following this specification, our implementation is as follows:

  ocaml \| Literal n -&gt; \[Push n\]

* **Induction step for additions:** for any arithmetic expression e1 \(represented as e1\) that was translated as the list of byte-code instructions bcis1 \(which is the first induction hypothesis\) and any arithmetic expression e2 \(represented as e2\) that was translated as the list of byte-code instructions bcis2 \(which is the second induction hypothesis\), translating Plus \(e1, e2\) yields the list of byte-code instructions that begins with the instructions in bcis1, continues with the instructions in bcis2, and ends with the **Add** instruction.

  In our implementation, we simply append the translated expressions to the list, with the **add** instruction. Concretely,

  ocaml \| Plus \(e1, e2\) -&gt; List.append \(List.append \(translate e1\) \(translate e2\) \[Add\]

* **Induction step for subtractions:** for any arithmetic expression e1 \(represented as e1\) that was translated as the list of byte-code instructions bcis1 \(which is the first induction hypothesis\) and any arithmetic expression e2 \(represented as e2\) that was translated as the list of byte-code instructions bcis2 \(which is the second induction hypothesis\), translating Minus \(e1, e2\) yields the list of byte-code instructions that begins with the instructions in bcis1, continues with the instructions in bcis2, and ends with the **Sub** instruction. Our implementation reflected this specification:

  ocaml \| Minus \(e1, e2\) -&gt; List.append \(List.append \(translate e1\) \(translate e2\)\) \[Sub\]

* **Induction step for quotients:** for any arithmetic expression e1 \(represented as e1\) that was translated as the list of byte-code instructions bcis1 \(which is the first induction hypothesis\) and any arithmetic expression e2 \(represented as e2\) that was translated as the list of byte-code instructions bcis2 \(which is the second induction hypothesis\), translating Quotient \(e1, e2\) yields the list of byte-code instructions that begins with the instructions in bcis1, continues with the instructions in bcis2, and ends with the **Quo** instruction. Our implementation reflected this specification:

  ocaml \| Quotient \(e1, e2\) -&gt; List.append\(List.append \(translate e1\) \(translate e2\)\) \[Quo\]

* **Induction step for remainders:** for any arithmetic expression e1 \(represented as e1\) that was translated as the list of byte-code instructions bcis1 \(which is the first induction hypothesis\) and any arithmetic expression e2 \(represented as e2\) that was translated as the list of byte-code instructions bcis2 \(which is the second induction hypothesis\),

  translating Remainder \(e1, e2\) yields the list of byte-code instructions that begins with the instructions in bcis1, continues with the instructions in bcis2, and ends with the **Rem** instruction. Our implementation reflected this specification:

  ocaml \| Remainder \(e1, e2\) -&gt; List.append\(List.append \(translate e1\) \(translate e2\)\) \[Rem\]

Finally, we verified that our function passed the unit-test function:

**Subsidiary questions:**

* Does your compiler translate from left to right or from right to left?

  The compiler translates from left to right because of the use of List.append\(\). To verify, we use the same examples as in Task 1. It is clear from the output that the compiler translates the source program to a list of byte-code instructions \(BCI\) from left to right.

* Does it make any observable difference whether your compiler translates from left to right or from right to left? Yes, it does. If the order of translation is changed, the order in which the corresponding BCI will be changed. This will cause `decode_execute` and `run` to generate different output. Therefore, the order of translation makes observable difference.

## Task 5: Implement an alternative compiler

Presumably your compiler \(in Task 4\) uses list concatenation \(i.e., List.append\). List concatenation, however, incurs a linear cost since it prepends its first argument onto its second by copying it.

Program an alternative version of your compiler that does not use list concatenation \(and verify that it passes the unit tests\).

The goal for this exercise is to bypass the use of List.append in order to avoid the linear cost. Our solution took inspiration from the mini-project about multiplying integers in a tree with an accumulator. In this exercise, instead of "accumulating\" the product after repeated multiplication, we are "accumulating\" the byte-code instruction after translation.

* Base case: `Literal n` is analogous to the `Leaf` case in the binary tree. Whenever we encounters `Literal n`, we yield a list by adding the BCI `Push n` into the existing BCIs and returns the updated accumulator. This is our base case.
* Induction step: Each of `Plus`,`Minus`,`Quotient`,`Remainder` is analogous of the `Node` case in the binary tree. The only difference \(and hence the challenge\) is that we now have "four distinct types of nodes\" and we need to specify that by adding the corresponding BCI \(`Add`, `Sub`,`Quo`,`Rem`\) after specifying the inductive hypothesis. Concretely, our implementation is as follows.

We verified that this version of the compiler also passed the unit-test.

## Task 6: Unit-test to verify interpreter and the compiler yield the same output

Implement a unit-test function to verify that interpreting source programs yields the same result as compiling them and running the resulting target programs.

Food for thought:

The resource file for the present lecture note contains a generator of random arithmetic expressions, generate\_random\_arithmetic\_expression that, given a non-negative integer n, yields a random arithmetic expression of depth at most n.

Inspired by the food for thought above, we made use of the `generate_random_arithmetic_expression` function to generate random arithmetic expressions to make up the source program and use it to build our unit-test function. While solving this exercises, we were reminded and inspired by exercise 2 in the mini-project about multiplying integers in a binary tree, where we expanded the unit-test function for one particular tree and then used a random binary tree generator to conduct the test several times to improve its robustness. Thus, to define the commutativity test, we first define the commutativty test for one source program:

With this, we expanded the unit test function:

Subsidiary question:

What is the impact of

* evaluating from left to right vs. from right to left, and of
* translating from left to right vs. from right to left on the commuting diagram? Does it always commute, no matter the order? If not, can you exhibit a counter-example? What is the consequence of this commutation or of this non-commutation?

  We made use of the unit-test function that we have just defined to test for commutativity:

  It was observed that both tests were passed. This means that if the evaluation order of the interpreter and the translation order of the compiler are the same, then the two commute. To be more precise, the compiler combined with the virtual machine is commutative with the interpreter. For simplicity, we may use `run(compile)` to refer to the compiler combined with the virtual machine which is defined in the function `run`.

  We confirmed our answer by printing out the unit-test function, using the function provided in the accompanying file. All the tests showed that the output result of the interpreter and compiler are the same. Below we have shown one of such examples.

  This showed that since the interpreter evaluates from left to right and the compiler translates from left to right, `interpret` generates the same output as `run(compile)`. We also verified that it is also the case when the target program has the type `Expressible_msg`:

## Task 7: The fold-right version of the interpreter and the compiler

Define the fold-right function associated with arithmetic\_expression and use it to express both interpret \(in Task 1\) and compile \(in Task 4\).

First, we defined the fold\_right arithmetic taking the the five cases as its parameters, namely, literal\_case \(the base case for the interpreter\), plus\_case \(induction step for additions\), minus\_case \(induction step for subtractions\), quotient\_case \(induction step for quotients\), remainder\_case \(induction step for remainders\). Concretely,

With the `fold_right_arithmetic` defined, we can rewrite the interpreter in terms of the `fold_right_arithmetic`, that is, with a structure parallel to the five cases so that each can be passed to `fold_right_arithmetic` as a parameter. Notice that the following implementation is the same as the previous version of interpreter, except that now we define each case _explicitly_ with let-expressions and at the end of the program pass the five cases to `fold_right_arithmetic`.

Finally, we verified that our fold-right version passed the unit-test function for interpreter.

Using exactly the same logic behind how we have rewritten the interpreter, it is rather straightforward to write the first version of our compiler in terms of `fold_right_arithmetic` by explicitly defining the cases \(namely, `literal_case`, `minus_case`, `quotient_case`, `remainder_case` and pass them to the previously defined `fold_right_arithmetic` function. Our implementation is as follows:

Finally, we verified that our fold-right compiler passed the unit-test function for compiler, and that the fold-right compiler combined with virtual machine \(i.e., the function `run`\) commutes with the fold-right interpreter:

## Task 8: The fold-right version of the alternative compiler

As a followup to Task 7 \(optional\), use the fold-right function associated with arithmetic\_expression to express the compiler from Task 5 \(optional\).

Rewriting the accumulator version of the compiler required some extra thoughts and considerations. The solution was, again, inspired by the mini-project about multiplying integers in a binary tree. We now need to remove the accumulator from the parameters, but we also need to define and initialize it. To resolve this issue, we return a function instead.

Finally, we verified that our second version of fold-right compiler passed the unit-test function for compiler, and that the this second version of fold-right compiler also commutes with the fold-right interpreter:

## Extension: the world seen from right to left

We were led to wonder: can we implement a right-to-left interpreter/compiler?

We implemented the right-to-left version by essentially - at the risk of oversimplifying of course - swapping e1 and e2. To be specific, in the right-to-left interpret function, we always `(evaluate e2)` and then match `(evaluate e1)` \(the opposite of what has been done in the left-to-right version\), for example:

Similarly, for `compile`, we swap the order of `translate`, for example:

ocaml \(\* in the list append version of compile \*\) \| Plus \(e1, e2\) -&gt; List.append \(List.append \(translate e2\) \(translate e1\)\) \[Add\]

\(\* in the accumulator version of compile\*\) \| Plus \(e1, e2\) -&gt; \(let a = \[Add\] @ a in translate e2 \(translate e1 a\)\)

The same logic applies to the right-to-left version for the folded interpret and compile, and more implementation details can be found in the code files. Note that other functions are not changed, including `fold_right_arithmetic`, `decode_execute`, `run`, since we are only changing the order of two actions, namely **evaluate** and **translate**.

Our implementation for the right-to-left version is symmetrical \("laterally\"\) to the left-to-right version. To test the new right-to-left version, we modified the unit-test function in the following parts.

Making use of the right-to-left implementation, we experimented with the different possible combinations of different versions of `interpret` and `run(compile)`: \(Note that in order to run both left-to-right and right-to-left language processors in one single OCaml session, we renamed the functions in a way that is quite self-explanatory.\)

From the above results, we can see that when the order of evaluation for the interpreter is the same as the order of translation for the compiler, i.e., either both are left-to-right or both are right-to-left, `interpret` and `run(compile)` are commutative.

Otherwise, we have two cases:

1. If the final `expressible_value` is of the type `Expressible_msg`, they are not commutative when \(unless by coincidence the message from `interpret` and the message from `run(compile)` _appear_ the same\).
2. If the final `expressible_value` is of the type `Expressible_int`, they are still commutative because they are observationally equivalent despite the difference in order.

## Conclusion

One thing that surprised us was the parallel structure in these implementations, which perhaps echoed an earlier metaphor of "the same elephant.\" More generally, in this project, we observed many pieces of the module coming together: the basics of computation, fold\_right recursion, order of evaluation, list constructor, stack, just to name a few. We started the course by discussing the use of interpreter and compiler on an abstract level. Now, we were finally able to implement them and compare them in more technical details.

