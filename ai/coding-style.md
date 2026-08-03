---
title: Coding Style
category: AI Rules
tags:
  - ai-rules
  - style
  - lua
framework:
  - vorp
  - rsg
  - redem
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://www.lua.org/manual/5.4/
  - https://www.lua.org/pil/contents.html
---

# Coding Style

- Prefer local variables over globals.
- Keep functions small and single-purpose.
- Use descriptive names for game entities and framework handles.
- Separate server, client, and shared logic.
- Keep examples copy-pasteable.

## Lua habits

- Use tables for structured data.
- Use metatables only when they clearly reduce repetition.
- Prefer event-driven flow over busy loops.
- Use coroutines only when the yield points are explicit and safe.

---

## Source: Lua 5.4 Reference Manual

by Roberto Ierusalimschy, Luiz Henrique de Figueiredo, Waldemar Celes

Copyright © 2020–2025 Lua.org, PUC-Rio. Freely available under the terms of the Lua license.

### 1 – Introduction

Lua is a powerful, efficient, lightweight, embeddable scripting language. It supports procedural programming, object-oriented programming, functional programming, data-driven programming, and data description.

Lua combines simple procedural syntax with powerful data description constructs based on associative arrays and extensible semantics. Lua is dynamically typed, runs by interpreting bytecode with a register-based virtual machine, and has automatic memory management with a generational garbage collection, making it ideal for configuration, scripting, and rapid prototyping.

Lua is implemented as a library, written in clean C, the common subset of standard C and C++. The Lua distribution includes a host program called lua, which uses the Lua library to offer a complete, standalone Lua interpreter, for interactive or batch use. Lua is intended to be used both as a powerful, lightweight, embeddable scripting language for any program that needs one, and as a powerful but lightweight and efficient stand-alone language.

As an extension language, Lua has no notion of a "main" program: it works embedded in a host client, called the embedding program or simply the host. (Frequently, this host is the stand-alone lua program.) The host program can invoke functions to execute a piece of Lua code, can write and read Lua variables, and can register C functions to be called by Lua code. Through the use of C functions, Lua can be augmented to cope with a wide range of different domains, thus creating customized programming languages sharing a syntactical framework.

Lua is free software, and is provided as usual with no guarantees, as stated in its license. The implementation described in this manual is available at Lua's official web site, www.lua.org.

Like any other reference manual, this document is dry in places. For a discussion of the decisions behind the design of Lua, see the technical papers available at Lua's web site. For a detailed introduction to programming in Lua, see Roberto's book, Programming in Lua.

### 2 – Basic Concepts

This section describes the basic concepts of the language.

#### 2.1 – Values and Types

Lua is a dynamically typed language. This means that variables do not have types; only values do. There are no type definitions in the language. All values carry their own type.

All values in Lua are first-class values. This means that all values can be stored in variables, passed as arguments to other functions, and returned as results.

There are eight basic types in Lua: nil, boolean, number, string, function, userdata, thread, and table. The type nil has one single value, nil, whose main property is to be different from any other value; it often represents the absence of a useful value. The type boolean has two values, false and true. Both nil and false make a condition false; they are collectively called false values. Any other value makes a condition true. Despite its name, false is frequently used as an alternative to nil, with the key difference that false behaves like a regular value in a table, while a nil in a table represents an absent key.

The type number represents both integer numbers and real (floating-point) numbers, using two subtypes: integer and float. Standard Lua uses 64-bit integers and double-precision (64-bit) floats, but you can also compile Lua so that it uses 32-bit integers and/or single-precision (32-bit) floats. The option with 32 bits for both integers and floats is particularly attractive for small machines and embedded systems. (See macro LUA_32BITS in file luaconf.h.)

Unless stated otherwise, any overflow when manipulating integer values wrap around, according to the usual rules of two-complement arithmetic. (In other words, the actual result is the unique representable integer that is equal modulo 2^n to the mathematical result, where n is the number of bits of the integer type.)

Lua has explicit rules about when each subtype is used, but it also converts between them automatically as needed. Therefore, the programmer may choose to mostly ignore the difference between integers and floats or to assume complete control over the representation of each number.

