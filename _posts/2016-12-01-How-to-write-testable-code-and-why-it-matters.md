Unit testing is an essential instrument in the toolbox of any serious software developer. However, it can sometimes be quite difficult to write a good unit test for a particular piece of code. Having difficulty testing their own or someone else’s code, developers often think that their struggles are caused by a lack of some fundamental testing knowledge or secret unit testing techniques.

In this article, I would like to show that unit tests are quite easy; the real problems that complicate unit testing, and introduce expensive complexity, are a result of poorly-designed, untestable code. We will discuss what makes code hard to test, which anti-patterns and bad practices we should avoid to improve testability, and what other benefits we can achieve by writing testable code. We will see that writing testable code is not just about making testing less troublesome, but about making the code itself more robust, and easier to maintain.  
![](/assets/articles/how-to-write-testable-code-and-why-it-matters/1.gif) 

## What is Unit Testing?  
Essentially, a unit test is a method that instantiates a small portion of our application and verifies its behavior **independently from other parts**. A typical unit test contains 3 phases: First, it initializes a small piece of an application it wants to test (also known as the system under test, or SUT), then it applies some stimulus to the system under test (usually by calling a method on it), and finally, it observes the resulting behavior. If the observed behavior is consistent with the expectations, the unit test passes, otherwise, it fails, indicating that there is a problem somewhere in the system under test. These three unit test phases are also known as Arrange, Act and Assert, or simply AAA.

A unit test can verify different behavioral aspects of the system under test, but most likely it will fall into one of the following two categories: state-based or interaction-based. Verifying that the system under test produces correct results, or that its resulting state is correct, is called **state-based** unit testing, while verifying that it properly invokes certain methods is called **interaction-based** unit testing.

As a metaphor for a proper unit test, imagine a mad scientist who wants to build some supernatural chimera, with frog legs, octopus tentacles, bird wings, and a dog’s head. (This metaphor is pretty close to what programmers actually do at work). How would that scientist make sure that every part (or unit) he picked actually works? Well, he can take, let’s say, a single frog’s leg, apply an electrical stimulus to it, and check for proper muscle contraction. What he is doing is essentially the same Arrange-Act-Assert steps of the unit test; the only difference is that, in this case, unit refers to a physical object, not to an abstract object we build our programs from.  
![](/assets/articles/how-to-write-testable-code-and-why-it-matters/2.jpg)   
> I will use C# for all examples in this article, but the concepts described apply to all object-oriented programming languages.

A simple unit test could look like this:  


```

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

```

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

+ **It is tightly coupled to the concrete data source**. It is not possible to reuse this method for processing date and time retrieved from other sources, or passed as an argument; the method works only with the date and time of the particular machine that executes the code. Tight coupling is the primary root of most testability problems.
+ **It violates the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) (SRP)**. The method has multiple responsibilities; it consumes the information and also processes it. Another indicator of SRP violation is when a single class or method has more than one reason to change. From this perspective, the GetTimeOfDay() method could be changed either because of internal logic adjustments, or because the date and time source should be changed.
+ **It lies about the information required to get its job done**. Developers must read every line of the actual source code to understand what hidden inputs are used and where they come from. The method signature alone is not enough to understand the method’s behavior.
+ **It is hard to predict and maintain**. The behavior of a method that depends on a mutable global state cannot be predicted by merely reading the source code; it is necessary to take into account its current value, along with the whole sequence of events that could have changed it earlier. In a real-world application, trying to unravel all that stuff becomes a real headache.

After reviewing the API, let’s finally fix it! Fortunately, this is much easier than discussing all of its flaws — we just need to break the tightly coupled concerns.

## Fixing the API: Introducing a Method Argument
The most obvious and easy way of fixing the API is by introducing a method argument:  

