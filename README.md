## Introduction

RIDE is a statically typed lazy functional expression-based compiled programming language designed for building bug-free and developer-friendly decentralised applications. 

RIDE is not Turing Complete and there are no loops, recursions and there are a lot of limitations by design - it helps to keep it simple and straightforward.

Despite being simple, it gives a lot of power to the developer. 

It's similar to Scala and is also influenced by F# and functional paradigm.

RIDE is a very simple language. Going through this documentation will take you about an hour, and by the end of it, you will learn pretty much the entire language.

## Hello world

```scala
func say() = {
  "Hello world!"
}
```

Functions are declared with `func`. Functions do have return types, but they are inferred automatically by the compiler, so you don't have to declare it. In the case above the function `say` returns `"Hello World!"` string. There is no `return` statement in the language because RIDE is expression-based (everything is an expression) and the last statement is a result of the function.

## Blockchain

RIDE is designed to be used inside the blockchain and there is no way to access to filesystem or print something in the console.

RIDE functions can read data from the blockchain and return transactions as a result, which would be applied to the blockchain.

## Comments

```scala
# This is a comment line

# And there is no multiline comments

"Hello world!" # You can write comments like here
```

## Directives

Every RIDE script should start with directives for the compiler. There are 3 possible types of directives with different possible values. 

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE DAPP #-}
{-# SCRIPT_TYPE ACCOUNT #-}
```

`STDLIB_VERSION` sets the version of the standard library. The latest version in production is 3.

`CONTENT_TYPE` sets the type of the file you're working on. There are different content types - `DAPP` and `EXPRESSION`. `DAPP` type allows to define functions and finish the execution with some transactions (changes in the blockchain) and use annotations, while `EXPRESSION`  should always finish with a boolean value.

`SCRIPT_TYPE` sets the entity type we want to add the script and change the default behavior. RIDE scripts can be attached to `ACCOUNT` or `ASSET`.

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE DAPP #-}
{-# SCRIPT_TYPE ASSET #-} # dApp content type is not allowed for an asset
```

Not all combinations of directives are correct. The example above won't work, because `DAPP` content type is allowed only for accounts, while `EXPRESSION` type is allowed for assets and accounts.

## Functions

```scala
func greet(name: String) = {
  "Hello, " + name
}

func add(a: Int, b: Int) = {
  a + b
}
```

The type comes after the argument's name. 

Like in many other languages functions cannot be overloaded. It helps to keep the code simple, readable and maintainable.

Functions can be used only after their declaration.

## Variables

```scala
let a = "Bob"
let b = 1
```

Variables are declared and initialized with `let` keyword.  This is the only way to declare variables in RIDE. All variables in RIDE are immutable. It means that you cannot change the value of the variable after declaration.

The variable's type is inferred from the value on the right hand side.

RIDE allows defining variables inside any function or in the global scope.

```scala
a = "Alice"
```

The code above will not compile, because variable `a` is not defined. All variables need to be declared in RIDE.

```scala
func lazyIsGood() = {
  let a = "Bob"
  true
}
```

The function above will compile and return `true` as a result, but variable a won't be initialized, because RIDE is lazy, which means that all unused variables will not be calculated.

```scala
func callable() = {
  42
}

func caller() = { 
  let a = callable()
  true
}
```

The `callable` function will not be called either, because unused variable `a` is unused.

Unlike most languages, variable shadowing is not allowed. Declaring a variable with a name that is already used in a parent scope will result in a compilation error.

## Basic types

The main basic types are listed below:

```scala
Boolean    #   true
String     #   "Hey"
Int        #   1610
ByteVector #   base58'...', base64'...', base16'...', fromBase58String("...") etc.
```

### Strings

```scala
let name = "Bob"
name + " is cool!" # string concatenation by + sign

name.indexOf("o")  # 1
```

In RIDE, a string is a read-only array of bytes. String data is encoded using UTF-8.

Only double quotes can be used to denote strings. Strings are immutable as all other types. This means that the substring function is very efficient: no copying is performed, no extra allocations required.

All operators in RIDE must have values of the same type on both sides. This code will not compile because `age` is an `int`:

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

```scala
List    # [16, 10, "hello"]
Nothing # 
Unit    # unit
```

RIDE has few types which "**look** like  ~~a duck~~ in Scala, swim like ~~a duck~~ in Scala and quack like ~~a duck~~ in Scala". 

There is no `null` type RIDE like in many other languages. Usually, built-in functions return Unit type instead of `null`.

```scala
"String".indexOf("substring") == unit # true
```

### List

```scala
let list = [16, 10, 1997, "birthday"]       # collection can contain different data types
let second = list[1]                        # 10 - read second value from the list

```

To work properly with lists in RIDE they should always be with known size because there are no loops and recursions.

`List` doesn't have any fields, but there are functions in the standard library which allow working with them easier

```scala
let list = [16, 10, 1997, "birthday"]

let last = list.getElement(list.size() - 1) # "birthday", postfix call of size() function

let lastAgain = getElement(collection, size(collection) - 1) # the same as above
```

`.size()` function returns the length of a list. Note, that it's a read-only value, and it can't be modified by the user.

```scala
let initList = [16, 10]            # init value
let newList = cons(1997, initList) # prepend a new element to the list - [1997, 16, 10]
```

You can prepend a new element to an existing list using `cons` function. There is no way to concatenate two lists or prepend multiple values to a list.

### Union types

```scala
let valueFromBlockchain = getString("3PHHD7dsVqBFnZfUuDPLwbayJiQudQJ9Ngf", "someKey") # Union(String | Unit)
```

Union types are a very convenient way to work with abstractions, `Union(String | Unit)` shows that the value is an intersection of these types.

The simplest intuition about `Union`types is below: 
```scala
type Human : { firstName: String, lastName: String, age: Int}	
type Cat : {name: String, age: Int } 
```

`Unioin(Human | Cat)` is an object with one field `age`:
```scala
Human | Cat => { age: Int }
```

`getString` returns `Union(String | Unit)` because while reading data from the blockchain (key-value state of accounts) some key-value pairs may not exist.

```scala
let valueFromBlockchain = getString("3PHHD7dsVqBFnZfUuDPLwbayJiQudQJ9Ngf", "someKey")
let realStringValue = valueFromBlockchain.extract()

# or
let realStringValue2 = getStringValue(this, "someKey")
```

To get the real type and value from Union use `extract` function, which will terminate the script in case of `Unit` value. Another option is to use specialized functions like `getStringValue`, `getIntegerValue` etc.

## If

```scala
let amount = 1610
if (amount > 42) then "I claim that amount is bigger than 42"
else if (amount > 100500) then "Too big!"
else "I claim something else"
```

`if` statements are pretty straightforward and similar to most other languages, except two differences: `if` can be used as an expression (result is assignable to a variable) and `else` branch is always required.

```scala
let a = 16
let result = if (a > 0) then a / 10 else 0 # 
```

## Pattern matching

```scala
let readOrInit = match getInteger(this, "someKey") {
    case a:Int => a
    case _ => 0
}
```

Pattern matching is a mechanism for checking a value against a pattern. RIDE allows to use pattern matching only for predefined types. 

// : fix the definition below

Pattern matching in RIDE looks like in Scala, but the only use case right now is getting the real type of `Union` typed variable. Pattern matching can be useful in cases of very complicated types like `Union(Order | ReissueTransaction | BurnTransaction | MassTransferTransaction | ExchangeTransaction | TransferTransaction | SetAssetScriptTransaction | InvokeScriptTransaction | IssueTransaction | LeaseTransaction | LeaseCancelTransaction | CreateAliasTransaction | SetScriptTransaction | SponsorFeeTransaction | DataTransaction)`.

```scala
let amount = match tx { # tx is a current outgoing transaction object in the global scope
  case t: TransferTransaction => t.amount
  case m: MassTransferTransaction => m.totalAmount
  case _ => 0
}
```

The code above shows an example of pattern matching using. There are different types of transactions in Waves blockchain and depending on the type the real amount of transferred tokens can be stored in different fields. If a transaction is `TransferTransaction` or ` MassTransferTransaction` we will take the proper field, in all other cases, we will get 0.

## Pure functions

\#**LET THE HOLY WAR BEGIN**

RIDE functions are pure by default, meaning that their return values are only determined by their arguments, and their evaluation has no side effects.

This is achieved by the lack of global variables and all function arguments being immutable by default.

RIDE is not a pure functional language however, because there is `throw()` function which terminates script execution at any point. 

```scala
let a = getInteger(this, "key").extract()
throw("I will terminate it!")
if a < 0 then 
	"a is negative" 
else 
	"a is positive or 0"
```

In the example above the script will terminate on line 2 with message `I will terminate it!` and never reach the `if` statement.

## Annotations / Access modifiers

Functions can be defined only in a script with `{-# CONTENT_TYPE DAPP #-}` declaration. Functions can be without annotations, with `@Callable` or `@Verifier` annotations.

```scala
func getPayment(i: Invocation) = {
  let pmt = extract(i.payment)
  if (isDefined(pmt.assetId)) then 
    throw("This function accetps waves tokens only")
  else
  	pmt.amount
}

@Callable(i)
func pay() = {
  let amount = getPayment(i)
  WriteSet([DataEntry(i.caller.bytes, amount)])
}
```

Functions with `@Callable` annotation can be called (or invoked) from the outside of the blockchain. To call the callable functions you have to send `InvokeScriptTransaction`

Annotations can bind some values to the function. In the example above variable `i` was bound to the function `pay` and stored all the information about the fact of invocation (callers' public key, address, payment attached to the transaction, fee, transactionId etc.).

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

Function with `@Verifier` annotation sets the rules for outgoing transactions of a decentralized application (dApp). Verifier functions can't be called from the outside, but they are executed every time an attempt is made to send a transaction from a dApp.

Verifier functions should always return `Boolean` value as a result, depending on it transaction will go the blockchain or not. 

Verifier function binds variable `tx` which is an object with all fields of the current outgoing transaction.

Only one verifier function can be defined in one script.

```scala
@Callable(i)
func callMeMaybe() = {
  let randomValue = getRandomValue()
  WriteSet([DataEntry("key", randomValue)])
}

func getRandomValue() = {
  16101997 # random enough
}
```

This won't compile, because functions **without** annotations should be defined **before** functions with annotations.

### Predefined data structures

RIDE has a lot of predefined specific for Waves Blockchain data structures like: `Address`, `Alias`, `DataEntry`, `ScriptResult`, `Invocation`, `ScriptTransfer`, `TransferSet`, `WriteSet`, `AssetInfo`, `BlockInfo`.

```scala
let keyValuePair = DataEntry("someKey", "someStringValue")
```

`DataEntry` is a data structure which describes a key-value pair like in account storage. 

```scala
let transferSet = TransferSet([ScriptTransfer("3P23fi1qfVw6RVDn4CH2a5nNouEtWNQ4THs", amount, unit)])
```



All data structures can be used for type checking, pattern matching and they constructors as well.

## Execution Results

```scala
@Verifier(tx)
func verifier() = {
  "Returning some string"
}
```

Expression scripts (with directive `{-# CONTENT_TYPE EXPRESSION #-}`) along with functions annotated by `@Verifier` should always return boolean value. Depending on that value transaction will be accepted (in case of `true`) or rejected (in case of `false`) by the blockchain. 

```scala
@Callable(i)
func giveAway(age: Int) = {
  ScriptResult(
    WriteSet([DataEntry("age", age)]),
    TransferSet([ScriptTransfer(i.caller, age, unit)])
  )
}
```

Every caller of `giveAway` function will get as many Waves as his age and dApp will store information about the fact of transfer in its' state.

`@Callable` functions can finish with two types of changes of the blockchain - state changes and tokens transfer. 

List of `DataEntry` structures in `WriteSet`  will set or update key-value pairs in the storage of an account, while list of `ScriptTransfer` structures in `TransferSet` will move tokens from dApp account to other accounts.

```scala
@Callable(i)
func callMePlease(age: Int) = {
  TransferSet([ScriptTransfer(i.caller, age, unit)])
}
```

`@Callable` functions can return one of the next structures: `ScriptResult`, `WriteSet`, `TransferSet`. 

`WriteSet` can contain up to 100 `DataEntry`, `TransferSet` can contain up to 10 `ScriptTransfer`. 

## Exceptions

```scala
throw("Here is exception text")
```

`throw` function will terminate the script execution immediately with the provided text. There are no ways to catch thrown exceptions. 

The main idea of `throw` is to stop an execution and send informative feedback to the user. 

```scala
let a = 12
if (a != 100) then throw ("a is not 100, actual value is " + a.toString())
else throw("A is 100")
```

`throw` function can be used for debug purposes while developing dApps, because there are no debuggers for RIDE yet.

## Execution context

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE EXPRESSION #-}
{-# SCRIPT_TYPE ACCOUNT #-}

let a = this # Address of the current account
a == Address(base58'3P9DEDP5VbyXQyKtXDUt2crRPn5B7gs6ujc') # true if script is running on the account with defined address
```

RIDE scripts in Waves blockchain can be attached to accounts and assets (`{-# SCRIPT_TYPE ACCOUNT #-}` defines it) and depending  on the `SCRIPT_TYPE` keyword `this` can refer to the different entities. For `ACCOUNT` script types `this` is an `Address` type.

For `ASSET` script type `this` will have `AssetInfo` type.

```scala
{-# STDLIB_VERSION 3 #-}
{-# CONTENT_TYPE EXPRESSION #-}
{-# SCRIPT_TYPE ACCOUNT #-}

let a = this # Address of the current account
a == Address(base58'3P9DEDP5VbyXQyKtXDUt2crRPn5B7gs6ujc') # true if script is running on the account with defined address
```

## Testing

You cannot test RIDE code using RIDE, but there is JavaScript/Typescript [tool](https://wavesplatform.github.io/js-test-env/) to write integration tests in the [online IDE](https://ide.wavesplatform.com). 

## Toolkit

All needed tools like IDE support, custom blockchain networks, explorer, CLI tools etc. are described in [this great article.](https://blog.wavesplatform.com/how-to-build-deploy-and-test-a-waves-ride-dapp-785311f58c2)

More interesting frameworks and tools you can find in the [awesome list](https://github.com/msmolyakov/awesome-waves).

## MORE 

More details about standard library functions, limitations, complexity and examples you can find in [the documentation](https://docs.wavesplatform.com/en/ride/about-ride.html).

Examples of RIDE scripts are in the [ride-examples](https://github.com/wavesplatform/ride-examples) repository.

**The last, but not the least. If you want to see real-world application with dApp as a backend - welcome to [Mastering Web3 with Waves Course](https://stepik.org/course/54415/syllabus).**

