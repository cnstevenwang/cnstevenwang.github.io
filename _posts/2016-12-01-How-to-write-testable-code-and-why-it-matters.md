Unit testing is an essential instrument in the toolbox of any serious software developer. However, it can sometimes be quite difficult to write a good unit test for a particular piece of code. Having difficulty testing their own or someone else’s code, developers often think that their struggles are caused by a lack of some fundamental testing knowledge or secret unit testing techniques.

In this article, I would like to show that unit tests are quite easy; the real problems that complicate unit testing, and introduce expensive complexity, are a result of poorly-designed, untestable code. We will discuss what makes code hard to test, which anti-patterns and bad practices we should avoid to improve testability, and what other benefits we can achieve by writing testable code. We will see that writing testable code is not just about making testing less troublesome, but about making the code itself more robust, and easier to maintain.  
![](/assets/articles/how-to-write-testable-code-and-why-it-matters/1.gif) 

## What is Unit Testing?  
Essentially, a unit test is a method that instantiates a small portion of our application and verifies its behavior **independently from other parts**. A typical unit test contains 3 phases: First, it initializes a small piece of an application it wants to test (also known as the system under test, or SUT), then it applies some stimulus to the system under test (usually by calling a method on it), and finally, it observes the resulting behavior. If the observed behavior is consistent with the expectations, the unit test passes, otherwise, it fails, indicating that there is a problem somewhere in the system under test. These three unit test phases are also known as Arrange, Act and Assert, or simply AAA.

A unit test can verify different behavioral aspects of the system under test, but most likely it will fall into one of the following two categories: state-based or interaction-based. Verifying that the system under test produces correct results, or that its resulting state is correct, is called **state-based** unit testing, while verifying that it properly invokes certain methods is called **interaction-based** unit testing.

As a metaphor for a proper unit test, imagine a mad scientist who wants to build some supernatural chimera, with frog legs, octopus tentacles, bird wings, and a dog’s head. (This metaphor is pretty close to what programmers actually do at work). How would that scientist make sure that every part (or unit) he picked actually works? Well, he can take, let’s say, a single frog’s leg, apply an electrical stimulus to it, and check for proper muscle contraction. What he is doing is essentially the same Arrange-Act-Assert steps of the unit test; the only difference is that, in this case, unit refers to a physical object, not to an abstract object we build our programs from.  
![](/assets/articles/how-to-write-testable-code-and-why-it-matters/2.jpg)   
>>>
I will use C# for all examples in this article, but the concepts described apply to all object-oriented programming languages.
>>>
A simple unit test could look like this:  
```C#
[TestMethod]
public void IsPalindrome_ForPalindromeString_ReturnsTrue()
{
    // In the Arrange phase, we create and set up a system under test.
    // A system under test could be a method, a single object, or a graph of connected objects.
    // It is OK to have an empty Arrange phase, for example if we are testing a static method -
    // in this case SUT already exists in a static form and we don't have to initialize anything explicitly.
    PalindromeDetector detector = new PalindromeDetector(); 

    // The Act phase is where we poke the system under test, usually by invoking a method.
    // If this method returns something back to us, we want to collect the result to ensure it was correct.
    // Or, if method doesn't return anything, we want to check whether it produced the expected side effects.
    bool isPalindrome = detector.IsPalindrome("kayak");

    // The Assert phase makes our unit test pass or fail.
    // Here we check that the method's behavior is consistent with expectations.
    Assert.IsTrue(isPalindrome);
}
```
## Unit Tests vs. Integration Tests
Another important thing to consider is the difference between unit and integration tests.

The purpose of a unit test is to verify the behavior of a relatively small piece of software, independently from other parts. Unit tests are narrow in scope, and allow us to cover all cases, ensuring that every single part works correctly.

On the other hand, integration tests demonstrate that different parts of a system **work together in the real-life environment**. They validate complex scenarios (we can think of integration tests as a user performing some high-level operation within our system), and usually require external resources, like databases or web servers, to be present.

Let’s go back to our mad scientist metaphor, and suppose that he has successfully combined all the parts of the chimera. He wants to perform an integration test of the resulting creature, making sure that it can, let’s say, walk on different types of terrain. First of all, the scientist must emulate an environment for the creature to walk on. Then, he throws the creature into that environment and pokes it with a stick, observing if it walks and moves as designed. After finishing a test, the mad scientist cleans up all the dirt, sand and rocks that are now scattered in his lovely laboratory.  
![](/assets/articles/how-to-write-testable-code-and-why-it-matters/3.jpg)   

Notice the significant difference between unit and integration tests: A unit test verifies the behavior of small part of the application, isolated from the environment and other parts, and is quite easy to implement, while an integration test covers interactions between different components, in the close-to-real-life environment, and requires more effort, including additional setup and teardown phases.

