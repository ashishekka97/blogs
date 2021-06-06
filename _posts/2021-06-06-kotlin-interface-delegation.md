---
title: Interface Delegation in Kotlin
cover-image: sea_almond.jpg
---

Interface Delegation is not a new thing.  While it is easy to understand and implement this design pattern, this usually involves a lot of boiler plate code, and the huge changelog that comes with it in case of future alterations. Thankfully, Kotlin does provide a beautiful construct to implement this.

<!--more-->

Before we jump directly into Kotlin's way to dealing with delegations, let us try to understand this very concept of delegation. Here's an excerpt from [Wikipedia](https://en.wikipedia.org/wiki/Delegation_pattern) (as defined by Grady Booch).

> **Delegation** is a way to make composition as powerful for reuse as inheritance. In delegation, *two* objects are involved in handling a request: a receiving object delegates operations to its **delegate**.

Just as in real life, delegation is literally like delegating / passing over your responsibility to someone else (the delegate).

### Delegating your work

Consider this simple example of building an app. We will need to work on following action items:

1. Requirement gathering
2. UI Mockups for the App
3. Code as per designs and requirements
4. Test all the functionality of the app
5. Publish the app

While it is possible to single handedly do all this and put this app to life, that would require quite a lot of hard work and time. You also need to be equipped with all the skills required the complete these action items. That is quite a lot of work.

A better way would be to rather manage the app development process by simply delegating the work to different people who have expertise in their respective skill. Thus, these action items can be handled much better and efficiently by the following people:

| Action Item                           | Delegate To   |
| :------------------------------------ | -------------:|
| Requirement Gathering                 | Product Owner |
| UI Mockups for the App                | Designer      |
| Code as per designs and requirements  | Developer     |
| Test all the functionality of the app | Tester        |
| Publish the app                       | Product Owner |

So, the Product Owner is one who is responsible for building this product. He/She will gather all the requirements for the app and simply delegate the work to respective people. Once all the skill specific action items are done, he/she will publish the app to the market.

### The Usual Way

Let's create the interfaces defining people with different skill sets.

``` kotlin
interface Designer {
    fun design()
}

interface Developer {
    fun code()
}

interface Tester {
    fun test()
}
```

Let us also consider that this app is an Android app, having a minimal UI design. We also want the app to be thoroughly tested, preferably using automation tools. The people that we now require are:

```kotlin
class MinimalDesigner : Designer {
    override fun design() {
        println("I love minimal designs!")
    }
}

class AndroidDeveloper : Developer {
    override fun code() {
        println("I develop Android Apps using Kotlin!")
    }
}

class AutomationTester : Tester {
    override fun test() {
        println("I use automation tools to test Android apps!")
    }
}
```

Now the `ProductOwner` can work on building the app, by delegating the work to people who have expertise in these skills. This would look something like this in code:

```kotlin
class ProductOwner(
    private val designer: Designer,
    private val developer: Developer,
    private val tester: Tester
) : Designer, Developer, Tester {
    
    override fun design() {
        // Delegate Designing to a designer
        designer.design()
    }
    
    override fun code() {
        // Delegate development to a developer
        developer.code()
    }
    
    override fun test() {
        // Delegate testing to a tester
        tester.test()
    }
    
    fun gatherRequirements() {
        println("Gather all the requirements.")
    }
    
    fun publishApp() {
        println("Publish app on the Play Store")
    }
}

fun main() {
    val productOwner = ProductOwner(
        MinimalDesigner(),
        AndroidDeveloper(),
        AutomationTester()
    )

    productOwner.apply {
        gatherRequirements()
        design()
        code()
        test()
        publishApp()
    }
}
```

As you can guess, delegation saved a ton of effort and the app was delivered in a relatively small time frame. In order change the implementation to let say for developing an web app instead, we can simply implement `WebDeveloper` class instead, and pass it into the constructor of `ProductOwner`.
Same goes for involving a `NeumorphicDesigner` or a `BetaTester`.

```kotlin
val productOwner = ProductOwner(
    NeumorphicDesigner(),
    WebDeveloper(),
    BetaTester()
)
```

That worked really well, but we do end up with a lot of boiler-plate code. Furthermore, if any changes are made at the interface level, we will have to carefully override and make changes in the `ProductOwner` class manually.

### The Kotlin Way

Let's see how we can reduce this boiler-plate code using Kotlin's Interface delegation construct - `by`.

```kotlin
class ProductOwner(
    private val designer: Designer,
    private val developer: Developer,
    private val tester: Tester
) : Designer by designer, Developer by developer, Tester by tester {

    fun gatherRequirements() {
        println("Gather all the requirements.")
    }
    
    fun publishApp() {
        println("Publish app on the Play Store")
    }
}

fun main() {

    val productOwner = ProductOwner(
        MinimalDesigner(),
        AndroidDeveloper(),
        AutomationTester()
    )

    productOwner.apply {
        gatherRequirements()
        design()
        code()
        test()
        publishApp()
    }
}
```

Wasn't that cool? We no longer need to override these interfaces manually.
By using the `by` keyword (no pun intended!), Kotlin will provide this implementation on behalf of us. This makes the code much more readable and concise.

In case the interfaces `Designer`, `Developer` and `Tester` have a parent interface `Employee`, then we would need to override the `Employee` related behaviour (say `eat()`) manually, else Kotlin compiler will throw error, as it does not knows which `Employee`'s behaviour it needs to consider.

```bash
Delegation.kt:80:1: error: class 'ProductOwner' must override public open fun eat(): 
Unit defined in me.ashishekka.ProductOwner because it inherits many implementations of it
class ProductOwner(...
^
```

One of the best use-case of delegates, which I recently came across is to implement inheritance like hierarchy in Kotlin's data classes (data classes [can't inherit](https://discuss.kotlinlang.org/t/data-class-inheritance/4107/5) from other data classes):

```kotlin
interface CommonDataHolder {
    val x: Int
    val y: Int
}

data class CommonData(
    override val x: Int,
    override val y: Int
) : CommonDataHolder

data class(
    val a: Int,
    private val commonData: CommonDataHolder
) : CommonDataHolder by commonData
```

### In A Glimpse
* The delegation pattern is used to delegate a class' responsibility to some other class (the delegate).
* Use the `by` keyword to delegate interface implementations in Kotlin.
* A class implementing multiple interfaces having a common parent interface, will need to manually override the parent's behaviour.

I hope that this blog post helped you in understanding Kotlin's way of dealing with delegations.  See you in the next one.