The type string represents immutable sequences of bytes. Lua is 8-bit clean: strings can contain any 8-bit value, including embedded zeros ('\0'). Lua is also encoding-agnostic; it makes no assumptions about the contents of a string. The length of any string in Lua must fit in a Lua integer.

Lua can call (and manipulate) functions written in Lua and functions written in C. Both are represented by the type function.

The type userdata is provided to allow arbitrary C data to be stored in Lua variables. A userdata value represents a block of raw memory. There are two kinds of userdata: full userdata, which is an object with a block of memory managed by Lua, and light userdata, which is simply a C pointer value. Userdata has no predefined operations in Lua, except assignment and identity test. By using metatables, the programmer can define operations for full userdata values. Userdata values cannot be created or modified in Lua, only through the C API. This guarantees the integrity of data owned by the host program and C libraries.

The type thread represents independent threads of execution and it is used to implement coroutines. Lua threads are not related to operating-system threads. Lua supports coroutines on all systems, even those that do not support threads natively.

The type table implements associative arrays, that is, arrays that can have as indices not only numbers, but any Lua value except nil and NaN. (Not a Number is a special floating-point value used by the IEEE 754 standard to represent undefined numerical results, such as 0/0.) Tables can be heterogeneous; that is, they can contain values of all types (except nil). Any key associated to the value nil is not considered part of the table. Conversely, any key that is not part of a table has an associated value nil.

Tables are the sole data-structuring mechanism in Lua; they can be used to represent ordinary arrays, lists, symbol tables, sets, records, graphs, trees, etc. To represent records, Lua uses the field name as an index. The language supports this representation by providing a.name as syntactic sugar for a["name"]. There are several convenient ways to create tables in Lua.

Like indices, the values of table fields can be of any type. In particular, because functions are first-class values, table fields can contain functions. Thus tables can also carry methods.

The indexing of tables follows the definition of raw equality in the language. The expressions a[i] and a[j] denote the same table element if and only if i and j are raw equal (that is, equal without metamethods). In particular, floats with integral values are equal to their respective integers (e.g., 1.0 == 1). To avoid ambiguities, any float used as a key that is equal to an integer is converted to that integer. For instance, if you write a[2.0] = true, the actual key inserted into the table will be the integer 2.

Tables, functions, threads, and (full) userdata values are objects: variables do not actually contain these values, only references to them. Assignment, parameter passing, and function returns always manipulate references to such values; these operations do not imply any kind of copy.

The library function type returns a string describing the type of a given value.

#### 2.2 – Environments and the Global Environment

Any reference to a free name (that is, a name not bound to any declaration) var is syntactically translated to _ENV.var. Moreover, every chunk is compiled in the scope of an external local variable named _ENV, so _ENV itself is never a free name in a chunk.

Despite the existence of this external _ENV variable and the translation of free names, _ENV is a completely regular name. In particular, you can define new variables and parameters with that name. Each reference to a free name uses the _ENV that is visible at that point in the program, following the usual visibility rules of Lua.

Any table used as the value of _ENV is called an environment.

Lua keeps a distinguished environment called the global environment. This value is kept at a special index in the C registry. In Lua, the global variable _G is initialized with this same value. (_G is never used internally, so changing its value will affect only your own code.)

When Lua loads a chunk, the default value for its _ENV variable is the global environment. Therefore, by default, free names in Lua code refer to entries in the global environment and, therefore, they are also called global variables. Moreover, all standard libraries are loaded in the global environment and some functions there operate on that environment. You can use load (or loadfile) to load a chunk with a different environment. (In C, you have to load the chunk and then change the value of its first upvalue.)

#### 2.3 – Error Handling

Several operations in Lua can raise an error. An error interrupts the normal flow of the program, which can continue by catching the error.

Lua code can explicitly raise an error by calling the error function. (This function never returns.)

To catch errors in Lua, you can do a protected call, using pcall (or xpcall). The function pcall calls a given function in protected mode. Any error while running the function stops its execution, and control returns immediately to pcall, which returns a status code.

Because Lua is an embedded extension language, Lua code starts running by a call from C code in the host program. (When you use Lua standalone, the lua application is the host program.) Usually, this call is protected; so, when an otherwise unprotected error occurs during the compilation or execution of a Lua chunk, control returns to the host, which can take appropriate measures, such as printing an error message.