A reasonable combination of unit and integration tests ensures that every single unit works correctly, independently from others, and that all these units play nicely when integrated, giving us a high level of confidence that the whole system works as expected. However, we must remember to always identify what kind of test we are implementing: a unit or an integration test. The difference can sometimes be deceiving. If we think we are writing a unit test to verify some subtle edge case in a business logic class, and realize that it requires external resources like web services or databases to be present, something is not right — essentially, we are using a sledgehammer to crack a nut. And that means bad design.

## What Makes a Good Unit Test?
Before diving into the main part of this tutorial, let’s quickly discuss the properties of a good unit test. A good unit test should be:

+ **Easy to write**. Developers typically write lots of unit tests to cover different cases and aspects of the application’s behavior, so it should be easy to code all of those test routines without enormous effort.
+ **Readable**. The intent of a unit test should be clear. A good unit test tells a story about some behavioral aspect of our application, so it should be easy to understand which scenario is being tested and — if the test fails — easy to detect how to address the problem. With a good unit test, we can fix a bug without actually debugging the code!  
+ **Reliable**. Unit tests should fail only if there’s a bug in the system under test. That seems pretty obvious, but programmers often run into an issue when their tests fail even when no bugs were introduced. For example, tests may pass when running one-by-one, but fail when running the whole test suite, or pass on our development machine and fail on the continuous integration server. These situations are indicative of a design flaw. Good unit tests should be reproducible and independent from external factors such as the environment or running order.
+ **Fast**. Developers write unit tests so they can repeatedly run them and check that no bugs have been introduced. If unit tests are slow, developers are more likely to skip running them on their own machines. One slow test won’t make a significant difference; add one thousand more and we’re surely stuck waiting for a while. Slow unit tests may also indicate that either the system under test, or the test itself, interacts with external systems, making it environment-dependent.
+ **Truly unit, not integration**. As we already discussed, unit and integration tests have different purposes. Both the unit test and the system under test should not access the network resources, databases, file system, etc., to eliminate the influence of external factors.

That’s it — there are no secrets to writing unit tests. However, there are some techniques that allow us to write testable code.

## Testable and Untestable Code  
Some code is written in such a way that it is hard, or even impossible, to write a good unit test for it. So, what makes code hard to test? Let’s review some anti-patterns, code smells, and bad practices that we should avoid when writing testable code.

## Poisoning the Codebase with Non-Deterministic Factors
Let’s start with a simple example. Imagine that we are writing a program for a smart home microcontroller, and one of the requirements is to automatically turn on the light in the backyard if some motion is detected there during the evening or at night. We have started from the bottom up by implementing a method that returns a string representation of the approximate time of day (“Night”, “Morning”, “Afternoon” or “Evening”):
```C#
public static string GetTimeOfDay()
{
    DateTime time = DateTime.Now;
    if (time.Hour >= 0 && time.Hour < 6)
    {
        return "Night";
    }
    if (time.Hour >= 6 && time.Hour < 12)
    {
        return "Morning";
    }
    if (time.Hour >= 12 && time.Hour < 18)
    {
        return "Afternoon";
    }
    return "Evening";
}
```
Essentially, this method reads the current system time and returns a result based on that value. So, what’s wrong with this code?

If we think about it from the unit testing perspective, we’ll see that it is not possible to write a proper state-based unit test for this method. DateTime.Now is, essentially, a hidden input, that will probably change during program execution or between test runs. Thus, subsequent calls to it will produce different results.

Such **non-deterministic** behavior makes it impossible to test the internal logic of the GetTimeOfDay() method without actually changing the system date and time. Let’s have a look at how such test would need to be implemented:
```
[TestMethod]
public void GetTimeOfDay_At6AM_ReturnsMorning()
{
    try
    {
        // Setup: change system time to 6 AM
        ...

        // Arrange phase is empty: testing static method, nothing to initialize

        // Act
        string timeOfDay = GetTimeOfDay();

        // Assert
        Assert.AreEqual("Morning", timeOfDay);
    }
    finally
    {
        // Teardown: roll system time back
        ...
    }
}
```

Tests like this would violate a lot of the rules discussed earlier. It would be expensive to write (because of the non-trivial setup and teardown logic), unreliable (it may fail even if there are no bugs in the system under test, due to system permission issues, for example), and not guaranteed to run fast. And, finally, this test would not actually be a unit test — it would be something between a unit and integration test, because it pretends to test a simple edge case but requires an environment to be set up in a particular way. The result is not worth the effort, huh?

Turns out that all these testability problems are caused by the low-quality GetTimeOfDay() API. In its current form, this method suffers from several issues:

[Origin Post: https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters](https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters)