# Coding Guidelines

## Introduction

### 1.1. What is this
Coding guidelines we should ideally follow. You are always free to challenge these decisions and should do so. The importance of the application of certain rules is always dependent on the situation. If you find yourself stuck, chat with other developers in your team and try and reach the most rational solution.

Use the basic principles outlined in this guide to help you make that decision.

### 1.2. Purpose of this document
Some notes on guidelines as per the original document (see: https://github.com/dennisdoomen/CSharpGuidelines).

Not every developer:

- is aware that code is generally read 10 times more than it is changed;
- is aware of the potential pitfalls of certain constructions in C#;
- is up to speed on certain conventions when using the .NET Framework such as `IDisposable` or the deferred execution nature of LINQ;
- is aware of the impact of using (or neglecting to use) particular solutions on aspects like security, performance, multi-language support, etc;
- realizes that not every developer is as capable, skilled or experienced to understand elegant, but potentially very abstract solutions;

### 1.3. Basic principles
- The Principle of Least Surprise (or Astonishment): you should choose a solution that everyone can understand, and that keeps them on the right track.
- Keep It Simple Stupid (a.k.a. KISS): the simplest solution is more than sufficient.
- You Ain't Gonna Need It (a.k.a. YAGNI): create a solution for the problem at hand, not for the ones you think may happen later on. Can you predict the future?
- Don't Repeat Yourself (a.k.a. DRY): avoid duplication within a component, a source control repository or a [bounded context](http://martinfowler.com/bliki/BoundedContext.html), without forgetting the [Rule of Three](http://lostechies.com/derickbailey/2012/10/31/abstraction-the-rule-of-three/) heuristic.

Regardless of the elegance of someone's solution, if it's too complex for the ordinary developer, exposes unusual behavior, or tries to solve many possible future issues, it is very likely the wrong solution and needs redesign.

**The worst response a developer can give you to these principles is: "But it works?".**

### 1.4. Things to note
- Give the original document a read. It might be worth comparing with what we are following here.
- **Discuss and expand on this document.** Nothing is set in stone. 
- Make sure there are always a few hard copies of the [Cheat Sheet]() close at hand. 
- Include the most critical coding guidelines on your [Project Checklist]() and verify the remainder as part of your [Code Review]().

### 1.5 Adding new rules
If you add a new rule to this guideline, please consider using the following template:

``` markdown
	### <rule-name> 
	[optionally add :white_circle:, :large_large_blue_circle: or :red_circle: - least-to-most strict from white->red]

	<description>

	**Bad**
	```
	codeblock
	```

	**Good**
	```
	codeblock
	```

	**Tip** <tip-to-avoid-doing-this>
	**Note** <additional-info>
	**Exception** <exception-to-the-rule>
```

## Class Design

### A class or interface should have a single purpose :red_circle:
A class or interface should have a single purpose within the system it functions in. In general, a class either represents a primitive type like an email or ISBN number, an abstraction of some business concept, a plain data structure, or is responsible for orchestrating the interaction between other classes. It is never a combination of those. This rule is widely known as the [Single Responsibility Principle](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html), one of the S.O.L.I.D. principles.

**Tip:** A class with the word `And` in it is an obvious violation of this rule.

**Tip:** Use [Design Patterns](http://en.wikipedia.org/wiki/Design_pattern_(computer_science)) to communicate the intent of a class. If you can't assign a single design pattern to a class, chances are that it is doing more than one thing.

**Note** If you create a class representing a primitive type you can greatly simplify its use by making it immutable.

### Only create a constructor that returns a useful object :white_circle:
There should be no need to set additional properties before the object can be used for whatever purpose it was designed. However, if your constructor needs more than three parameters [LINK TO RULE](/), your class might have too much responsibility.

**Note** EF core also supports [entity types with constructors](https://docs.microsoft.com/en-gb/ef/core/modeling/constructors)

### An interface should be small and focused :large_blue_circle:
Interfaces should have a name that clearly explains their purpose or role in the system. Do not combine many vaguely related members on the same interface just because they were all on the same class. Separate the members based on the responsibility of those members, so that callers only need to call or implement the interface related to a particular task. This rule is more commonly known as the [Interface Segregation Principle](https://lostechies.com/wp-content/uploads/2011/03/pablos_solid_ebook.pdf).

<!-- ### Use an interface rather than a base class to support multiple implementations :white_circle:

If you want to expose an extension point from your class, expose it as an interface rather than as a base class. You don't want to force users of that extension point to derive their implementations from a base class that might have an undesired behavior. However, for their convenience you may implement a(n abstract) default implementation that can serve as a starting point. -->

### Use an interface to decouple classes from each other :large_blue_circle:
Interfaces are a very effective mechanism for decoupling classes from each other:

- They can prevent bidirectional associations; 
- They simplify the replacement of one implementation with another; 
- They allow the replacement of an expensive external service or resource with a temporary stub for use in a non-production environment.
- They allow the replacement of the actual implementation with a dummy implementation or a fake object in a unit test; 
- Using a dependency injection framework you can centralize the choice of which class is used whenever a specific interface is requested.

### Avoid static classes :large_blue_circle:
With the exception of extension method containers, static classes very often lead to badly designed code. They are also very difficult, if not impossible, to test in isolation, unless you're willing to use some very hacky tools.

**Note:** If you really need that static class, mark it as static so that the compiler can prevent instance members and instantiating your class. This relieves you of creating an explicit private constructor.

<!-- Hiding this - people shouldn't even think about it

### Don't suppress compiler warnings using the `new` keyword

Compiler warning [CS0114](https://docs.microsoft.com/en-us/dotnet/csharp/misc/cs0114) is issued when breaking [Polymorphism](http://en.wikipedia.org/wiki/Polymorphism_in_object-oriented_programming), one of the most essential object-orientation principles.
The warning goes away when you add the `new` keyword, but it keeps sub-classes difficult to understand. Consider the following two classes:

	public class Book  
	{
		public virtual void Print()  
		{
			Console.WriteLine("Printing Book");
		}  
	}
	
	public class PocketBook : Book  
	{
		public new void Print()
		{
			Console.WriteLine("Printing PocketBook");
		}  
	}

This will cause behavior that you would not normally expect from class hierarchies:

	PocketBook pocketBook = new PocketBook();
	
	pocketBook.Print(); // Outputs "Printing PocketBook "
	
	((Book)pocketBook).Print(); // Outputs "Printing Book"

It should not make a difference whether you call `Print()` through a reference to the base class or through the derived class. -->

### It should be possible to treat a derived object as if it were a base class object :red_circle:
In other words, you should be able to use a reference to an object of a derived class wherever a reference to its base class object is used without knowing the specific derived class. A very notorious example of a violation of this rule is throwing a `NotImplementedException` when overriding some of the base-class methods. A less subtle example is not honoring the behavior expected by the base class.   
  
**Note:** This rule is also known as the Liskov Substitution Principle, one of the [S.O.L.I.D.](http://www.lostechies.com/blogs/chad_myers/archive/2008/03/07/pablo-s-topic-of-the-month-march-solid-principles.aspx) principles.

### Avoid exposing the other objects an object depends on :large_blue_circle:
If you find yourself writing code like this then you might be violating the [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter).

	repo.DbSet.DoThing();

An object should not expose any other classes it depends on because callers may misuse that exposed property or method to access the object behind it. By doing so, you allow calling code to become coupled to the class you are using, and thereby limiting the chance that you can easily replace it in a future stage.

### Avoid bidirectional dependencies :large_blue_circle:
This means that two classes know about each other's public members or rely on each other's internal behavior. Refactoring or replacing one of those classes requires changes on both parties and may involve a lot of unexpected work. The most obvious way of breaking that dependency is to introduce an interface for one of the classes and using Dependency Injection.

**Exception:** Domain models such as defined in [Domain-Driven Design](http://domaindrivendesign.org/) tend to occasionally involve bidirectional associations that model real-life associations. In those cases, make sure they are really necessary, and if they are, keep them in.

### Classes should have state and behavior :red_circle:
In general, if you find a lot of data-only classes in your code base, you probably also have a few classes with a lot of behavior. Use the principles of object-orientation explained in this section and move the logic close to the data it applies to.
ore complicated scenarios might break this principle and should be discussed.

**Bad**
``` c#
_userManager.GetFullName(user);
```

**Good**
``` c#
user.GetFullName;
```

**Exception:** The only exceptions to this rule are classes that are used to transfer data over a communication channel, also called [Data Transfer Objects](http://martinfowler.com/eaaCatalog/dataTransferObject.html), or a class that wraps several parameters of a method.

### Classes should protect the consistency of their internal state :large_blue_circle:
Validate incoming arguments from public members. For example:

	public void SetAge(int years)
	{
		AssertValueIsInRange(years, 0, 200, nameof(years));
		
		this.age = years;
	}

Protect invariants on internal state. For example:

	public void Render()
	{
		AssertNotDisposed();
		
		// ...
	}

## Member Design Guidelines

### Use a method instead of a property :red_circle:
:police_car: There might be some disagreement on this, so lets discuss in detail
- If the work is more expensive than setting a field value. 
- If it represents a conversion such as the `Object.ToString` method.
- If it returns a different result each time it is called, even if the arguments didn't change. For example, the `NewGuid` method returns a different value each time it is called.
- If the operation causes a side effect such as changing some internal state not directly related to the property (which violates the [Command Query Separation](http://martinfowler.com/bliki/CommandQuerySeparation.html) principle).

**Bad**
```
_mutator.SetLost(listing);
```

**Good**
```
listing.SetLost();
```

**Exception:** Populating an internal cache

### Don't use mutually exclusive properties :red_circle:
Having properties that cannot be used at the same time typically signals a type that represents two conflicting concepts. Even though those concepts may share some of their behavior and states, they obviously have different rules that do not cooperate.

This violation is often seen in domain models and introduces all kinds of conditional logic related to those conflicting rules, causing a ripple effect that significantly increases the maintenance burden.

**Bad**
```
_listing.IsLost = true;
_listing.IsWon = true;
```

### A property, method or local function should do only one thing :red_circle:
A method body should have a single responsibility.

### Don't expose stateful objects through static members :large_blue_circle:
A stateful object is an object that contains many properties and lots of behavior behind it. If you expose such an object through a static property or method of some other object, it will be very difficult to refactor or unit test a class that relies on such a stateful object.

A classic example of this is the `HttpContext.Current` property, part of ASP.NET. Many see the `HttpContext` class as a source of a lot of ugly code. In fact, the testing guideline [Isolate the Ugly Stuff](http://codebetter.com/jeremymiller/2005/10/21/haacked-on-tdd-and-jeremys-first-rule-of-tdd/) often refers to this class.

### Return an `IQueryable<T>` to clearly indicate when execution of a Linq-to-X query is deferred :red_circle:
:police_car: Should we rather always resolve the query before returning the result from a function?
When you are building a query and no execution has been triggered, explicitly keep the type to IQueryable to ensure the response of the function clearly indicates the possible side-effects of any execution againts it.

### Return an `IEnumerable<T>` or `ICollection<T>` instead of a concrete collection class :large_blue_circle:
You generally don't want callers to be able to change an internal collection, so don't return arrays, lists or other collection classes directly. Instead, return an `IEnumerable<T>`, or, if the caller must be able to determine the count, an `ICollection<T>`.

**Note:** You can also use `IReadOnlyCollection<T>`, `IReadOnlyList<T>` or `IReadOnlyDictionary<TKey, TValue>`.

**Exception:** Immutable collections such as `ImmutableArray<T>`, `ImmutableList<T>` and `ImmutableDictionary<TKey, TValue>` prevent modifications from the outside and are thus allowed.

### Properties, arguments and return values representing strings or collections should never be `null` :red_circle:
Returning `null` can be unexpected by the caller. Always return an empty collection or an empty string instead of a `null` reference. This also prevents cluttering your code base with additional checks for `null`, or even worse, `string.IsNullOrEmpty()`.

### Define parameters as specific as possible :red_circle:
If your method or local function needs a specific piece of data, define parameters as specific as that and don't take a container object instead. For instance, consider a method that needs a connection string that is exposed through a central `IConfiguration` interface. Rather than taking a dependency on the entire configuration, just define a parameter for the connection string. This not only prevents unnecessary coupling, it also improves maintainability in the long run.

**Note:** An easy trick to remember this guideline is the *Don't ship the truck if you only need a package*.

### Consider using domain-specific value types rather than primitives :large_blue_circle:
Instead of using strings, integers and decimals for representing domain-specific types such as an ISBN number, an email address or amount of money, consider creating dedicated value objects that wrap both the data and the validation rules that apply to it. By doing this, you prevent ending up having multiple implementations of the same business rules, which both improves maintainability and prevents bugs.

## Miscellaneous Design Guidelines

### Throw exceptions rather than returning some kind of status value :large_blue_circle:
:police_car: There might be some disagreement on this, so lets discuss in detail
A code base that uses return values to report success or failure tends to have nested if-statements sprinkled all over the code. Quite often, a caller forgets to check the return value anyway. Structured exception handling has been introduced to allow you to throw exceptions and catch or replace them at a higher layer. In most systems it is quite common to throw exceptions whenever an unexpected situation occurs.

### Provide a rich and meaningful exception message text or custom Exception class :large_blue_circle:
The message should explain the cause of the exception, and clearly describe what needs to be done to avoid the exception.

### Throw the most specific exception that is appropriate :large_blue_circle:
For example, if a method receives a `null` argument, it should throw `ArgumentNullException` instead of its base type `ArgumentException`. Throw your own custom Exception if nothing appropriate exists

```
throw new SalesforceSessionExpiredException();
```

### Don't catching generic exceptions :red_circle:
Avoid swallowing errors by catching non-specific exceptions, such as `Exception`, `SystemException`, and so on, in application code. Only in top-level code, such as a last-chance exception handler, you should catch a non-specific exception for logging purposes and a graceful shutdown of the application.

### Properly handle exceptions in asynchronous code :red_circle:
When throwing or handling exceptions in code that uses `async`/`await` or a `Task` remember the following two rules:

- Exceptions that occur within an `async`/`await` block and inside a `Task`'s action are propagated to the awaiter.
- Exceptions that occur in the code preceding the asynchronous block are propagated to the caller.

### Always call `ConfigureAwait(false)` on async functions :white_circle:
This avoids deadlocks but ensuring that continuation after await does not have to run in the caller context. See [this article](https://medium.com/bynder-tech/c-why-you-should-use-configureawait-false-in-your-library-code-d7837dce3d7f)

### Prefer `async` and `await` when creating methods :red_circle:
Using the `async` frees up resources on the current thread for other processes and should be preferred.

### Only use `async` for low-intensive long-running activities :red_circle:
The usage of `async` won't automagically run something on a worker thread like `Task.Run` does. It just adds the necessary logic to allow releasing the current thread, and marshal the result back on that same thread if a long-running asynchronous operation has completed. In other words, use `async` only for I/O bound operations.

### Use generic constraints if applicable :red_circle:
Instead of casting to and from the object type in generic types or methods, use `where` constraints or the `as` operator to specify the exact characteristics of the generic parameter.

**Bad**
``` c#
class MyClass  
{
	void SomeMethod(T t)  
	{  
		object temp = t;  
		SomeClass obj = (SomeClass) temp;  
	}  
}
```

**Good**	
``` c#  
class MyClass where T : SomeClass  
{
	void SomeMethod(T t)  
	{  
		SomeClass obj = t;  
	}  

}
```

## Maintainability Guidelines

### Methods should not exceed 7 statements :red_circle:
A method that requires more than 7 statements is simply doing too much or has too many responsibilities. It also requires the human mind to analyze the exact statements to understand what the code is doing. Break it down into multiple small and focused methods with self-explaining names, but make sure the high-level algorithm is still clear.

### Make all members `private` and types `internal sealed` by default :red_circle:
To make a more conscious decision on which members to make available to other classes, first restrict the scope as much as possible. Then carefully decide what to expose as a public member or type.

### Avoid conditions with double negatives :red_circle:
Although a property like `customer.HasNoOrders` makes sense, avoid using it in a negative condition like this:
``` c#
bool hasOrders = !customer.HasNoOrders;
```
Double negatives are more difficult to grasp than simple expressions, and people tend to read over the double negative easily.

### Name assemblies after their contained namespace :red_circle:
All DLLs should be named according to the pattern *Company*.*Component*.dll where *Company* refers to your company's name and *Component* contains one or more dot-separated clauses.

**Exception:** If you decide to combine classes from multiple unrelated namespaces into one assembly, consider suffixing the assembly name with `Core`, but do not use that suffix in the namespaces.

### Name a source file to the type it contains :large_blue_circle:
Use Pascal casing to name the file and don't use underscores. Don't include (the number of) generic type parameters in the file name.

### Limit the contents of a source code file to one type :red_circle:
**Exception:** Nested types should be part of the same file.

**Exception:** Types that only differ by their number of generic type parameters should be part of the same file.

### Avoid `partial` classes :large_blue_circle:
:police_car: Maybe we really don't and use extension classes in a case like this instead
**Exception:** Extending generated code

### Use `using` statements instead of fully qualified type names :white_circle:
Limit usage of fully qualified type names to prevent name clashing.

**Bad**
``` c#
var label = new System.Web.UI.WebControls.Label();
```

**Good**
``` c#
using Label = System.Web.UI.WebControls.Label;
```

### Don't use "magic" numbers :red_circle:
Don't use literal values, either numeric or strings, in your code, other than to define symbolic constants. For example:

**Good**
``` c#
public class Whatever  
{
	public static readonly Color PapayaWhip = new Color(0xFFEFD5);
	public const int MaxNumberOfWheels = 18;
	public const byte ReadCreateOverwriteMask = 0b0010_1100;
}
```

Strings intended for logging or tracing are exempt from this rule. Literals are allowed when their meaning is clear from the context, and not subject to future changes.

**Good**
``` c#
mean = (a + b) / 2;
WaitMilliseconds(waitTimeInSeconds * 1000);
```

**Note:** An enumeration can often be used for certain types of symbolic constants.

### Only use `var` when the type is very obvious :red_circle:
Only use `var` as the result of a LINQ query, or if the type is very obvious from the same statement and using it would improve readability. So don't

**Bad**
``` c#
var item = 3;                              // what type? int? uint? float?
var myfoo = MyFactoryMethod.Create("arg"); // Not obvious what base-class or interface to expect
```

**Good**
``` c#
var query = from order in orders where order.Items > 10 and order.TotalValue > 1000;
var repository = new RepositoryFactory.Get();	
var list = new ReadOnlyCollection();
```

In all of three above examples it is clear what type to expect. For a more detailed rationale about the advantages and disadvantages of using `var`, read Eric Lippert's [Uses and misuses of implicit typing](http://blogs.msdn.com/b/ericlippert/archive/2011/04/20/uses-and-misuses-of-implicit-typing.aspx).

### Declare and initialize variables as late as possible :large_blue_circle:
Avoid the C and Visual Basic styles where all variables have to be defined at the beginning of a block, but rather define and initialize each variable at the point where it is needed.

### Assign each variable in a separate statement :red_circle:
Don't use confusing constructs like:
``` c#
var result = someField = GetSomeMethod();
```

### Favor object and collection initializers over separate statements :large_blue_circle:

**Bad**
``` c#
var startInfo = new ProcessStartInfo("myapp.exe");	
startInfo.StandardOutput = Console.Output;
startInfo.UseShellExecute = true;

var countries = new List();
countries.Add("Netherlands");
countries.Add("United States");

var countryLookupTable = new Dictionary<string, string>();
countryLookupTable.Add("NL", "Netherlands");
countryLookupTable.Add("US", "United States");
```

**Good**
``` c#
var startInfo = new ProcessStartInfo("myapp.exe")  
{
	StandardOutput = Console.Output,
	UseShellExecute = true  
};

var countries = new List { "Netherlands", "United States" };

var countryLookupTable = new Dictionary<string, string>
{
	["NL"] = "Netherlands",
	["US"] = "United States"
};
```

[Object and Collection Initializers](http://msdn.microsoft.com/en-us/library/bb384062.aspx):

### Use named arguments when the argument provided does not give a clear indication of it's purpose :red_circle:

**Bad**
``` c#
someMethod.Execute(true);
```

**Good**
``` c#
someMethod.Execute(skipProcessed: true)
```

### Avoid using named arguments :red_circle:
If you need named arguments to improve the readability of the call to a method, that method is probably doing too much and should be refactored.

### Avoid signatures that take a `bool` parameter :large_blue_circle:
Consider the following method signature:

``` c#
public Customer CreateCustomer(bool platinumLevel) {}

Customer customer = CreateCustomer(true);
```

Often, a method taking such a bool is doing more than one thing and needs to be refactored into two or more methods. An alternative solution is to replace the bool with an enumeration.

### Don't make explicit comparisons to `true` or `false` :red_circle:
**Bad**
``` c#
while (condition == false)
while (condition != true)
while (((condition == true) == true) == true)
```

**Good**
``` c#
while (condition)
```

### Don't change a loop variable inside a `for` loop :large_blue_circle:
Updating the loop variable within the loop body is generally considered confusing, even more so if the loop variable is modified in more than one place.

**Bad**
``` c#
for (int index = 0; index < 10; ++index)
{  
	if (someCondition)
	{
		index = 11; // Use 'break' or 'continue' instead.
	}
}
```

### Avoid nested loops :red_circle:
A method that nests loops is more difficult to understand than one with only a single loop.

In most cases nested loops can be replaced with a much simpler LINQ query.

### Always add a block after the keywords `if`, `else`, `do`, `while`, `for`, `foreach` and `case` :red_circle:
:police_car: There is most definitely some disagreement around this

**Bad**
``` c#
if (isActive)
	if (isVisible)
		Foo();
	else
		Bar();
```

**Good**
``` c#
if (isActive)
{
	if (isVisible)
	{
		Foo();
	}
	else
	{
		Bar();
	}
}
```

### Always add a `default` block after the last `case` in a `switch` statement :red_circle:
Add a descriptive comment if the `default` block is supposed to be empty.

Moreover, if that block is not supposed to be reached throw an `InvalidOperationException` or `ArgumentOutOfRangeException` to detect future changes that may fall through the existing cases. 

**Good**
``` c#
void Foo(string answer)  
{  
	switch (answer)  
	{  
		case "no":  
		{
			Console.WriteLine("You answered with No");  
			break;
		}
		case "yes":
		{  
			Console.WriteLine("You answered with Yes");  
			break;
		}
		default:  
		{
			// Not supposed to end up here.  
			throw new InvalidOperationException("Unexpected answer " + answer);
		}  
	}  
}
```

### Finish every `if`-`else`-`if` statement with an `else` clause :white_circle:
:police_car: Not too picky about this one, but would like some opinions

**Good**
``` c#
void Foo(string answer)  
{  
	if (answer == "no")  
	{  
		Console.WriteLine("You answered with No");  
	}  
	else if (answer == "yes")  
	{  
		Console.WriteLine("You answered with Yes");  
	}  
	else  
	{  
		// What should happen when this point is reached? Ignored? If not,
		// throw an InvalidOperationException.  
	}  
}
```

###  Be reluctant with multiple `return` statements :white_circle:
One entry, one exit is a sound principle and keeps control flow readable. However, if the method body is very small then multiple return statements may actually improve readability

### Don't use an `if`-`else` construct instead of a simple (conditional) assignment :red_circle:

**Bad**
``` c#
bool isPositive;
if (value > 0)
{
	isPositive = true;
}
else
{
	isPositive = false;
}

string classification;
if (value > 0)
{
	classification = "positive";
}
else
{
	classification = "negative";
}

int result;
if (offset == null)
{
	result = -1;
}
else
{
	result = offset.Value;
}

string name;
if (employee.Manager != null)
{
	return employee.Manager.Name;
}
else
{
	return null;
}
```

**Good**
``` c#
bool isPositive = (value > 0);
bool classification (value > 0) ? "positive" : "negative";
int result = offset ?? -1;
string name = employee.Manager?.Name;
```

### Encapsulate complex expressions in a property, method or local function :red_circle:
Consider the below *good* and *bad* examples. In order to understand what the expression is about, you need to analyze its exact details and all of its possible outcomes. Obviously, you can add an explanatory comment on top of it, but it is much better to replace this complex expression with a clearly named method.

You still need to understand the expression if you are modifying it, but the calling code is now much easier to grasp.

**Bad**
``` c#
if (member.HidesBaseClassMember && (member.NodeType != NodeType.InstanceInitializer))
{
	// do something
}
```

**Good**
``` c#
if (NonConstructorMemberUsesNewKeyword(member))  
{  
	// do something
}

private bool NonConstructorMemberUsesNewKeyword(Member member)  
{  
	return
		(member.HidesBaseClassMember &&
		(member.NodeType != NodeType.InstanceInitializer)  
}
```

### Call the more overloaded method from other overloads :large_blue_circle:
This guideline only applies to overloads that are intended to provide optional arguments. Consider, for example, the following code snippet:

``` c#
public class MyString  
{
	private string someText;
	
	public int IndexOf(string phrase)  
	{  
		return IndexOf(phrase, 0); 
	}
	
	public int IndexOf(string phrase, int startIndex)  
	{  
		return IndexOf(phrase, startIndex, someText.Length - startIndex);
	}
	
	public virtual int IndexOf(string phrase, int startIndex, int count)  
	{  
		return someText.IndexOf(phrase, startIndex, count);
	}  
}
```

The class `MyString` provides three overloads for the `IndexOf` method, but two of them simply call the one with one more parameter. Notice that the same rule applies to class constructors; implement the most complete overload and call that one from the other overloads using the `this()` operator. Also notice that the parameters with the same name should appear in the same position in all overloads.

**Note:** If you also want to allow derived classes to override these methods, define the most complete overload as a non-private `virtual` method that is called by all overloads.

### Only use optional arguments to replace overloads :red_circle:
:police_car: Would like to relook at this one

Consider:
``` c#
public virtual int IndexOf(string phrase, int startIndex = 0, int count = -1)
{
	int length = (count == -1) ? (someText.Length - startIndex) : count;
	return someText.IndexOf(phrase, startIndex, length);
}
```

If the optional parameter is a reference type then it can only have a default value of `null`. But since strings, lists and collections should never be `null`, you must use overloaded methods instead.

**Note:** The default values of the optional parameters are stored at the caller side. As such, changing the default value without recompiling the calling code will not apply the new default value.

**Note:** When an interface method defines an optional parameter, its default value is discarded during overload resolution unless you call the concrete class through the interface reference. See [this post by Eric Lippert](http://blogs.msdn.com/b/ericlippert/archive/2011/05/09/optional-argument-corner-cases-part-one.aspx) for more details.

### Don't declare signatures with more than 3 parameters :large_blue_circle:
:police_car: I think this is open for discussion as well

To keep constructors, methods, delegates and local functions small and focused, do not use more than three parameters. Do not use tuple parameters. Do not return tuples with more than two elements.

If you want to use more parameters, use a structure or class to pass multiple arguments, as explained in the [Specification design pattern](http://en.wikipedia.org/wiki/Specification_pattern).

### Don't use `ref` or `out` parameters :red_circle:
They make code less understandable and might cause people to introduce bugs. Instead, return compound objects or tuples.

**Exception:** Calling and declaring members that implement the [TryParse](https://docs.microsoft.com/en-us/dotnet/api/system.int32.tryparse) pattern is allowed.
``` c#
bool success = int.TryParse(text, out int number);
```

### Prefer `is` patterns over `as` operations :red_circle:

If you use 'as' to safely upcast an interface reference to a certain type, always verify that the operation does not return `null`. Failure to do so may cause a `NullReferenceException` at a later stage if the object did not implement that interface.

**Bad**
``` c#
var remoteUser = user as RemoteUser;
if (remoteUser != null)
{
}
```

**Good**
``` c#
if (user is RemoteUser remoteUser)
{
}
```

### Don't comment out code :red_circle:

Never check in code that is commented out. Instead, use a work item tracking system to keep track of some work to be done. Nobody knows what to do when they encounter a block of commented-out code. Was it temporarily disabled for testing purposes? Was it copied as an example? Should I delete it?

### Use proper casing for language elements :red_circle: 

| Language element&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|Casing&nbsp;&nbsp;&nbsp;&nbsp;|Example|
|--------------------|----------|:-----------
| Namespace | Pascal | `System.Drawing` |
| Type parameter | Pascal | `TView` |
| Interface | Pascal | `IBusinessService`
| Class, struct | Pascal | `AppDomain`
| Enum | Pascal | `ErrorLevel` |
| Enum member | Pascal | `FatalError` |
| Resource key | Pascal | `SaveButtonTooltipText` |
| Constant field | Pascal | `MaximumItems` |
| Private static readonly field | Pascal | `RedValue` |
| Private field | Camel | `_listItem` |
| Non-private field | Pascal | `MainPanel` |
| Property | Pascal | `BackColor` |
| Event | Pascal | `Click` |
| Method | Pascal | `ToString` |
| Local function | Pascal | `FormatText` |
| Parameter | Camel | `typeName` |
| Tuple element names | Pascal | `(string First, string Last) name = ("John", "Doe");` |
| | | `var name = (First: "John", Last: "Doe");` |
| | | `(string First, string Last) GetName() => ("John", "Doe");` |
| Variables declared using tuple syntax | Camel | `(string first, string last) = ("John", "Doe");` |
| | | `var (first, last) = ("John", "Doe");` |
| Local variable | Camel | `listOfValues` |

**Note:** in case of ambiguity, the rule higher in the table wins.

### Don't include numbers in variables, parameters and type members :white_circle:
In most cases they are a lazy excuse for not defining a clear and intention-revealing name.

### Don't prefix fields except if otherwise stated in the naming definitions :red_circle:
For example, don't use `g_` or `s_` to distinguish static from non-static fields. A method in which it is difficult to distinguish local variables from member fields is generally too big. Examples of incorrect identifier names are: `mUserName`, `m_loginTime`.

### Don't use abbreviations :large_blue_circle:
For example, use `ButtonOnClick` rather than `BtnOnClick`. Avoid single character variable names, such as `i` or `q`. Use `index` or `query` instead.

### Name members, parameters and variables according to their meaning and not their type :large_blue_circle:
- Use functional names. For example, `GetLength` is a better name than `GetInt`.
- Don't use terms like `Enum`, `Class` or `Struct` in a name.
- Identifiers that refer to a collection type should have plural names.

### Name types using nouns, noun phrases or adjective phrases :large_blue_circle:
For example, the name IComponent uses a descriptive noun, ICustomAttributeProvider uses a noun phrase and IPersistable uses an adjective.
Bad examples include `SearchExamination` (a page to search for examinations), `Common` (does not end with a noun, and does not explain its purpose) and `SiteSecurity` (although the name is technically okay, it does not say anything about its purpose).

Don't include terms like `Utility` or `Helper` in classes. Classes with names like that are usually static classes and are introduced without considering object-oriented principles.

### Name generic type parameters with descriptive names :large_blue_circle:
- Always prefix type parameter names with the letter `T`.
- Always use a descriptive name unless a single-letter name is completely self-explanatory and a longer name would not add value. Use the single letter `T` as the type parameter in that case.
- Consider indicating constraints placed on a type parameter in the name of the parameter. For example, a parameter constrained to `ISession` may be called `TSession`.

### Don't repeat the name of a class or enumeration in its members :red_circle:
``` c#
class Employee
{
	// Wrong!
	static GetEmployee() {...}
	DeleteEmployee() {...}
	
	// Right
	static Get() {...}
	Delete() {...}
	
	// Also correct.
	AddNewJob() {...}
	RegisterForMeeting() {...}
}
```

### Name members similarly to members of related .NET Framework classes :large_blue_circle:
.NET developers are already accustomed to the naming patterns the framework uses, so following this same pattern helps them find their way in your classes as well. For instance, if you define a class that behaves like a collection, provide members like `Add`, `Remove` and `Count` instead of `AddItem`, `Delete` or `NumberOfItems`.

### Avoid short names or names that can be mistaken for other names :red_circle:
Although technically correct, statements like the following can be confusing:

``` c#
bool b001 = (lo == l0) ? (I1 == 11) : (lOl != 101);
```

### Properly name properties :large_blue_circle:
- Name properties with nouns, noun phrases, or occasionally adjective phrases. 
- Name boolean properties with an affirmative phrase. E.g. `CanSeek` instead of `CannotSeek`.
- Consider prefixing boolean properties with `Is`, `Has`, `Can`, `Allows`, or `Supports`.
- Consider giving a property the same name as its type. When you have a property that is strongly typed to an enumeration, the name of the property can be the same as the name of the enumeration. For example, if you have an enumeration named `CacheLevel`, a property that returns one of its values can also be named `CacheLevel`.

### Name methods and local functions using verbs or verb-object pairs :large_blue_circle:
Name a method or local function using a verb like `Show` or a verb-object pair such as `ShowDialog`. A good name should give a hint on the *what* of a member, and if possible, the *why*.

Also, don't include `And` in the name of a method or local function. That implies that it is doing more than one thing, which violates the Single Responsibility Principle.

### Name namespaces using names, layers, verbs and features :white_circle:
For instance, the following namespaces are good examples of that guideline.

``` c#
NHibernate.Extensibility
Microsoft.ServiceModel.WebApi
Microsoft.VisualStudio.Debugging
```

**Note:** Never allow namespaces to contain the name of a type, but a noun in its plural form (e.g. `Collections`) is usually OK.

### Group extension methods in a class suffixed with Extensions :red_circle:
If the name of an extension method conflicts with another member or extension method, you must prefix the call with the class name. Having them in a dedicated class with the `Extensions` suffix improves readability.

### Postfix asynchronous methods with `Async` or `TaskAsync` :red_circle:
The general convention for methods and local functions that return `Task` or `Task<TResult>` is to postfix them with `Async`. But if such a method already exists, use `TaskAsync` instead.

## Performance Guidelines

### Consider using `Any()` to determine whether an `IEnumerable<T>` is empty :large_blue_circle:
When a member or local function returns an `IEnumerable<T>` or other collection class that does not expose a `Count` property, use the `Any()` extension method rather than `Count()` to determine whether the collection contains items. If you do use `Count()`, you risk that iterating over the entire collection might have a significant impact (such as when it really is an `IQueryable<T>` to a persistent store).

**Note:** If you return an `IEnumerable<T>` to prevent changes from calling code, consider read-only classes.

### Beware of mixing up `async`/`await` with `Task.Wait` :red_circle:
`await` does not block the current thread but simply instructs the compiler to generate a state-machine. However, `Task.Wait` blocks the thread and may even cause deadlocks.

### Prefer creating background tasks and batches with Hangfire for CPU-intensive activities :red_circle:
:police_car: Original rule stated that you `Prefer Task.Run for CPU-intensive activities`
Offload any heavy processing to a background task managed by the worker.