Whenever there is an error, an error object is propagated with information about the error. Lua itself only generates errors whose error object is a string, but programs can generate errors with any value as the error object. It is up to the Lua program or its host to handle such error objects. For historical reasons, an error object is often called an error message, even though it does not have to be a string.

When you use xpcall (or lua_pcall, in C) you can give a message handler to be called in case of errors. This function is called with the original error object and returns a new error object. It is called before the error unwinds the stack, so that it can gather more information about the error, for instance by inspecting the stack and creating a stack traceback. This message handler is still protected by the protected call; so, an error inside the message handler will call the message handler again. If this loop goes on for too long, Lua breaks it and returns an appropriate message. The message handler is called only for regular runtime errors. It is not called for memory-allocation errors nor for errors while running finalizers or other message handlers.

Lua also offers a system of warnings. Unlike errors, warnings do not interfere in any way with program execution. They typically only generate a message to the user, although this behavior can be adapted from C.

#### 2.4 – Metatables and Metamethods

Every value in Lua can have a metatable. This metatable is an ordinary Lua table that defines the behavior of the original value under certain events. You can change several aspects of the behavior of a value by setting specific fields in its metatable. For instance, when a non-numeric value is the operand of an addition, Lua checks for a function in the field __add of the value's metatable. If it finds one, Lua calls this function to perform the addition.

The key for each event in a metatable is a string with the event name prefixed by two underscores; the corresponding value is called a metavalue. For most events, the metavalue must be a function, which is then called a metamethod. In the previous example, the key is the string "__add" and the metamethod is the function that performs the addition. Unless stated otherwise, a metamethod can in fact be any callable value, which is either a function or a value with a __call metamethod.

You can query the metatable of any value using the getmetatable function. Lua queries metamethods in metatables using a raw access.

You can replace the metatable of tables using the setmetatable function. You cannot change the metatable of other types from Lua code, except by using the debug library.

Tables and full userdata have individual metatables, although multiple tables and userdata can share their metatables. Values of all other types share one single metatable per type; that is, there is one single metatable for all numbers, one for all strings, etc. By default, a value has no metatable, but the string library sets a metatable for the string type.

A detailed list of operations controlled by metatables is given next. Each event is identified by its corresponding key. By convention, all metatable keys used by Lua are composed by two underscores followed by lowercase Latin letters.

