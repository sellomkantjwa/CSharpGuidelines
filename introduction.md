# Coding Guidelines

> Adapted from https://csharpcodingguidelines.com/

## Index

- [Class Design](pages/class-design.md)
- [Member Design Guidelines](pages/member-design.md)
- [Miscellaneous Design Guidelines](pages/misc-design.md)
- [Maintainability Guidelines](pages/maintainability.md)
- [Performance Guidelines](pages/performance.md)
- [Framework Guidelines](pages/framework.md)
- [Layout Guidelines](pages/layout.md)

## Introduction

### 1.1. What is this
Coding guidelines we should ideally follow. You are always free to challenge these decisions and should do so. The importance of the application of certain rules is always dependent on the situation. If you find yourself stuck, chat with other developers in your team and try and reach the most rational solution.

Use the basic principles outlined in this guide to help you make that decision.

### 1.2. Purpose of this document

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