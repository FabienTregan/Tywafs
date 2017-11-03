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
>bufferSource
>    A typed array or ArrayBuffer containing the binary code of the .wasm module you want to compile. 


Ok, great. Hopefully this ".wasm module" is not tied to browser but is specified by the web assembly thing. Let look at the specification. Use navigator to seach for "module" in the index and we find : https://webassembly.github.io/spec/syntax/modules.html
>WebAssembly programs are organized into modules, which are the unit of deployment, loading, and compilation.
Ok, this is what we are looking for.

Now how to make one ? Just after the first paragph, there is a block of text starting with `module ::=`. If you are not familiar with the notation, it might be time to read the conventions used in the [specification](https://webassembly.github.io/spec/syntax/conventions.html) and come back, but for now a basic understanding is enough : a module is made of types, funcs, tables, mems... all of which are mandatory and can have one or zero start.

Since we want to start, let's have a look at this `start` : 
>The start component of a module optionally declares the function index of a start function that is automatically invoked when the module is instantiated, after tables and memories have been initialized.

Ok, so we will need to have a function probably in the `funcs` part of the module, and if we want a start we will need to know its index, of which the value will be assigned to `start`. 