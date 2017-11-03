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
>`bufferSource`
>
>    A typed array or ArrayBuffer containing the binary code of the .wasm module you want to compile.


Ok, great. Hopefully this ".wasm module" is not tied to browser but is specified by the web assembly thing. Let look at the specification. Use navigator to seach for "module" in the index and we find : https://webassembly.github.io/spec/syntax/modules.html
>WebAssembly programs are organized into modules, which are the unit of deployment, loading, and compilation.
Ok, this is what we are looking for.

Now how to make one ? Just after the first paragph, there is a block of text starting with `module ::=`. If you are not familiar with the notation, it might be time to read the conventions used in the [specification](https://webassembly.github.io/spec/syntax/conventions.html) and come back, but for now a basic understanding is enough : a module is made of types, funcs, tables, mems... all of which are mandatory and can have one or zero start.

Since we want to start, let's have a look at this `start` :
>The start component of a module optionally declares the function index of a start function that is automatically invoked when the module is instantiated, after tables and memories have been initialized.

Ok, we are probably in the right place. We know we have to make a module with at least those parts :

* types vec(functype)
* funcs vec(func)
* tables vec(table)
* mems vec(mem)
* globals vec(global)
* elem vec(elem)
* data vec(data)
* imports vec(import)
* exports vec(export)

## Building an unpopulated module.

We could now try to make a `func`, then build a `funcs` 'vec' (probably a vector ?) from it, and carry on until all the content of a `module` is ready, or we can go the other way around : start making a module then populate it. Let's try the later.

The `modules` page of the specification tells me what, from a logical point of view, is inside a module, but not how to build a module and actually get the .wasm file we need to give to the WebAssembly API in the browser.

Looking at the index on the right side, we see two top level chapter which are "Binary Format" and "Text Format". Having a look at the "Text Format", going to the part dedicated to "Modules", it describes, as expected, a text format to describe a module.

But remember what we have in the API:
>`bufferSource`
>
>    A typed array or ArrayBuffer containing the binary code of the .wasm module you want to compile.
It does not expect a text representation. Searching for "text" in this page gives no result, so I probably need to either have a look at the binary format, or find a tool to convert from text to binary format (which hopefully should be called an Assembler ?)

The second option seems the abvious one. But finding and installing the assembler won't be fun. I will probably need to compile it and, since I boot my computer under Windows this morning, I will probably need to install a toolchain before I can compile the assembler. It seems that for now at least, I will have more fun looking at the [Binary Format](https://webassembly.github.io/spec/binary/index.html).

We will, of course, skip the Conventions sub-chapter, and go directly to the last chapter : "[Modules](https://webassembly.github.io/spec/binary/modules.html)". The short text introduction is interesting : we basically will need to write one "section" for each record of a module, except for the "function" record that is split in two sections. We skip all the Indices part for now, and go to Sections :

>Each section consists of
>
>* a one-byte section id,
>
>* the u32 size of the contents, in bytes,
>
>* the actual contents, whose structure is depended on the section id.

ok, easy. We then have a table of the section Id's, with a 0 Id-ed "custom" section (which we apparently can ignore), and eleven sections. We have nine mandatory records in the module, one having two corresponding sections, plus the opional "start" section. Eleven section. Everything is here, we should soon be able to write some code. Or opcodes. Bytes.

Binary formats often have headers and footers. It is a bit unexpected that we directly ran into the Sections. Looking at the index, we see that the last sub-chapter is about... modules !

The introduction text makes things easy to understand, and using the [binary grammar](https://webassembly.github.io/spec/binary/conventions.html#grammar) we are told how a module is made from bytes !

It starts with `0x00 0x61 0x73 0x6D 0x01 0x00 0x00 0x00` (module version 1) and have sections that all can be empty (but still present) and no footer.

So we basically may have succeed writing our unpopulated module !
```
0x00 0x61 0x73 0x6D             # Magic number, indicates this is a WebAssembly module
0x01 0x00 0x00 0x00             # Version 1 of the WebAssembly binary format
```

## Populating the module with empty sections

Since we need all the (non custom) sections to be present once, let's start with the first one : `typesec`, which contains any quantity of `functype`. This quantity will be zero for now, making things easyer. clicking on "typesec" we reach it's definition. It has the Id 1 and decodes into a vector of function types :
>`typesec::=ft∗:section1(vec(functype))⇒ft∗`
From the binary grammar in the convention chapter, we have :
>x:B denotes the same language as the nonterminal B, but also binds the variable x to the attribute synthesized for B.
and :
>Productions are written sym::=B1⇒A1 | … | Bn⇒An, where each Ai is the attribute that is synthesized for sym in the given case, usually from attribute variables bound in Bi.
What does that mean ? That typesec will be made of `section1(vec(functype))` (probably a `section1` containing an array of `functype`) that we will name that we will name `ft*`, and when this data will be loaded, the result will be `ft*`. Why some so complicated notation ? Because some of the data before the arrow might be used only to control the creation of the structure / object (e.g. : size, checksum, define nature of data...) but not be part of the output of parsing the data.

We already hade a look at the specification where it defines [`section1`](https://webassembly.github.io/spec/binary/modules.html#sections) :
```
sectionN(B)::=
	N:byte  size:u32  cont:B⇒cont
	|ϵ⇒ϵ
```
The second alternative means that if `B` is empty, then `sectionN` is just empty. Great, nothing to do !

You can see that, if we hade a non-empty section, it would start with N (the section Id), then the size (of cont), then cont. But only cont (after the double arrow) will be produced when parsing the data.

Now all the sections that only contains a vector can be empty, which are they ? All but `start`, which is optional. So our module should already be valid !

Can we test it ? We should just need to put our byte sequence into a file and load it using the API.

## Testing a first module

According to the API, we should be able to instanciate a module from a TypedArray, let's try this in the JS console in FireFox :
```JavaScript
var moduleBytes = new Uint8Array([0x00, 0x61, 0x73, 0x6D, 0x01, 0x00, 0x00, 0x00])
var myModule = new WebAssembly.Module(moduleBytes);
```
It works !! Does it really ? Try changing the version number from 0x01 to 0x02 and see if you still can instantiate a WebAssembly.Module :
>`CompileError: at offset 8: binary version 0x2 does not match expected version 0x1`

:)

##  Adding a section

By looking at the JS API, it seems that we need to export something in order to be able to call it from the JS side :

>`WebAssembly.Module.exports()`
>
>Given a `Module`, returns an array containing descriptions of all the declared exports.

Then we probably need to have something in the `Export` section, defined as this :
```
exportsec  ::= ex*:section7(vec(export)) ⇒ ex*
export ::= nm:name d:exportdesc  ⇒ {name nm,desc d}
exportdesc ::= 0x00 x:funcidx  ⇒ func x
	|0x01 x:tableidx ⇒ table x
	|0x02 x:nameidx  ⇒ name x
	|0x03 x:memidx ⇒ memid x
```
So we want a not empty section, containing a vector of exports, the export seems to be a succession of a name and an `exportdesc`, which is a function, table, name, or memory ID.

We want our JS to call one of our functions, so we need a funicdx which is a `u32` (unsigned 32bit). And digging the specification in /Structure/Modules/Indices, we find :
>Definitions are referenced with zero-based indices. Each class of definition has its own index space,

Now we need to understand what is the "index space" for the class funcidx. All I could find for now is :
>The index space for functions, tables, memories and globals includes respective imports declared in the same module. The indices of these imports precede the indices of other definitions in the same index space.

If I understand correctly, it means that if I have no import, the function Id will be the (zero based) index of the function in the vector of the function section. Since I have only one function for now, I should be 0.

We'll build the function section later, for now is seems that the `exportdesc` is rather simple : a 0 (to indicate we export a function), followed by another 0 (index of the function we will write) : `0 0`

From this, we should be able to write the `export`.

The export needs a name (`nm`), which is probably the name under wich the function will be available in the `Instance.prototype.exports` object on the JS side, and the `exportdesc` we just encoded.

Let's see how to encode names in the spec [binary format / values / name](https://webassembly.github.io/spec/binary/values.html#names) :
>Names are encoded as a vector of bytes containing the Unicode UTF-8 encoding of the name’s code point sequence.
>`name::=b∗:vec(byte)⇒name`

Easy ?

We can decide our function to be names "foo" (`[102,111,111]` in UTF8), which is hasa length of 3, and voila! only terminal symbols (it mean we no longer have to digg, we can climb up now)
```
//exportsec (section7(vec(export)))
	7 //sectionId 7 (export section)
	7 //size 7 (number of bytes in the content of the section)
	//content of the section (vect(export))
	1 //size 1
	//export
		//nm:name (which is a vect(byte))
			3 //size 3
			102, 111, 111 //UTF8 for "foo"
		//d:exportdesc
			0 //it is a function
			0 //funcidx of the first function in function section (if we have no import)
```

Ok, this might work. But we can not test for now because we do not have the function section but we use a funcidx.

You might have noticed that the u32's have been encoded as a single byte instead of the abvious 4. We'll come back to this later, but the spec in [binary format / values / integer](https://webassembly.github.io/spec/binary/values.html#integers) tells us that they are encoded in [LEB128](https://en.wikipedia.org/wiki/LEB128). All you need to know for now is that u32 smaller than 128 will be encoded in just one byte, saving space.

We can make a simpler exportsec by puting a 0-length vector as the content of the section, dodging the need for the function section for now. I will let you do this as an exercice, but once you add the section to the javascript array we made earlier, you should find this:
```JavaScript
var moduleBytes = new Uint8Array([0x00, 0x61, 0x73, 0x6D, 0x01, 0x00, 0x00, 0x00, 0x07, 0x01, 0x00])
var myModule = new WebAssembly.Module(moduleBytes);
```
