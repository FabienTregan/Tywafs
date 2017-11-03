# Tywafs
Teach Yourself Web Assembly From Specification

## Why ?

tl;dr : because we can.

* Tutorials will give you a pro-owned view, and most of the time you won't even get it, you'll rather just learn to repeat pieces of code without developping the understanding to tell wether it is the good way, in your particular case, to do things nor why you do them that way.
* Learning from specification might help you understand how things work under the hood, and will force you to build your own understanding. Also, it will teach you to read the specification, then you won't hesitate consulting it when appropriate.
* By taking a simple problem and trying to implement a solution using only the specification, you will feel like you are trying to solve a puzzle, understanding what to do with information, clues, and question you find along the way.
* You may have fun.

## Which specification ?

Web Assembly specification is available at https://webassembly.github.io/spec/

It gives the specification of both the execution environment and the thing that it will execute. Since I want to focus on the later, I will use fireFox (56.0.2 64bit) as en environment, and need access to the specification of the API that will help me run some WebAssembly in it.

It is found at https://developer.mozilla.org/en-US/docs/WebAssembly#API_reference

## Take a starting point

Developers should be proud of their traditions and history. After reading https://en.wikipedia.org/wiki/%22Hello,_World!%22_program , we can agree that this is the first thing we want to do.

Let's read the documentation, skipping any part that does not answer to an actual question I have. My first question is "how do I send some webassembly code to the browser" ?

The answer should be in the MDN documentation. The second object in the documentation is WebAssembly.Module and and it is said to contain the "stateless WebAssembly code". Sounds good. Let's see how to get an instance of it. The first parameter of the constructor is :

```
bufferSource
    A typed array or ArrayBuffer containing the binary code of the .wasm module you want to compile. 
```