```

public static string GetTimeOfDay(DateTime dateTime)
{    
    if (dateTime.Hour >= 0 && dateTime.Hour < 6)
    {
        return "Night";
    }
    if (dateTime.Hour >= 6 && dateTime.Hour < 12)
    {
        return "Morning";
    }
    if (dateTime.Hour >= 12 && dateTime.Hour < 18)
    {
        return "Noon";
    }
    return "Evening";
}

```

Now the method requires the caller to provide a DateTime argument, instead of secretly looking for this information by itself. From the unit testing perspective, this is great; the method is now deterministic (i.e., its return value fully depends on the input), so state-based testing is as easy as passing some DateTime value and checking the result:  

```

[TestMethod]
public void GetTimeOfDay_For6AM_ReturnsMorning()
{
    // Arrange phase is empty: testing static method, nothing to initialize

    // Act
    string timeOfDay = GetTimeOfDay(new DateTime(2015, 12, 31, 06, 00, 00));

    // Assert
    Assert.AreEqual("Morning", timeOfDay);
}

```

Notice that this simple refactor also solved all the API issues discussed earlier (tight coupling, SRP violation, unclear and hard to understand API) by introducing a clear seam between what data should be processed and how it should be done.

Excellent — the method is testable, but how about its clients? Now it is the caller’s responsibility to provide date and time to the GetTimeOfDay(DateTime dateTime) method, meaning that they could become untestable if we don’t pay enough attention. Let’s have a look how we can deal with that.

## Fixing the Client API: Dependency Injection
Say we continue working on the smart home system, and implement the following client of the GetTimeOfDay(DateTime dateTime) method — the aforementioned smart home microcontroller code responsible for turning the light on or off, based on the time of day and the detection of motion:  

```

public class SmartHomeController
{
    public DateTime LastMotionTime { get; private set; }

    public void ActuateLights(bool motionDetected)
    {
        DateTime time = DateTime.Now; // Ouch!

        // Update the time of last motion.
        if (motionDetected)
        {
            LastMotionTime = time;
        }
        
        // If motion was detected in the evening or at night, turn the light on.
        string timeOfDay = GetTimeOfDay(time);
        if (motionDetected && (timeOfDay == "Evening" || timeOfDay == "Night"))
        {
            BackyardLightSwitcher.Instance.TurnOn();
        }
        // If no motion is detected for one minute, or if it is morning or day, turn the light off.
        else if (time.Subtract(LastMotionTime) > TimeSpan.FromMinutes(1) || (timeOfDay == "Morning" || timeOfDay == "Noon"))
        {
            BackyardLightSwitcher.Instance.TurnOff();
        }
    }
}

```


Ouch! We have the same kind of hidden DateTime.Now input problem — the only difference is that it is located on a little bit higher of an abstraction level. To solve this issue, we can introduce another argument, again delegating the responsibility of providing a DateTime value to the caller of a new method with signature ActuateLights(bool motionDetected, DateTime dateTime). But, instead of moving the problem a level higher in the call stack once more, let’s employ another technique that will allow us to keep both ActuateLights(bool motionDetected) method and its clients testable: [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control), or IoC.

Inversion of Control is a simple, but extremely useful, technique for decoupling code, and for unit testing in particular. (After all, keeping things loosely coupled is essential for being able to analyze them independently from each other.) The key point of IoC is to separate decision-making code (when to do something) from action code (what to do when something happens). This technique increases flexibility, makes our code more modular, and reduces coupling between components.

Inversion of Control can be implemented in a number of ways; let’s have a look at one particular example — [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) using a constructor — and how it can help in building a testable SmartHomeController API.

First, let’s create an IDateTimeProvider interface, containing a method signature for obtaining some date and time:

```

public interface IDateTimeProvider
{
    DateTime GetDateTime();
}

```

Then, make SmartHomeController reference an IDateTimeProvider implementation, and delegate it the responsibility of obtaining date and time:

