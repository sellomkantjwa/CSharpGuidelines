## Layout Guidelines

### Use a common layout :red_circle:

- Keep the length of each line under 130 characters.
- Use tab indentation
- Keep one space between keywords like `if` and the expression, but don't add spaces after `(` and before `)` such as: `if (condition == null)`.
- Add a space around operators like `+`, `-`, `==`, etc.
- Always put opening and closing curly braces on a new line. :police_car:
- Don't indent object/collection initializers and initialize each property on a new line, so use a format like this: 
	``` c#
	var dto = new ConsumerDto
	{
		Id = 123,
		Name = "Microsoft",
		PartnerShip = PartnerShip.Gold,
		ShoppingCart =
		{
			["VisualStudio"] = 1
		}
	};
	```
- Don't indent lambda statement blocks and use a format like this:
	``` c#
	methodThatTakesAnAction.Do(x =>
	{ 
		// do something like this 
	}
	```

- Keep expression-bodied-members on one line. Break long lines after the arrow sign, like this:
	``` c#
	private string GetLongText =>
		"ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC ABC";
	```

- Put the entire LINQ statement on one line, or start each keyword at the same indentation, like this:
	``` c#
	var query = from product in products where product.Price > 10 select product;
	```
  	or
	``` c#
	var query =  
		from product in products  
		where product.Price > 10  
		select product;
	```
- Start the LINQ statement with all the `from` expressions and don't interweave them with restrictions.
- Add parentheses around every binary expression, but don't add parentheses around unary expressions. For example `if (!string.IsNullOrEmpty(str) && (str != "new"))`
- Add an empty line between multi-line statements, between multi-line members, after the closing curly braces, between unrelated code blocks, around the `#region` keyword, and between the `using` statements of different root namespaces.

### Order and group namespaces according to the company :white_circle:
``` c#
// Microsoft namespaces are first
using System;
using System.Collections.Generic;
using System.XML;

// Then any other namespaces in alphabetic order
using AvivaSolutions.Business;
using AvivaSolutions.Standard;
using Telerik.WebControls;
using Telerik.Ajax;
```

### Place members in a well-defined order :red_circle:
Maintaining a common order allows other team members to find their way in your code more easily. In general, a source file should be readable from top to bottom, as if reading a book, to prevent readers from having to browse up and down through the code file.

1. Private fields and constants (in a region) :police_car:
2. Public constants
3. Public static readonly fields
4. Factory methods
5. Constructors and the finalizer
6. Events 
7. Public properties
8. Other methods and private properties in calling order

Declare local functions at the bottom of their containing method bodies (after all executable code).

### Be reluctant with `#region` :red_circle:
:police_car: Might be worth keeping the rule around but, feels like one of those class-doing-too-much things
Regions can be helpful, but can also hide the main purpose of a class. Therefore, use `#region` only for:

- Private fields and constants (preferably in a `Private Definitions` region).
- Nested classes
- Interface implementations (only if the interface is not the main purpose of that class)

### Use expression-bodied members appropriately 
Favor expression-bodied member syntax over regular member syntax only when:

- the body consists of a single statement and
- the body fits on a single line.

**Good**
``` c#
public double RatePerHour => (Salary * 40) / 100;
```
**Bad**
``` c#
public double LeaveDays => RatePerHour > 40
	? 14 : RatePerHour > 50
	? 15 : RatePerHour > 60
	? ...etc
```
