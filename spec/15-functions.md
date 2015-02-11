#Functions

##General

When a function is called, information may be passed to it by the caller
via an *argument list*, which contains one or more *argument
expressions*, or more simply, *arguments*. These correspond by position
to the *parameters* in a *parameter list* in the called function's
definition (§15.3). Hack supports *variadic functions*; that is, functions that can be called with a variable number of arguments. 

Any function containing `yield` ([§§](10-expressions.md#yield-operator)) is a *generator function*.

##Function Calls

A function is called via the function-call operator `()` (10.5.6).

##Function Definitions

**Syntax**

<pre>
  <i>function-deﬁnition:</i>
    <i>function-deﬁnition-header   compound-statement</i>

  <i>function-deﬁnition-header:</i>
    <i>attribute-specification<sub>opt</sub></i>  async<sub>opt</sub>  function <i>name</i>  <i>generic-type-parameter-list<sub>opt</sub></i>  (  <i>parameter-list<sub>opt</sub></i>  ) :  <i>return-type</i>

  <i>parameter-list:</i>
    ...
    <i>parameter-declaration-list</i>
    <i>parameter-declaration-list</i>  ,  ...

  <i>parameter-declaration-list:</i>
    <i>parameter-declaration</i>
    <i>parameter-declaration-list</i>  ,  <i>parameter-declaration</i>

  <i>parameter-declaration:</i>
    <i>attribute-specification<sub>opt</sub>  <i>type-specifier</i>  <i>variable-name  default-argument-specifier<sub>opt</sub></i>

  <i>default-argument-specifier:</i>
    =  <i>const-expression</i>

  <i>return type:</i>
    <i>type-specifier</i>
    this
</pre>

*compound-statement* is defined in §11.2; *attribute-specification* is defined in §21.2; *name* is defined in §9.4.4.2; *generic-type-parameter-list* is defined in §14.2; *type-specifier* is described in §5.1; *variable-name* is defined in §9.4.4.2; and *const-expression* is defined in [§§](10-expressions.md#constant-expressions).

**Constraints**

The name of a method must not be the same as that of its parent class.

Each *variable-name* in a *parameter-declaration-list* must be distinct.

If any *parameter-declaration* has a *default-argument-specifier*, then all subsequent *parameter-declarations* in the same *parameter-declaration-list* must also have a *default-argument-specifier*.

If the *type-specifier* in *return-type* is `void`, the *compound-statement* must not contain any `return` statements (§11.7.4) having an *expression*. Otherwise, all `return` statements must contain an *expression* whose type is a subtype of the type indicated by *type-specifier*.

If *async* is present, *return-type* must be a type that implements `Awaitable<T>` (§17.6.2). 

A generic function and a non-generic function in the same scope cannot have the same *name*.

If the function being defined is an overriding method in a derived class, the *return-type* of the derived-class method must be a subtype of the *return-type* of the base-class method.

A function that is not a method must not have a *return-type* of `this`.

**Semantics**

A *function-definition* defines a function called *name*. A function can be defined with zero or more parameters, each of which is specified in its own *parameter-declaration* in a *parameter-declaration-list*. Each parameter has a name (*variable-name*), a type (*type-specifier*), and optionally, a *default-argument-specifier*. 

If *parameter-list* contains an ellipses (`...`), the function is variadic, in which case, *parameter-declaration-list* (if any) to the left of this ellipse represents the *fixed parameter list*. The ellipsis represents the *variable parameter list*. A non-variadic function has only a fixed parameter list. If *parameter-list* is omitted, the function has no parameters, in which case, both the fixed and the variable parameter lists are empty.

When the function is called, if there exists a parameter for which there is a corresponding argument, the argument is assigned to the parameter variable. If that parameter has no corresponding argument, but the parameter has a *default-argument-specifier*, the default value is assigned to the parameter variable. After all parameters have been assigned initial values, the body of the function, *compound-statement*, is executed. This execution may terminate normally (§4.3) or abnormally (§4.3).

Each parameter is a variable local to the parent function, it is a modifiable lvalue, and its type can change. A parameter's *type-specifier* constrains the type of any corresponding argument in a call to that function and in subsequent uses of that parameter's name in a non-lvalue context, provided that parameter is not used in an lvalue context. However, the *type-specifier* does not constrain the use of that parameter's name in an lvalue context. For example, a parameter *$p1* having a *type-specifier* of `int` could have assigned to it in the *compound-statement* the value of a `float`, in which case, being a local variable, `$p1` takes on that type instead.

There may be more arguments than parameters, in which case, the library functions `func_num_args` (§xx), `func_get_arg` (§xx), and `func_get_args` (§xx) can be used to get access to the complete argument list that was passed.

The presence of the `async` modifier declares the function to be asynchronous (§15.5). For an async function, control may be transferred back to the caller before the flow of execution reaches any `return` statement or the function-closing brace. In such a case, the Awaitable object that will be returned to the caller later on acts like a placeholder that will eventually be filled with the return result.

A method having a *return-type* of `this` allows that method to return an object created via `new static` (§10.5.3). In such cases, the type actually returned depends on whether the method is called on an instance of its own class or on one that is an instance of a derived class. (See §21.3.2 for an example.)

**Examples**

```php
function square(num $v): num {
  return $v * $v;
}
// -----------------------------------------
function break_string_into_substrings(string $str, string $separator = ';'):
  array<string> {
  $result = array();
  …
  return $result;
}
// -----------------------------------------
function addVector(Vector<int> $v1, Vector<int> $v2): Vector<int> {
  $result = Vector{};
  // add two vectors
  return $result;
}
// -----------------------------------------
function factorial(int $i): int   // contains a recursive call {
  return ($i > 1) ? $i * factorial($i - 1) : $i;
}
// -----------------------------------------
function variable_args(...): void {   // variadic function
  $argList = func_get_args();
  echo "# arguments passed is " . count($argList)."\n";

  foreach ($argList as $k => $e) {
    echo "\targ[$k] = >$e<\n";
  }
}
```


##Anonymous Functions

An *anonymous function*, also known as a *closure*, is a function
defined with no name. As such, it must be defined in the context of an
expression whose value is used immediately to call that function, or
that is saved in a variable of closure type (§5.17) for later execution. An anonymous function
is defined via the anonymous function-creation operator (§10.4.6).

For both `__FUNCTION__` and `__METHOD__` ([§§](06-constants.md#context-dependent-constants)), an anonymous
function's name is `{closure}`. All anonymous functions created in the
same scope have the same name.

##Asynchronous Functions

The term *asynchronous programming* generally refers to design patterns that allow for cooperative transfer of control between several distinct tasks on a given thread of execution (or possibly a pool of threads) in a manner that isolates tasks from each other and minimizes unnecessary dependencies and interactions between tasks. Asynchronous programming is often used in the context of I/O, and with tasks that depend on I/O. Using asynchronous programming can make it significantly easier for a program to batch work together when calling synchronous I/O APIs (APIs that cause the thread to block until the I/O operation is complete) and it can make it easier to use asynchronous APIs (APIs that allow a thread to continue executing while the I/O operation is in progress).

A function (or method) is declared to be asynchronous via the function modifier `async` (§15.3). The execution of an asynchronous function can be *suspended*; that is, control is returned to its caller, until a designated *await condition* is satisfied. This condition is specified via the operator `await` (§10.6.6).

When an async function is compiled, a special transformation is performed, so that an object that implements `Awaitable<T>` (§17.6.2) is returned to the caller (where `T` is the function's *return-type*). This object is used to keep track of the state of execution for the async function and to keep track of dependencies on other tasks. Each time a given async function is called, a newly allocated object is returned to the caller.

When an async function is invoked, it executes synchronously until it returns normally, it throws an error/exception, or it reaches an await operation.

The await operation provides a way for an async function to yield control in cases where further progress cannot be made until the result of a dependency is available.

An await operation takes a single operand called the *dependency*, which must be an object that implements the `Awaitable` interface. When an await operation executes, it checks the status of the dependency. If the result of the dependency is available, the await operation produces the result (without yielding control elsewhere) and the current async function continues executing. If the dependency has failed due to an exception, the await operation re-throws this exception. Otherwise, the await operation updates the current async function's `Awaitable` to reflect its dependence on the dependency and then yields control. The async function will be considered *blocked* and ineligible to resume execution until the result of the dependency is available.

When yielding control, the implementation of `await` may choose to yield control back to the current async function's caller. Alternatively, if the dependency is a task and its async function is not blocked (i.e., it is eligible to resume execution), the implementation may choose (for optimization purposes) to perform an invocation to resume the dependency's async function to make further progress and then check again if the result of the dependency is available.

If an await operation would result in a dependency cycle (i.e., a task waiting on itself, or a group of two or more tasks that wait on each other in a cycle), the await operation will fail and an exception will be thrown.

When a `return` statement executes inside an async function, typically, control is transferred back to the caller. However, the implementation may choose (for scheduling or optimization reasons) to perform invocations to resume other dependent async functions that are now eligible to execute before ultimately transferring control back to the caller.

If an async function returns a value of type `T`, the return type visible to its callers is `Awaitable<T>`. (If an async function returns no value, the return type visible to its callers is `Awaitable<void>`.)