```

public class SmartHomeController
{
    private readonly IDateTimeProvider _dateTimeProvider; // Dependency

    public SmartHomeController(IDateTimeProvider dateTimeProvider)
    {
        // Inject required dependency in the constructor.
        _dateTimeProvider = dateTimeProvider;
    }

    public void ActuateLights(bool motionDetected)
    {
        DateTime time = _dateTimeProvider.GetDateTime(); // Delegating the responsibility

        // Remaining light control logic goes here...
    }
}

```

Now we can see why Inversion of Control is so called: the control of what mechanism to use for reading date and time was inverted, and now belongs to the client of SmartHomeController, not SmartHomeController itself. Thereby, the execution of the ActuateLights(bool motionDetected) method fully depends on two things that can be easily managed from the outside: the motionDetected argument, and a concrete implementation of IDateTimeProvider, passed into a SmartHomeController constructor.

Why is this significant for testing? It means that different IDateTimeProvider implementations can be used in production code and unit test code. In the production environment, some real-life implementation will be injected (e.g., one that reads actual system time). In the unit test, however, we can inject a “fake” implementation that returns a constant or predefined DateTime value suitable for testing the particular scenario.

A fake implementation of IDateTimeProvider could look like this:  

```

public class FakeDateTimeProvider : IDateTimeProvider
{
    public DateTime ReturnValue { get; set; }

    public DateTime GetDateTime() { return ReturnValue; }

    public FakeDateTimeProvider(DateTime returnValue) { ReturnValue = returnValue; }
}

```

With the help of this class, it is possible to isolate SmartHomeController from non-deterministic factors and perform a state-based unit test. Let’s verify that, if motion was detected, the time of that motion is recorded in the LastMotionTime property:  

```

[TestMethod]
void ActuateLights_MotionDetected_SavesTimeOfMotion()
{
    // Arrange
    var controller = new SmartHomeController(new FakeDateTimeProvider(new DateTime(2015, 12, 31, 23, 59, 59)));

    // Act
    controller.ActuateLights(true);

    // Assert
    Assert.AreEqual(new DateTime(2015, 12, 31, 23, 59, 59), controller.LastMotionTime);
}

```

Great! A test like this was not possible before refactoring. Now that we’ve eliminated non-deterministic factors and verified the state-based scenario, do you think SmartHomeController is fully testable?
## Poisoning the Codebase with Side Effects  
Despite the fact that we solved the problems caused by the non-deterministic hidden input, and we were able to test certain functionality, the code (or, at least, some of it) is still untestable!

Let’s review the following part of the ActuateLights(bool motionDetected) method responsible for turning the light on or off:  

```

// If motion was detected in the evening or at night, turn the light on.
if (motionDetected && (timeOfDay == "Evening" || timeOfDay == "Night"))
{
    BackyardLightSwitcher.Instance.TurnOn();
}
// If no motion was detected for one minute, or if it is morning or day, turn the light off.
else if (time.Subtract(LastMotionTime) > TimeSpan.FromMinutes(1) || (timeOfDay == "Morning" || timeOfDay == "Noon"))
{
    BackyardLightSwitcher.Instance.TurnOff();
}

```

