---
layout: post
title: Legacy Code
categories: Best Practices
banner: /images/legacy_code/header.png
banner_alt: Legacy Code
banner_author: <a href="https://pixabay.com/users/4064462-4064462/">Pixabay Contributor</a>
---

As developers, we know all too well that code has a lifespan. Knowing how to avoid code re-writes, how to improve software longevity, readability, maintainability and reduce the risk of bugs are all ongoing challenges. It's clear that a good team will need to set some best practices and standards either directly through documentation or indirectly through consistency in the code. Some recent reading around the topic of legacy code (Working Effectively with Legacy Code by Michael Feathers) and technical debt inspired me to write this post to re-collect what I've learned. There are actually some very simple tweaks that everyone can do to make technical debt easier to deal with. When done right and often, small changes can hang together as part of a much bigger picture or help the next guy pull out some code, uplift a module or re-write some functionality from scratch. Sure enough ensuring your code reads well and that it is free from bugs will mean a focus on writing tests. Although I find that the cost of writing tests is not actually as time consuming as people think, instead it tends to be more about letting go of some utopian best practices and being willing to make some short term compromises. Here are my tips and thoughts on improving legacy code.

### 1. Breaking Dependencies

When we talk about legacy code the largest burden by far and large is making new changes without changing or losing any existing functionality. TDD tackles this but some classes can grow so out of control that it may seem difficult to test them. Testable code in general leads to well designed and clean code. Here are some approaches to breaking out dependencies to lead to easier testing.

<ul>
<li><b>Interfaces and Abstractions</b> - Depending on abstractions over implementations is a key OOP concept, if a class under inspection refers to concrete classes in the constructor's signature then this could be the first place to turn to. Sure you can test with some dummy object values but at some point you are going to need to need to mock out this behaviour fully or swap in some new classes perhaps. If a particular dependency is causing you trouble, the dependency is used in more than one place and you haven't written an interface or abstract class for it already, then do this first.
</li>

