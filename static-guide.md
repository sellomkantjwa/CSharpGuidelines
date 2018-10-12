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
	### <rule-name> [optionally add !, !! or !!! - depending on rule importance]

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

### A class or interface should have a single purpose (!!!)

A class or interface should have a single purpose within the system it functions in. In general, a class either represents a primitive type like an email or ISBN number, an abstraction of some business concept, a plain data structure, or is responsible for orchestrating the interaction between other classes. It is never a combination of those. This rule is widely known as the [Single Responsibility Principle](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html), one of the S.O.L.I.D. principles.

**Tip:** A class with the word `And` in it is an obvious violation of this rule.

**Tip:** Use [Design Patterns](http://en.wikipedia.org/wiki/Design_pattern_(computer_science)) to communicate the intent of a class. If you can't assign a single design pattern to a class, chances are that it is doing more than one thing.

**Note** If you create a class representing a primitive type you can greatly simplify its use by making it immutable.

### Only create a constructor that returns a useful object (!)

There should be no need to set additional properties before the object can be used for whatever purpose it was designed. However, if your constructor needs more than three parameters [LINK TO RULE](/), your class might have too much responsibility.

**Note** EF core also supports [entity types with constructors](https://docs.microsoft.com/en-gb/ef/core/modeling/constructors)

### An interface should be small and focused (!!)

Interfaces should have a name that clearly explains their purpose or role in the system. Do not combine many vaguely related members on the same interface just because they were all on the same class. Separate the members based on the responsibility of those members, so that callers only need to call or implement the interface related to a particular task. This rule is more commonly known as the [Interface Segregation Principle](https://lostechies.com/wp-content/uploads/2011/03/pablos_solid_ebook.pdf).

<!-- ### Use an interface rather than a base class to support multiple implementations (!)

If you want to expose an extension point from your class, expose it as an interface rather than as a base class. You don't want to force users of that extension point to derive their implementations from a base class that might have an undesired behavior. However, for their convenience you may implement a(n abstract) default implementation that can serve as a starting point. -->

### Use an interface to decouple classes from each other (!!)

Interfaces are a very effective mechanism for decoupling classes from each other:

- They can prevent bidirectional associations; 
- They simplify the replacement of one implementation with another; 
- They allow the replacement of an expensive external service or resource with a temporary stub for use in a non-production environment.
- They allow the replacement of the actual implementation with a dummy implementation or a fake object in a unit test; 
- Using a dependency injection framework you can centralize the choice of which class is used whenever a specific interface is requested.

### Avoid static classes (!!)

With the exception of extension method containers, static classes very often lead to badly designed code. They are also very difficult, if not impossible, to test in isolation, unless you're willing to use some very hacky tools.

**Note:** If you really need that static class, mark it as static so that the compiler can prevent instance members and instantiating your class. This relieves you of creating an explicit private constructor.

<!-- Hiding this - people shouldn't even think about it

### <a name="av1010"></a> Don't suppress compiler warnings using the `new` keyword (AV1010) ![](/assets/images/1.png)

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

### It should be possible to treat a derived object as if it were a base class object (!!!)

In other words, you should be able to use a reference to an object of a derived class wherever a reference to its base class object is used without knowing the specific derived class. A very notorious example of a violation of this rule is throwing a `NotImplementedException` when overriding some of the base-class methods. A less subtle example is not honoring the behavior expected by the base class.   
  
**Note:** This rule is also known as the Liskov Substitution Principle, one of the [S.O.L.I.D.](http://www.lostechies.com/blogs/chad_myers/archive/2008/03/07/pablo-s-topic-of-the-month-march-solid-principles.aspx) principles.

### Avoid exposing the other objects an object depends on (!!)

If you find yourself writing code like this then you might be violating the [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter).

	repo.DbSet.DoThing();

An object should not expose any other classes it depends on because callers may misuse that exposed property or method to access the object behind it. By doing so, you allow calling code to become coupled to the class you are using, and thereby limiting the chance that you can easily replace it in a future stage.

### Avoid bidirectional dependencies (!!)

This means that two classes know about each other's public members or rely on each other's internal behavior. Refactoring or replacing one of those classes requires changes on both parties and may involve a lot of unexpected work. The most obvious way of breaking that dependency is to introduce an interface for one of the classes and using Dependency Injection.

**Exception:** Domain models such as defined in [Domain-Driven Design](http://domaindrivendesign.org/) tend to occasionally involve bidirectional associations that model real-life associations. In those cases, make sure they are really necessary, and if they are, keep them in.

### Classes should have state and behavior (!!!)

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

### Classes should protect the consistency of their internal state (!!)

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

### Use a method instead of a property (!!!)

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

### Don't use mutually exclusive properties (!!!)

Having properties that cannot be used at the same time typically signals a type that represents two conflicting concepts. Even though those concepts may share some of their behavior and states, they obviously have different rules that do not cooperate.

This violation is often seen in domain models and introduces all kinds of conditional logic related to those conflicting rules, causing a ripple effect that significantly increases the maintenance burden.

**Bad**
```
_listing.IsLost = true;
_listing.IsWon = true;
```

### A property, method or local function should do only one thing (!!!)

A method body should have a single responsibility.

### Don't expose stateful objects through static members (!!)

A stateful object is an object that contains many properties and lots of behavior behind it. If you expose such an object through a static property or method of some other object, it will be very difficult to refactor or unit test a class that relies on such a stateful object.

A classic example of this is the `HttpContext.Current` property, part of ASP.NET. Many see the `HttpContext` class as a source of a lot of ugly code. In fact, the testing guideline [Isolate the Ugly Stuff](http://codebetter.com/jeremymiller/2005/10/21/haacked-on-tdd-and-jeremys-first-rule-of-tdd/) often refers to this class.


### Return an `IQueryable<T>` to clearly indicate when execution of a Linq-to-X query is deferred (!!!)

When you are building a query and no execution has been triggered, explicitly keep the type to IQueryable to ensure the response of the function clearly indicates the possible side-effects of any execution againts it.

### Return an `IEnumerable<T>` or `ICollection<T>` instead of a concrete collection class (!!)

You generally don't want callers to be able to change an internal collection, so don't return arrays, lists or other collection classes directly. Instead, return an `IEnumerable<T>`, or, if the caller must be able to determine the count, an `ICollection<T>`.

**Note:** You can also use `IReadOnlyCollection<T>`, `IReadOnlyList<T>` or `IReadOnlyDictionary<TKey, TValue>`.

**Exception:** Immutable collections such as `ImmutableArray<T>`, `ImmutableList<T>` and `ImmutableDictionary<TKey, TValue>` prevent modifications from the outside and are thus allowed.

### Properties, arguments and return values representing strings or collections should never be `null` (!!!)

Returning `null` can be unexpected by the caller. Always return an empty collection or an empty string instead of a `null` reference. This also prevents cluttering your code base with additional checks for `null`, or even worse, `string.IsNullOrEmpty()`.

### Define parameters as specific as possible (!!!)

If your method or local function needs a specific piece of data, define parameters as specific as that and don't take a container object instead. For instance, consider a method that needs a connection string that is exposed through a central `IConfiguration` interface. Rather than taking a dependency on the entire configuration, just define a parameter for the connection string. This not only prevents unnecessary coupling, it also improves maintainability in the long run.

**Note:** An easy trick to remember this guideline is the *Don't ship the truck if you only need a package*.

### Consider using domain-specific value types rather than primitives (!!)

Instead of using strings, integers and decimals for representing domain-specific types such as an ISBN number, an email address or amount of money, consider creating dedicated value objects that wrap both the data and the validation rules that apply to it. By doing this, you prevent ending up having multiple implementations of the same business rules, which both improves maintainability and prevents bugs.