As we can see, SmartHomeController delegates the responsibility of turning the light on or off to a BackyardLightSwitcher object, which implements a [Singleton pattern](https://en.wikipedia.org/wiki/Singleton_pattern). What’s wrong with this design?

To fully unit test the ActuateLights(bool motionDetected) method, we should perform interaction-based testing in addition to the state-based testing; that is, we should ensure that methods for turning the light on or off are called if, and only if, appropriate conditions are met. Unfortunately, the current design does not allow us to do that: the TurnOn() and TurnOff() methods of BackyardLightSwitcher trigger some state changes in the system, or, in other words, produce [side effects](https://en.wikipedia.org/wiki/Side_effect_(computer_science)). The only way to verify that these methods were called is to check whether their corresponding side effects actually happened or not, which could be painful.

Indeed, let’s suppose that the motion sensor, backyard lantern, and smart home microcontroller are connected into an Internet of Things network and communicate using some wireless protocol. In this case, a unit test can make an attempt to receive and analyze that network traffic. Or, if the hardware components are connected with a wire, the unit test can check whether the voltage was applied to the appropriate electrical circuit. Or, after all, it can check that the light actually turned on or off using an additional light sensor.

As we can see, unit testing side-effecting methods could be as hard as unit testing non-deterministic ones, and may even be impossible. Any attempt will lead to problems similar to those we’ve already seen. The resulting test will be hard to implement, unreliable, potentially slow, and not-really-unit. And, after all that, the flashing of the light every time we run the test suite will eventually drive us crazy!

Again, all these testability problems are caused by the bad API, not the developer’s ability to write unit tests. No matter how exactly light control is implemented, the SmartHomeController API suffers from these already-familiar issues:  
+ **It is tightly coupled to the concrete implementation**. The API relies on the hard-coded, concrete instance of BackyardLightSwitcher. It is not possible to reuse the ActuateLights(bool motionDetected) method to switch any light other than the one in the backyard.
+ **It violates the Single Responsibility Principle**. The API has two reasons to change: First, changes to the internal logic (such as choosing to make the light turn on only at night, but not in the evening) and second, if the light-switching mechanism is replaced with another one.
+ **It lies about its dependencies**. There is no way for developers to know that SmartHomeController depends on the hard-coded BackyardLightSwitcher component, other than digging into the source code.
+ **It is hard to understand and maintain**. What if the light refuses to turn on when the conditions are right? We could spend a lot of time trying to fix the SmartHomeController to no avail, only to realize that the problem was caused by a bug in the BackyardLightSwitcher (or, even funnier, a burned out lightbulb!).  

The solution of both testability and low-quality API issues is, not surprisingly, to break tightly coupled components from each other. As with the previous example, employing Dependency Injection would solve these issues; just add an ILightSwitcher dependency to the SmartHomeController, delegate it the responsibility of flipping the light switch, and pass a fake, test-only ILightSwitcher implementation that will record whether the appropriate methods were called under the right conditions. However, instead of using Dependency Injection again, let’s review an interesting alternative approach for decoupling the responsibilities.  

## Fixing the API: Higher-Order Functions  
This approach is an option in any object-oriented language that supports [first-class functions](https://en.wikipedia.org/wiki/First-class_function). Let’s take advantage of C#’s functional features and make the ActuateLights(bool motionDetected) method accept two more arguments: a pair of Action delegates, pointing to methods that should be called to turn the light on and off. This solution will convert the method into a [higher-order function](https://en.wikipedia.org/wiki/Higher-order_function):

```

public void ActuateLights(bool motionDetected, Action turnOn, Action turnOff)
{
    DateTime time = _dateTimeProvider.GetDateTime();
    
    // Update the time of last motion.
    if (motionDetected)
    {
        LastMotionTime = time;
    }
    
    // If motion was detected in the evening or at night, turn the light on.
    string timeOfDay = GetTimeOfDay(time);
    if (motionDetected && (timeOfDay == "Evening" || timeOfDay == "Night"))
    {
        turnOn(); // Invoking a delegate: no tight coupling anymore
    }
    // If no motion is detected for one minute, or if it is morning or day, turn the light off.
    else if (time.Subtract(LastMotionTime) > TimeSpan.FromMinutes(1) || (timeOfDay == "Morning" || timeOfDay == "Noon"))
    {
        turnOff(); // Invoking a delegate: no tight coupling anymore
    }
}

```

This is a more functional-flavored solution than the classic object-oriented Dependency Injection approach we’ve seen before; however, it lets us achieve the same result with less code, and more expressiveness, than Dependency Injection. It is no longer necessary to implement a class that conforms to an interface in order to supply SmartHomeController with the required functionality; instead, we can just pass a function definition. Higher-order functions can be thought of as another way of implementing Inversion of Control.

Now, to perform an interaction-based unit test of the resulting method, we can pass easily verifiable fake actions into it:  


```

[TestMethod]
public void ActuateLights_MotionDetectedAtNight_TurnsOnTheLight()
{
    // Arrange: create a pair of actions that change boolean variable instead of really turning the light on or off.
    bool turnedOn  = false;
    Action turnOn  = () => turnedOn = true;
    Action turnOff = () => turnedOn = false;
    var controller = new SmartHomeController(new FakeDateTimeProvider(new DateTime(2015, 12, 31, 23, 59, 59)));

    // Act
    controller.ActuateLights(true, turnOn, turnOff);

    // Assert
    Assert.IsTrue(turnedOn);
}

```

Finally, we have made the SmartHomeController API fully testable, and we are able to perform both state-based and interaction-based unit tests for it. Again, notice that in addition to improved testability, introducing a seam between the decision-making and action code helped to solve the tight coupling problem, and led to a cleaner, reusable API.

Now, in order to achieve full unit test coverage, we can simply implement a bunch of similar-looking tests to validate all possible cases — not a big deal since unit tests are now quite easy to implement.

## Impurity and Testability
Uncontrolled non-determinism and side effects are similar in their destructive effects on the codebase. When used carelessly, they lead to deceptive, hard to understand and maintain, tightly coupled, non-reusable, and untestable code.

On the other hand, methods that are both deterministic and side-effect-free are much easier to test, reason about, and reuse to build larger programs. In terms of functional programming, such methods are called pure functions. We’ll rarely have a problem unit testing a [pure function](https://en.wikipedia.org/wiki/Pure_function); all we have to do is to pass some arguments and check the result for correctness. What really makes code untestable is hard-coded, impure factors that cannot be replaced, overridden, or abstracted away in some other way.

Impurity is toxic: if method Foo() depends on non-deterministic or side-effecting method Bar(), then Foo() becomes non-deterministic or side-effecting as well. Eventually, we may end up poisoning the entire codebase. Multiply all these problems by the size of a complex real-life application, and we’ll find ourselves encumbered with a hard to maintain codebase full of smells, anti-patterns, secret dependencies, and all sorts of ugly and unpleasant things.  
![](/assets/articles/how-to-write-testable-code-and-why-it-matters/4.jpg)   
However, impurity is inevitable; any real-life application must, at some point, read and manipulate state by interacting with the environment, databases, configuration files, web services, or other external systems. So instead of aiming to eliminate impurity altogether, it’s a good idea to limit these factors, avoid letting them poison your codebase, and break hard-coded dependencies as much as possible, in order to be able to analyze and unit test things independently.  

##  Common Warning Signs of Hard to Test Code
> Trouble writing tests? The problem's not in your test suite. It's in your code.

Finally, let’s review some common warning signs indicating that our code might be difficult to test.

## Static Properties and Fields
Static properties and fields or, simply put, global state, can complicate code comprehension and testability, by hiding the information required for a method to get its job done, by introducing non-determinism, or by promoting extensive usage of side effects. Functions that read or modify mutable global state are inherently impure.

For example, it is hard to reason about the following code, which depends on a globally accessible property:  

```

if (!SmartHomeSettings.CostSavingEnabled) { _swimmingPoolController.HeatWater(); }

```

What if the HeatWater() method doesn’t get called when we are sure it should have been? Since any part of the application might have changed the CostSavingEnabled value, we must find and analyze all the places modifying that value in order to find out what’s wrong. Also, as we’ve already seen, it is not possible to set some static properties for testing purposes (e.g., DateTime.Now, or Environment.MachineName; they are read-only, but still non-deterministic).

On the other hand, immutable and deterministic global state is totally OK. In fact, there’s a more familiar name for this — a constant. Constant values like Math.PI do not introduce any non-determinism, and, since their values cannot be changed, do not allow any side effects:  

```

double Circumference(double radius) { return 2 * Math.PI * radius; } // Still a pure function!

```


## Singletons
Essentially, the Singleton pattern is just another form of the global state. Singletons promote obscure APIs that lie about real dependencies and introduce unnecessarily tight coupling between components. They also violate the Single Responsibility Principle because, in addition to their primary duties, they control their own initialization and lifecycle.

Singletons can easily make unit tests order-dependent because they carry state around for the lifetime of the whole application or unit test suite. Have a look at the following example:  

```

User GetUser(int userId)
{
    User user;
    if (UserCache.Instance.ContainsKey(userId))
    {
        user = UserCache.Instance[userId];
    }
    else
    {
        user = _userService.LoadUser(userId);
        UserCache.Instance[userId] = user;
    }
    return user;
}

```

In the example above, if a test for the cache-hit scenario runs first, it will add a new user to the cache, so a subsequent test of the cache-miss scenario may fail because it assumes that the cache is empty. To overcome this, we’ll have to write additional teardown code to clean the UserCache after each unit test run.

Using Singletons is a bad practice that can (and should) be avoided in most cases; however, it is important to distinguish between Singleton as a design pattern, and a single instance of an object. In the latter case, the responsibility of creating and maintaining a single instance lies with the application itself. Typically, this is handed with a factory or Dependency Injection container, which creates a single instance somewhere near the “top” of the application (i.e., closer to an application entry point) and then passes it to every object that needs it. This approach is absolutely correct, from both testability and API quality perspectives.

## The `new` Operator
Newing up an instance of an object in order to get some job done introduces the same problem as the Singleton anti-pattern: unclear APIs with hidden dependencies, tight coupling, and poor testability.

For example, in order to test whether the following loop stops when a 404 status code is returned, the developer should set up a test web server:    

```

using (var client = new HttpClient())
{
    HttpResponseMessage response;
    do
    {
        response = await client.GetAsync(uri);
        // Process the response and update the uri...
    } while (response.StatusCode != HttpStatusCode.NotFound);
}

```

However, sometimes new is absolutely harmless: for example, it is OK to create simple entity objects:    

```

var person = new Person("John", "Doe", new DateTime(1970, 12, 31));

```

It is also OK to create a small, temporary object that does not produce any side effects, except to modify their own state, and then return the result based on that state. In the following example, we don’t care whether Stack methods were called or not — we just check if the end result is correct:  

```

string ReverseString(string input)
{
    // No need to do interaction-based testing and check that Stack methods were called or not;
    // The unit test just needs to ensure that the return value is correct (state-based testing).
    var stack = new Stack<char>();
    foreach(var s in input)
    {
        stack.Push(s);
    }
    string result = string.Empty;
    while(stack.Count != 0)
    {
        result += stack.Pop();
    }
    return result;
}

```

## Static Methods

Static methods are another potential source of non-deterministic or side-effecting behavior. They can easily introduce tight coupling and make our code untestable.

For example, to verify the behavior of the following method, unit tests must manipulate environment variables and read the console output stream to ensure that the appropriate data was printed:  

```

void CheckPathEnvironmentVariable()
{

    if (Environment.GetEnvironmentVariable("PATH") != null)
    {
        Console.WriteLine("PATH environment variable exists.");
    }

    else
    {
       Console.WriteLine("PATH environment variable is not defined.");
    }

}

```

However, pure static functions are OK: any combination of them will still be a pure function. For example:  

```

double Hypotenuse(double side1, double side2) { return Math.Sqrt(Math.Pow(side1, 2) + Math.Pow(side2, 2)); }

```

## Conclusion

Obviously, writing testable code requires some discipline, concentration, and extra effort. But software development is a complex mental activity anyway, and we should always be careful, and avoid recklessly throwing together new code from the top of our heads.

As a reward, we’ll end up with clean, easy-to-maintain, loosely coupled, and reusable APIs, that won’t damage developers’ brains when they try to understand it. After all, the ultimate advantage of testable code is not only the testability itself, but the ability to easily understand, maintain and extend that code as well.

[Original Post: https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters](https://www.toptal.com/qa/how-to-write-testable-code-and-why-it-matters)