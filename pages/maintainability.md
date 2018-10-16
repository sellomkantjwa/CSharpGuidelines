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