<li><b>Stub new methods</b> - A technique many of us use subconsciously when writing tests. You add some new functionality to a method, the method becomes larger and you need a quick way to test the old method without what you added getting in the way. Perhaps you have now have a file that you now need to read, a call for a web request or a long running task you need to dispatch. Instead of writing the change inline inside of an existing function it would seem natural to at least create a new method to separate the new functionality, thus enabling you to mock the new method and test the old functionality. The downside being that in order to mock the function it instantly becomes public. This isn't a huge problem in my view if the function is consistent with our external view of this class's responsibilities (if it isn't we should probably break out a new class anyway and use DI). While this is very familiar technique to many, it is important to remember this is always a choice you can make over writing more mocking code. Some engineers can jump the gun and start to mock out libraries or dependencies rather than stubbing functions.
</li>

<li>
<b>Stub new classes</b> - Similar to the above, getting code under test can mean shifting your logic into a new class to help modularise your code. Create a new parameter for the default constructor of the original class under test and inject the new object through either a Dependency Injection framework or by explicitly creating a new instance in the calling code. In a vast majority of cases breaking out functionality is a good thing, but be careful to think about re-usability and overall coherence of your codebase. There are probably going to be a few dependencies on the constructor so a quicker and less intrusive change may introduce a setter rather than modifying the constructor. Some will argue this is not clean code anymore as we're exposing more properties than needed just for testing. However the goal is much larger here, if you don't have time to make a change then breaking out a new class and creating a new setter saying 'SetMockFieldValue' is not going to hurt. The intent of the setter is still very clear and it means you can get along with writing valuable tests.</li>

{% highlight java %}
public interface MyNewInterface {
    public void functionIWouldLikeToTest
}

public class MyOriginalClass {
    private MyNewInterface dependency;

    public MyOriginalClass(MyNewInterface dependency) {
        this.dependency = dependency;
    }

    // Use this if it is not easy to change the constructor
    public void setDependencyThroughMethodDI(
        MyNewInterface dependency) {
        this.dependency = dependency;
    }
}
{% endhighlight %}
<li>
<b>Wrapper Classes/Decorators</b> - A great use of the <a href="https://www.geeksforgeeks.org/the-decorator-pattern-set-2-introduction-and-design/">Decorator Pattern</a> or wrapper classes is to use them for adding layers of new behaviour. You can build out new functionality into new wrappers so they are easily testable and isolated from your existing code. Decorators themselves bring an additional benefit such that we can use them to supersede original methods and re-implement any problematic initialisation code. For example, we could write a Mock wrapper from an original class. This can be useful if the existing class is already polluted with many dependencies and using dependency injection will cause more of a mess. We could simply extend the original class but by using a decorator we are favouring composition over inheritance which will make adding new functionality to classes easier in the long run. This does however throw around a little more boilerplate code and decorators do provide a temptation to wrap new functionality in infinite layers of complexity. I would say you should save this method for occasions when more complex functionality needs to be added to a class and for when dependency injection pollutes a class.
</li>

{% highlight java %}
public interface Actions {
    public void existingMethod1;
    public void existingMethod2;
}

public class OriginalClass implements Actions {
    public void existingMethod1() { .. }
    public void existingMethod2() { .. }
}

public class MyDecorator implements Actions {
    private Actions originalObj;

    public MyDecorator(Actions originalObject) {
        this.originalObj = originalObject;
    }

    public void newMethodToTest() {
        // code here
    }

    public void existingMethod1() {
        originalObj.existingMethod1();
    }

    public void existingMethod2() {
        originalObj.existingMethod2();
    }
}
{% endhighlight %}
<li>

<b>Subclasses</b> - Similar to wrapper classes. If writing wrappers is too much to ask, maybe this requires writing more code than you would like right now, then a quick and dirty solution may be just to extend the class under test and override the necessary functions. As mentioned in the wrapper classes section we could introduce a new Mock class as an extension of an original class and override any problematic initialisation.
</li>

<li>
<b>Null Object Pattern</b> - The <a href="https://www.geeksforgeeks.org/null-object-design-pattern/">Null Object Pattern</a> is useful for setting up test cases by passing a dummy equivalent of the dependency into a class that has empty attributes. Passing null may cause some errors to be thrown and in turn this reduces the risk of touching the existing code to handle NullPointerExceptions as the code should still handle an empty object just as any other.
</li>
<li>

<b>Static Methods</b> - While this is not a favorite choice of mine, consider turning a method into a static one if the method is hard to test as part of a class. If the method is a very isolated piece of functionality you could break this into a static method until you find a better solution.
</li>
</ul>

### 2. Naming
The power of naming can often be underestimated. Every team will have their own set standards and you can normally gauge by the existing code what makes a consistent name to fit within the mix. You should choose names that make sense to your team and within your domain but not unobvious to new comers. Failing that, my personal view is that a good name is one that is short as possible and as long as it needs to be. As a rule of thumb they should be at least more than one letter unless there is a good reason such as a lambda, algebra, formulas, for loops or in contexts where they make sense. Conversely they should be shorter than around 30-35 characters. A name should be descriptive and should not use un-documented or uncommon acronyms. It can be amazing how many short hands can be used through a codebase on the assumption that everyone is familiar with their meaning. If its a function include a verb to indicate it is actually a function and try not to use 'and', come up with a different name instead that represents the function as a whole or break down the method further. I am guilty of this but try not to embed a variable's type into the variable name, if anyone changes the type at any point this can be quite confusing. Think twice before including 'Interface' or 'Abstract' into your abstractions. These are a waste of characters and are not that much more expressive. This also goes to a lesser extent to Hungarian notation. For example, a IVehicle/Car could just simply be Vehicle/Car on their own and it's normally not too difficult in modern IDEs to see that a class is either abstract or an interface. Swap out comments with well-named variables as your code should be as good and self descriptive, comments should really be used sparingly. 

### 3. Design Roadmaps
There can be particular confusing parts to an application or there may be a lack of structure altogether. It pays to have discussions with team members about the overall architecture to reinforce understanding. Draw out some diagrams, document features and the overall design separately to the codebase. An up to date confluence page is going to help reinforce everyone's understanding and ensure everyone is on the same page. Moreover this encourages knowledge sharing in the team and for others to do the same.

### 4. Delete code and comments
There are countless times where unused code and old comments have led me to a bunch of confusion. Sometimes an old way of doing something is left around or some comments surrounding a method no longer make sense. These are small things but can introduce a lot of confusion and misdirection. If you know a line or two that doesn't belong there anymore you should spend a couple of minutes to clean it up or delete it in a separate commit.

### 5. Logging
Without logs debugging production issues is pure guess work. I personally advocate more logging at the risk of storing too much and over-polluting so I can at least capture issues, but you may choose to go with a more strategic logging strategy. Either way, define a standard, stick with it to make it easily searchable. Logging is going to aid your understanding on legacy code.

### 6. Error Handling
Putting best practices to good use when handling exceptions will pay dividends for debugging production issues. 
Use the right exception for an error, if unexpectedly a number is null then throw a NullPointerException, if an argument is incorrect throw an IllegalArgumentException. Try-catches should start with the most specific exception in the block and then cascade down to more generic ones so that errors are logged with greater accuracy. Re-throwing exceptions in many languages will mean you lose information about the original stack trace such as the original source and line number of the error. Personally I try to deal with an exception within the original try-catch block, log the exception and then use something similar to the Null Object pattern to notify the caller that something went wrong. If this is not possible it would be better to create a new type for your exception to best wrap the original exception and better describe the error. Last of all, I would try to standardise exceptions as much as possible as well as the messages that are thrown to improve readability.

<b>Bad Error Handling</b>
{% highlight java %}
public String getCustomerName(int id) {
    try {
        // stream will not close automatically 
        // unless handled ina finally statement
        FileInputStream fio = new FileInputStream(filename);
        Customer c = getCustomer(fio, id);
        return c.toString();
    }
    // no specific error handling
    catch (Exception e) {
        // re-throwing error loses stack trace
        // and can cause confusion if handled twice
        throw e;
    }
    return "";
}
{% endhighlight %}
<b>Better Error Handling</b>
{% highlight java %}
public String getCustomerName(int id) {
    // using a try-with statement so that the 
    // stream is automatically closed
    try (FileInputStream fio = new FileInputStream(filename)) {
        Customer c = getCustomer(fio);
        return c.toString();
    }
    catch (FileNotFoundException e2) {
        // specific error is handled first
        logError(e2);
    }
    catch (NullPointerException e1) {
        logError(e1);
        // if we really need to re-throw an exception then
        // we wrap it in a new exception to be handled 
        // by the caller
        throw new CustomerNotFoundException("Customer not found.", e1);
    }
    catch (Exception e) {
        logError(e);
    }
    return "";
}

{% endhighlight %}
These are just a few points I would consider important to moving legacy code forward with a particular emphasis on tests and breaking dependencies to facilitate change and refactoring without breaking functionality. Hopefully some of the techniques here reinforce what you may already know and help bring your code to a cleaner state.