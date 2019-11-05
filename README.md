# Ride 
A smart contract language for Waves Platform

## Introduction
Ride is Waves Platform’s purpose-designed programming language for smart contracts. It was created to address many of the most serious shortcomings of other popular smart contract languages. The overall idea was to offer a straightforward functional language for dApp development on the Waves blockchain. 

Ride is easy to learn, especially for beginning developers. This brochure gives a comprehensive introduction to Ride, along with examples and further tools and resources.

## Overview
Ride is a statically-typed, lazy, functional, expression-based compiled programming language. It is designed for building developer-friendly decentralized applications (dApps).

Ride is not Turing Complete and its execution engine (virtual machine) doesn’t have any concept of loops or possibility for recursions. Also, there are a number of limitations by design, helping to ensure execution is secure and straightforward. However, we recognize that iterations are necessary and have implemented them as FOLD macros (see below). One of the key features is that the execution cost is always predictable and known in advance, so all code executes as intended with no failed transactions recorded on-chain – removing a significant source of frustration.

Despite being simple to use, however, Ride is powerful and offers wide-ranging functionality to developers. It’s broadly based on Scala and is also influenced by F# and the functional paradigm.

Ride is simple and concise. It will take around an hour to read this brochure, after which you will know everything about the Ride and opportunities that it gives for dApps development.

## Disclaimer
Ride Standard Library (STDLIB) is under active development. At the time of publication, the most up-to-date version is STDLIB_VERSION 3, with STDLIB_VERSION 4 on the way. The brochure covers most of the projected features too. Those which are not part of STDLIB_VERSION 3 are marked with (*).

## “Hello world!”
Let’s start with a familiar example:

```scala
func say() = {
  "Hello world!"
}
```

Functions in Ride are declared with `func` (see further below). Functions do have return types, this is inferred automatically by the compiler, so you don't have to declare them. In the case above the function say returns the string `Hello World!`. There is no `return` statement in the language because Ride is expression-based (everything is an expression), and the last statement is a result of the function.


## Blockchain

Ride was created specifically for execution within a blockchain environment and is optimised for this purpose. Because the blockchain is a shared ledger, located on many computers all around the world, it works a little differently to conventional programming languages.

Since Ride is designed to be used inside the blockchain, there is no way to access the filesystem or display anything in the console. Instead, Ride functions can read data from the blockchain and return actions as a result, which can then be applied to the blockchain.

## Comments

You can add comments to your code much as you can with other languages such as Python:

```scala
# This is a comment line

# And there is no multiline comments

"Hello world!" # You can write comments like here
```

## Directives

Every Ride script should start with directives for the compiler. At the time of publication, there are three types of directive, with different possible values.

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE DAPP #-}
{-# SCRIPT_TYPE ACCOUNT #-}
```

`STDLIB_VERSION` sets the version of the standard library. The latest version currently in production is 3.

`CONTENT_TYPE` sets the type of the file you're working on. There are different content types, `DAPP` and `EXPRESSION`. The `DAPP` type allows you to define functions and finish execution with certain transactions (changes to the blockchain), as well as using annotations. The `EXPRESSION` type should always return a boolean value, since it’s used as a predicate for transaction validation.


`SCRIPT_TYPE` sets the entity type we want to add to the script to change its default behavior. Ride scripts can be attached to either an `ACCOUNT` or `ASSET`.

Not all combinations of directives are correct. The example below won’t work, because `DAPP` content type is allowed only for accounts, while `EXPRESSION` type is allowed for assets and accounts.

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE DAPP #-}
{-# SCRIPT_TYPE ASSET #-} # dApp content type is not allowed for an asset
```

## Variables

Variables are declared and initialized with the `let` keyword. This is the only way to declare variables in Ride.


```scala
let a = "Bob"
let b = 1
```

All variables in Ride are immutable. This means you cannot change the value of a variable after declaration.

Ride is strongly typed and the variable's type is inferred from the value on the right hand side. 

Ride allows you to define variables globally, inside any function, or even inside a variable definition.

```scala
func lazyIsGood() = {
  let a = "Bob"
  let b = {
     let x = 1
     “Alice”
    }  
  true
}
```

The function above will compile and return true as a result, but variable `a` won't be initialized because Ride is lazy, meaning that any unused variables will not be calculated.

## Functions

Functions in Ride can only be used after they are declared.

```scala
func greet(name: String) = {
  "Hello, " + name
}

func add(a: Int, b: Int) = {
  func m(a:Int) = a
  m(a) + b
}
```
The type (`Int`, `String`, etc) comes after the argument’s name.

As in many other languages, functions should not be overloaded. It helps to keep the code simple, readable and maintainable.

```scala
func calc() = {
  42
}

func do() = { 
  let a = calc()
  true
}
```

The `callable` function will not be called either, because variable a is unused.

Unlike most languages, variable shadowing is not allowed. Declaring a variable with a name that is already used in a parent scope will result in a compilation error. 

Functions should be defined before they are used.

Functions can be invoked in prefix and postfix order:

```scala
let list = [1, 2, 3]
let a1 = list.size()
let a2 = size(list)

let b1 = getInteger(this, “key”)
let b2 = this.getInteger(“key”)
```
In these examples `a1` is the same as `a2` and `b1` is the same as `b2`. 


## Basic types

The main basic types and examples are listed below:

```scala
Boolean    #   true
String     #   "Hey"
Int        #   1610
ByteVector #   base58'...', base64'...', base16'...', fromBase58String("...") etc.
```
We will explore Strings and special types further below.

### Strings

```scala
let name = "Bob"   # use "double" quotes only
let coolName = name + " is cool!" # string concatenation by + sign

name.indexOf("o")  # 1
```

Like other data structures in Ride, strings are immutable. String data is encoded using UTF-8.

Only double quotes can be used to denote strings. Strings are immutable, just like all other types. This means that the `substring` function is very efficient: no copying is performed and no extra allocations are required.

All operators in Ride must have values of the same type on both sides. The following code will not compile because `age` is an `int`:

```scala
let age = 21
"Bob is " + age # won't compile
```

To make it work we have to convert `age` to `string`:

```scala
let age = 21
"Alice is " + age.toString() # will work!
```

## Special types

Ride has few core types, which operate much as they do in Scala.

### Unit

There is no `null` in Ride, as is the case in many other languages. Usually, built-in functions return unit value of type `unit` instead of `null`.

```scala
"String".indexOf("substring") == unit # true
```

### Nothing

Nothing is the 'bottom type' of Ride’s type system. No value can be of type Nothing, but an expression of type Nothing can be used everywhere. In functional languages, this is essential for support for throwing an exception:

```scala
2 + throw() # the expression compiles because
 	    # there's a defined function +(Int, Int).
 	    # The type of the second operand is Nothing, 
 	    # which complies to any required type.
```

### List

```scala
let list = [16, 10, 1997, "birthday"]       # can contain different data types

let second = list[1]                        # 10 - read second value from the list

```

`List` doesn't have any fields, but there are functions in the standard library that make it easier to work with fields.

```scala
let list = [16, 10, 1997, "birthday"]

let last = list[(list.size() - 1)] # "birthday", postfix call of size() function

let lastAgain = getElement(collection, size(collection) - 1) # the same as above
```

`.size()` function returns the length of a list. Note that it's a read-only value, and it cannot be modified by the user. (Note also that `last` could be of more than one type, but this is only inferred when the variable is set.)

```scala
let initList = [16, 10]                   # init value
let newList = cons(1997, initList)        # [1997, 16, 10]
let newList2 = 1997 :: initList           # [1997, 16, 10]
let newList2 = initList :+ 1              # [16, 10, 1](* Available in STDLIB_VERSION 4)
let newList2 = [4, 8, 15, 16] ++ [23, 42]     # [4 8 15 16 23 42](*)
```

- To prepend an element to an existing list, use the cons function or :: operator 
- To append an element, use the :+ operator (*)
- To concatenate 2 lists, use the ++ operator (*)


### Union types & Type Matching

```scala
let valueFromBlockchain = getString("3PHHD7dsVqBFnZfUuDPLwbayJiQudQJ9Ngf", "someKey") # Union(String | Unit)
```

Union types are a very convenient way to work with abstractions. `Union(String | Unit)` shows that the value is an intersection of these types.

The simplest example of `Union` types is given below (please bear in mind that defining custom user types in dApp code will be supported in future versions):

```scala
type Human : { firstName: String, lastName: String, age: Int}
type Cat : {name: String, age: Int }
```

`Union(Human | Cat)` is an object with one field, `age`, but we can use pattern matching:
```scala
Human | Cat => { age: Int }
```
Pattern matching is designed to check a value against value type:
```scala
  let t = ...               # Cat | Human
  t.age                     # OK
  t.name                    # Compiler error
  let name = match t {      # OK
    case h: Human => h.firstName
    case c: Cat   => c.name
  }
```

Type matching is a mechanism for:

```scala
let amount = match tx {              # tx is a current outgoing transaction
  case t: TransferTransaction => t.amount
  case m: MassTransferTransaction => m.totalAmount
  case _ => 0
}
```

The code above shows an example of type matching. There are different types of transactions in Waves, and depending on the type, the real amount of transferred tokens can be stored in different fields. If a transaction is `TransferTransaction` or `MassTransferTransaction` we use the corresponding field, while in all other cases, we will get 0.

## State reader functions

```scala
let readOrZero = match getInteger(this, "someKey") { # reading data from state
    case a:Int => a
    case _ => 0
}

readOrZero + 1

```

`getString` returns `Union(String | Unit)` because while reading data from the blockchain (the key-value state of accounts) some key-value pairs may not exist.


```scala
let v = getInteger("3PHHD7dsVqBFnZfUuDPLwbayJiQudQJ9Ngf", "someKey")
v + 1    # doesn’t compile, forcing a developer to foresee the possibility of non-existing value for the key

v.valueOrErrorMessage(“oops”) +  1 # compiles and executes

let realStringValue2 = getStringValue(this, "someKey")

```

To get the real type and value from Union use the `extract` function, which will terminate the script in case of `Unit` value. Another option is to use specialized functions like `getStringValue`, `getIntegerValue`, etc.


## If

```scala
let amount = 1610
if (amount > 42) then "I claim that amount is bigger than 42"
  else if (amount > 100500) then "Too big!"
  else "I claim something else"
```

`if` statements are pretty straightforward and similar to most other languages, with an important difference from some: `if` is an expression, so it must have an `else` clause (the result is assignable to a variable).

```scala
let a = 16
let result = if (a > 0) then a / 10 else 0 #
```

## Exceptions

```scala
throw("Here is exception text")
```

The `throw` function will terminate script execution immediately, with the provided text. There is no way to catch thrown exceptions.

The idea of `throw` is to stop execution and send useful feedback to the user.


```scala
let a = 12
if (a != 100) then
  throw ("a is not 100, actual value is " + a.toString())
  else throw("A is 100")
```

## Predefined data structures

\#**LET THE HOLY WAR BEGIN**

Ride has many predefined data structures specific to the Waves blockchain, such as: `Address`, `Alias`, `DataEntry`, `ScriptResult`, `Invocation`, `ScriptTransfer`, `TransferSet`, `WriteSet`, `AssetInfo`, `BlockInfo`.

```scala
let keyValuePair = DataEntry("someKey", "someStringValue")
```

For example, `DataEntry` is a data structure which describes a key-value pair, e.g. for account storage.

```scala
let transferSet = TransferSet([ScriptTransfer("3P23fi1qfVw6RVDn4CH2a5nNouEtWNQ4THs", amount, unit)])
```
All data structures can be used for type checking, pattern matching and their constructors as well.

## Loops with FOLD<N>

Since Ride’s virtual machine doesn’t have any concept of loops, they are implemented at compiler level via the FOLD<N> macro. The macro behaves like the ‘fold’ function in other programming languages, taking the input arguments: collection for iteration, starting values of the accumulator and folding function.

The important aspect is N - the maximum amount of interactions over collections. This is necessary for maintaining predictable computation costs.

This code sums the numbers of the array:

```scala
let a = [1, 2, 3, 4, 5]
func foldFunc(acc: Int, e: Int) = acc + e
FOLD<5>(a, 0, foldFunc) # returns 15
```

`FOLD<N>` can also be used for filtering, mapping, and other operations. Here’s an example for map with reverse:

```scala
let a = [1, 2, 3, 4, 5]
func foldFunc(acc: List[Int], e: Int) = (e + 1) :: acc
FOLD<5>(a, [], foldFunc) # returns [6, 5, 4, 3, 2]
```

## Annotations

Functions can be without annotations, or with `@Callable` or `@Verifier` annotations.

```scala
func getPayment(i: Invocation) = {
  let pmt = i.payment.valueOrErrorMessage(“Payment must be attached”)
  if (isDefined(pmt.assetId)) then 
    throw("This function accepts waves tokens only")
  else
  	pmt.amount
}

@Callable(i)
func pay() = {
  let amount = getPayment(i)
  WriteSet([DataEntry(i.caller.bytes, amount)])
}
```

Annotations can bind some values to the function. In the example above, variable `i` was bound to the function `pay` and stored all the information about the fact of invocation (the caller’s public key, address, payment attached to the transaction, fee, transactionId etc.).

Functions without annotations are not available from the outside. You can call them only inside other functions.

```scala
@Verifier(tx)
func verifier() = {
  match tx {
    case m: TransferTransaction => tx.amount <= 100 # can send up to 100 tokens
    case _ => false
  }
}
```

### @Verifier annotation

```scala
@Verifier(tx)
func verifier() = {
  match tx {
    case m: TransferTransaction => tx.amount <= 100 # can send up to 100 tokens
    case _ => false
  }
}
```

A function with the `@Verifier` annotation sets the rules for outgoing transactions of a decentralized application (dApp). Verifier functions cannot be called from the outside, but they are executed every time an attempt is made to send a transaction from a dApp.

Verifier functions should always return a `Boolean` value as a result, depending on which a transaction will be recorded to the blockchain or not.

Expression scripts (with directive `{-# CONTENT_TYPE EXPRESSION #-}`) along with functions annotated by @Verifier should always return a boolean value. Depending on that value the transaction will be accepted (in case of `true`) or rejected (in case of `false`) by the blockchain.


```scala
@Verifier(tx)
func verifier() = {
  sigVerify(tx.bodyBytes, tx.proofs[0], tx.senderPublicKey)

}
```

The Verifier function binds variable `tx`, which is an object with all fields of the current outgoing transaction.

A maximum of one `@Verifier()` function can be defined in each dApp script.

### @Callable annotation

Functions with the `@Callable` annotation can be called (or invoked) from outside of the blockchain. To call a callable function you have to send `InvokeScriptTransaction`.

```scala
@Callable(i)
func giveAway(age: Int) = {
  ScriptResult(
    WriteSet([DataEntry("age", age)]),
    TransferSet([ScriptTransfer(i.caller, age, unit)])
  )
}
```

Every caller of `giveAway` function will receive as many WAVES as his age and the dApp will store information about the fact of the transfer in its state.


#### Actions

Initial Actions are DataEntry, which allows for writing data as a key-value pair, and ScriptTransfer, a transfer of tokens from dApp to addressee. Other actions such as Issue/Reissue/Burn are designed to support native token operations as well as the family of Leasing operations(Available in STDLIB_VERSION 4).

A list of DataEntry structures in `WriteSet` will set or update key-value pairs in the storage of an account, while a list of ScriptTransfer structures in `TransferSet` will move tokens from the dApp account to other accounts.


```scala
@Callable(i)
func callMePlease(age: Int) = {
  TransferSet([ScriptTransfer(i.caller, age, unit)])
}
```

In STDLIB_VERSION 3, `@Callable` functions can return one of the following structures: `ScriptResult`, `WriteSet`, `TransferSet`.

`WriteSet` can contain up to 100 `DataEntry`, while `TransferSet` can contain up to 10 `ScriptTransfer`.

## Account vs Asset scripts
```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE EXPRESSION #-}
{-# SCRIPT_TYPE ACCOUNT #-}

let a = this # Address of the current account
a == Address(base58'3P9DEDP5VbyXQyKtXDUt2crRPn5B7gs6ujc') # true if script is running on the account with defined address
```
Ride scripts on the Waves blockchain can be attached to accounts and assets (`{-# SCRIPT_TYPE ACCOUNT #-}` defines it) and depending on the `SCRIPT_TYPE` keyword this can refer to different entities. For `ACCOUNT` script types this is an `Address` type.

For `ASSET` script type this will have `AssetInfo` type.

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE EXPRESSION #-}
{-# SCRIPT_TYPE ASSET #-}
let a = this # AssetInfo of the current asset
a.assetId == AssetInfo(base58'3P9DEDP5VbyXQyKtXDUt2crRPn5B7gs6ujc').assetId # true if script is running for the asset with defined assetId
```


## Testing and tools

You can try out Ride in REPL both online at [https://ide.wavesplatform.com/](https://ide.wavesplatform.com/) and on desktop via terminal with `surfboard`:

```scala
> npm i -g @waves/surfboard
> surfboard repl
```

For further development, the following tools and utilities are useful:

- Visual Studio Code plugin: waves-ride
- The `surfboard` tool will allow you to REPL and run tests on your existing node: [https://github.com/wavesplatform/surfboard]
- You should also install the Waves Keeper browser extension: [https://wavesplatform.com/products-keeper](https://wavesplatform.com/products-keeper)
- Online IDE with examples: [https://ide.wavesplatform.com/](https://ide.wavesplatform.com/)

Further help and information about tools can be found here: [https://wavesplatform.com/developers](https://wavesplatform.com/developers)


## Enjoy the Ride!


Hopefully this brochure will have given you a good introduction to Ride: a straightforward, secure, powerful programming language for smart contracts and dApps on the Waves blockchain. 

You should now be able to write your own smart contracts, and have all the tools you need to test them before deploying them to the Waves blockchain.

If you need help learning the basics of the Ride language, you can take the “Mastering Web3 with Waves” course: [https://stepik.org/course/54415/syllabus](https://stepik.org/course/54415/syllabus). 
Waves also runs developer workshops and hackathons in different locations around the world – check out our community page to stay up to date: [https://wavescommunity.com](https://wavescommunity.com)

We hope to meet you online or offline soon!

