# Notes #1
### Scope

In traditional compiled-language process, a your program will undergo typically three steps _before_ it is executed, roughly called "compilation: 
 1. Tokenizing/Lexing
 	- Breaking up a string of characters into meaningful chunks, called tokens.
 2. Parsing
 	- Taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program.  This tree is called an "AST" (_abstract syntax tree_)
 3. Code Generation
 	- The process of taking an AST and turning it into executable code.  This part varies greatly depending on the language, the platform it's targeting, and so on.

### Lexical Scope

Lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code.

Scope look-up stops once it finds its first match.  The same identifies name can be specified at multiple layers of nested scope, which is called "shadowing" (the inner identifier "shadows" the outer identifier)

No matter _where_ a function is invoked from, or even _how_ it is invoked, its lexical scope is _only_ defined by where the function was declared.

### Function Versus Block Scope

Function scope encourages the idea that all variables belong to the function, and can be used and reused throughout the entirety of the function.  This design approach can be quite useful, and certainly can make full use of the  "dynamic" nature of JavaScript variables to take on values of different types as needed.

You can "hide" variables and functions by enclosing them in the scope of a function.

##### Why would "hiding" variables and functions be a useful technique?

One motivation is the _Principle of Least Privilege_, sometimes called _Least Authority_ or _Least Exposure_.  The principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and "hide" everything else.

Another benefit of "hiding" variables and functions inside a scope is to avoid unintended collision between two different identifiers with the same name but different intended usages.  Collision results often in unexpected overwriting of values.

##### Global namespaces

A standard application uses multiple libraries, and these libraries typically will create a single variable declaration, often an object, with a sufficiently unique name, in the global scope.  This object is then used as a _namespace_ for that library, where all specific exposures of functionality are made as properties off that object (namespace), rather than as top-level lexically scoped identifiers themselves.

For example:

```
 var MyLibrary = {
   name: "Reed",
   doSomething: function() {
    // ... 
   },
   doSomethingElse: function() {
    // ...
   }
 };
 ```
 
Another option for collision avoidance is the more modern _module_ approach, using any of various dependency managers. Using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifier(s) be explicitly imported into another specific scope through usage of the dependency manager's various mechanisms.

The easiest way to distinguish declaration vs. expression is the position of the word `function` in the statement.  If `function` is the very first thing in the statement, then it's a function declaration. Otherwise, it's a function expression.

_Inline function expressions_ are powerful and useful. Providing a name for your function expression quite effectively addresses some drawbacks of keeping your function anonymous.

For example:

```
setTimeout( function timeoutHandler() { 
  console.log("one second later..."); 
}, 1000);
```

### Hoisting

All declarations, both variables and functions, are processed first, before any part of your code is executed.  When you see `var a = 2`, one might think that is one statement.  But JavaScript considers it as two statements:
 - `var a;`
 - `a = 2`

The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left _in place_ for the execution phase.

Functions are hoisted first, and then variables.

### Scope Closure

Closures happen as a result of writing code that relies on lexical scope.  

Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

```
function foo() {
  var a = 2;
  
  function bar() {
    console.log(a); // 2
  }
  
  bar();
}

foo();
```

