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