- **__add:** the addition (+) operation. If any operand for an addition is not a number, Lua will try to call a metamethod. It starts by checking the first operand (even if it is a number); if that operand does not define a metamethod for __add, then Lua will check the second operand. If Lua can find a metamethod, it calls the metamethod with the two operands as arguments, and the result of the call (adjusted to one value) is the result of the operation. Otherwise, if no metamethod is found, Lua raises an error.
- **__sub:** the subtraction (-) operation. Behavior similar to the addition operation.
- **__mul:** the multiplication (*) operation. Behavior similar to the addition operation.
- **__div:** the division (/) operation. Behavior similar to the addition operation.
- **__mod:** the modulo (%) operation. Behavior similar to the addition operation.
- **__pow:** the exponentiation (^) operation. Behavior similar to the addition operation.
- **__unm:** the negation (unary -) operation. Behavior similar to the addition operation.
- **__idiv:** the floor division (//) operation. Behavior similar to the addition operation.
- **__band:** the bitwise AND (&) operation. Behavior similar to the addition operation, except that Lua will try a metamethod if any operand is neither an integer nor a float coercible to an integer.
- **__bor:** the bitwise OR (|) operation. Behavior similar to the bitwise AND operation.
- **__bxor:** the bitwise exclusive OR (binary ~) operation. Behavior similar to the bitwise AND operation.
- **__bnot:** the bitwise NOT (unary ~) operation. Behavior similar to the bitwise AND operation.
- **__shl:** the bitwise left shift (<<) operation. Behavior similar to the bitwise AND operation.
- **__shr:** the bitwise right shift (>>) operation. Behavior similar to the bitwise AND operation.
- **__concat:** the concatenation (..) operation. Behavior similar to the addition operation, except that Lua will try a metamethod if any operand is neither a string nor a number (which is always coercible to a string).
- **__len:** the length (#) operation. If the object is not a string, Lua will try its metamethod. If there is a metamethod, Lua calls it with the object as argument, and the result of the call (adjusted to one value) is the result of the operation. If there is no metamethod but the object is a table, then Lua uses the table length operation. Otherwise, Lua raises an error.
- **__eq:** the equal (==) operation. Behavior similar to the add operation, except that Lua will try a metamethod only when the values being compared are either both tables or both full userdata and they are not primitively equal. The result of the call must be a boolean.
- **__lt:** the less than (<) operation. Behavior similar to the add operation, except that Lua will try a metamethod only when the values being compared are neither both numbers nor both strings. The result of the call must be a boolean.
- **__le:** the less equal (<=) operation. Unlike other operations, the less-equal operation can use two different events. First, Lua looks for the __le metamethod in both operands, like in the less than operation. If it cannot find such a metamethod, then it will try the __lt metamethod, assuming that a <= b is equivalent to not (b < a). As with the other comparison operators, the result must be a boolean.
- **__index:** The indexing access table[key]. This event happens when table is not a table or when key is not present in table. The metamethod is looked up in table.
- **__newindex:** The indexing assignment table[key] = value. Like the index event, this event happens when table is not a table or when key is not present in table. The metamethod is looked up in table.
- **__call:** called when Lua calls a value. This happens when Lua tries to call something that is not a function (i.e., value does not have a __call metamethod either). The metamethod is looked up in value.
- **__close:** called when a to-be-closed variable goes out of scope.
- **__mode:** Makes a table have weak keys and/or values. This can be used to allow garbage collection of otherwise unused table entries.
- **__tostring:** Called by tostring. Must return a string.
- **__gc:** Called when the userdata is garbage collected. Only works for userdata in Lua 5.4 and full userdata.
- **__pairs:** Called by the pairs function. Must return an iterator function, a state, and an initial value for the iterator variable.
- **__name:** This is not a metamethod, but a field used by luaL_newmetatable to set the name of the metatable for use in error messages.

---

## Source: Programming in Lua (first edition)

by Roberto Ierusalimschy. Lua.org, December 2003. ISBN 8590379817.

This first edition was written for Lua 5.0. While still largely relevant for later versions, there are some differences. All corrections listed in the errata have been made in the online version.

### 1 – Getting Started

To keep with the tradition, our first program in Lua just prints "Hello World":

```
print("Hello World")
```

If you are using the stand-alone Lua interpreter, all you have to do to run your first program is to call the interpreter (usually named lua) with the name of the text file that contains your program. For instance, if you write the above program in a file hello.lua, the following command should run it:

```
prompt> lua hello.lua
```

As a slightly more complex example, the following program defines a function to compute the factorial of a given number, asks the user for a number, and prints its factorial:

```
-- defines a factorial function
function fact (n)
  if n == 0 then
    return 1
  else
    return n * fact(n-1)
  end
end

print("enter a number:")
a = io.read("*number") -- read a number
print(fact(a))
```

If you are using Lua embedded in an application, such as CGILua or IUPLua, you may need to refer to the application manual (or to a "local guru") to learn how to run your programs. Nevertheless, Lua is still the same language; most things that we will see here are valid regardless of how you are using Lua. For a start, we recommend that you use the stand-alone interpreter (that is, the lua executable) to run your first examples and experiments.

### 2 – Types and Values

Lua is a dynamically typed language. There are no type definitions in the language; each value carries its own type.

There are eight basic types in Lua: nil, boolean, number, string, userdata, function, thread, and table. The type function gives the type name of a given value:

```
print(type("Hello world"))   --> string
print(type(10.4*3))           --> number
print(type(print))            --> function
print(type(type))             --> function
print(type(true))             --> boolean
print(type(nil))              --> nil
print(type(type(X)))          --> string
```

The last example will result in "string" no matter the value of X, because the result of type is always a string.

Variables have no predefined types; any variable may contain values of any type:

```
print(type(a))       --> nil (`a' is not initialized)
a = 10
print(type(a))       --> number
a = "a string!!"
print(type(a))       --> string
a = print            -- yes, this is valid!
a(type(a))           --> function
```

Notice the last two lines: Functions are first-class values in Lua; so, we can manipulate them like any other value. Usually, when you use a single variable for different types, the result is messy code. However, sometimes the judicious use of this facility is helpful, for instance in the use of nil to differentiate a normal return value from an exceptional condition.