Function `bar()` has _access_ to the variable `a` in the outer enclosing scope because of lexical scope look-up rules (in this case, it's an RHS reference look-up).

It is said that `bar()` closes over the scope of `foo()` because `bar()` appears nested inside of `foo()`.

Whatever facility we use to _transport_ an inner function outside of its lexical scope, it will maintain a scope reference to where it was originally declared, and wherever we execute him, that closure will be exercised.

Linters often complain when you put functions inside of loops, because the mistakes of not understanding closures are common.

```
for (var i=1; i<=5; i++) {
  setTimeout( function timer() {
    console.log(i);
  }, i*1000);
}
```

The above code does not print out 1 through 5, but instead prints out 6 five times.  The mistake is that we are trying to _imply_ that each iteration of the loop "captures" its own copy of `i` at the time of the iteration.  But the way scope works, all five of those functions, though they are defined separately in each loop iteration, _are closed over the same shared global scope_, which has only one `i` in it.

You can fix this by wrapping your loop in an IIFE so that each iteration can have its own copy of `i`

```
for (var i=1; i<=5; i++) {
  (function(j) {
    setTimeout( function timer() {
      console.log(j);
    }, j*1000);
  })(i);
}
```
### Modules

```
function CoolModule() {
  var something = "cool"; 
  var another = [1, 2, 3];
  
  function doSomething() { 
    console.log( something );
  }
  
  function doAnother() {
    console.log( another.join( " ! " ) );
  }
  
  return {
    doSomething: doSomething, 
    doAnother: doAnother
  };
}
var foo = CoolModule(); 

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

`CoolModule()` is just a function, but it _has to be invoked_ for there to be a module instance created.

The `CoolModule()` function returns an object, denoted by the object-literal syntax `{ key: value, ... }`.  The object we return has references on it to our inner functions, but _not_ to our inner data variables.  We keep those hidden and private.  It's appropriate to think of this object return value as essentially a _public API for our module_.

There are two requirements for the module pattern to be exercised.
 1. There must be an outer enclosing function, and it must be invoked at least once (each time creates a new module instance).
 2. The enclosing function must return back at least one inner function, so that this inner function has closure over the private scope, and can access and/or modify that private state.
 

# JS Notes #2

### `this` and prototype

`this` is not an author-time binding but a runtime binding.  It is contextual based on the conditions of the function's invocation. `this` binding.

To understand `this` binding, we have to understand the _call-site_: the location in code where a function is called (not where it's declared). What's important is to think about the _call-stack_ (the stack of functions that have been called to get us to the current moment in execution). The call-site we care about is _in_ the invocation _before_ the currently executing function.  

Consider the following code: 

```
function foo() {
  console.log(this.a);
}

var a = 2;

foo(); // 2 
```

When `foo()` is called, `this.a` resolves to our global variable `a`.  This is because, in this case, the _default binding_ for `this` applies to the function call, and so points `this` to the global object.  

### Implicit Binding

Another rule to consider is whether the call-site ha a context object, also referred to as an owning or containing object.

Consider: 

```
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2, 
  foo, foo
};

obj.foo(); // 2
```

The call-site _uses_ the `obj` content to reference the function, so you _could_ say that the `obj` object "owns" or "contains" the function reference at the time the function is called. 

When there is a context object for a function reference, the _implicit binding_ rule says that it's _that_ object that should be used for the function call's `this` binding.  

### Explicit Binding

Consider: 

```
function foo() {
  console.log(this.a);
}

var obj = { 
  a: 2
};

foo.call(obj); // 2
```

Invoking `foo` with _explicit binding_ by `foo.call(...)` allows us to force its `this` to be `obj`.  

The most typical way to wrap a function with a _hard binding_ creates a pass-through of any arguments passed and any return value received:  

```
function foo(something) { 
  console.log( this.a, something ); 
  return this.a + something;
}
var obj = {
  a:2
};

var bar = function() {
  return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
Another way to express this pattern is to create a reusable helper:
```
function foo(something) { 
  console.log( this.a, something ); 
  return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) { \
  return function() {
    return fn.apply( obj, arguments ); 
  };
}
var obj = { 
  a:2
};

var bar = bind( foo, obj ); 

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

### `new` Binding

There really is _no connection_ to class-oriented functionality implied by `new` usage in JS.  In JS, constructors are just functions that happen to be called with the `new` operator in front of them.

When a function is invoked with new in front of it, otherwise known as a constructor call, the following things are done automatically:

1. A brand new object is created (aka constructed) out of thin air.
2. The newly constructed object is [[Prototype]]-linked.
3. The newly constructed object is set as the this binding for that function call.
4. Unless the function returns its own alternate object, the `new` invoked function call will automatically return the newly constructed object.

### Binding Precedence

_default binding_ is the lowest priority rule of the four types of binding.

_explicit binding_ takes precedence over _implicit binding_, which means you should ask first if _explicit binding_ applies before checking for _implicit binding_.

`new` is able to override _hard binding_, but is this useful?

The primary reason for this behavior is to create a function (that can be used with `new` for constructing objects) that essentially ignores the `this` _hard binding_, but which presets some or all of the function's arguments.  One of the capabilities of `bind(..)` is that any arguments passed after the first `this` binding argument are defaulted as standard arguments to the underlying function (technically called "partial application," which is a subset of "currying").

### Determining `this`

Follow these questions in order, stopping at the first applicable rule.

1. Is the function called with `new`? If so, `this` is the newly constructed object.
 - `var bar = new foo()`
   
2. Is the function called with `call` or `apply` (_explicit binding_), even hidden inside a `bind` hard binding? If so, `this` is the explicitly specified object.
 - `var bar = foo.call( obj2 )`
 
3. Is the function called with a context (_implicit binding_), otherwise known as an owning or containing object? If so, `this` is that context object.
 - `var bar = obj1.foo()`
 
4. Otherwise, default the `this` (_default binding_). If in strict mode, pick undefined, otherwise pick the global object.
 - `var bar = foo()`
