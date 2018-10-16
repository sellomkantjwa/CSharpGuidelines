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

### Favor `async`/`await` over `Task` continuations :red_circle:

**Good**
``` c#
public async Task<Data> GetDataAsync()
{
	string result = await MyWebService.FetchDataAsync();
	return new Data(result);
}
```

**Bad**
``` c#
public Task<Data> GetDataAsync()
{
	return MyWebService.FetchDataAsync()
	.ContinueWith(t => new Data(t.Result));
}
```

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
