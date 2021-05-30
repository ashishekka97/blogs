---
title: Suspension in Callbacks
cover-image: suspension_bridge.jpg
---

Using suspend functions, we can make asynchronous code to look alike any another synchronous code.
But what if we can't directly use suspend functions, and are instead dealing with good-old callbacks?

<!--more-->
### Suspend Functions

Most of us have been using `suspend` functions with _[Retrofit](https://square.github.io/retrofit/)_ to make network calls on Android. This works really great, while keeping the code _"readable like any other synchronous piece of code"_. All thanks to Retrofit's support for coroutines. Similarly, many other popular libraries also provide coroutine support.

>Suspending functions are at the center of everything coroutines. A suspending function is simply a function that can be paused and resumed at a later time. They can execute a long running operation and wait for it to complete without blocking.

All this is achieved, without even making any [spaghetti code](https://www.reddit.com/r/ProgrammerHumor/comments/chgnd3/callback_hell/) for handling asynchronousy.
Just look at this code. So concise and elegant.

{% highlight kotlin %}
private suspend fun loadUsers(): Result<List<User>> {
    val response = userService.getUsers()
    return if (response.isSuccessful) {
      Result.Success(response.body()?.data)
    } else {
      Result.Error(message = response.message())
    }
}
{% endhighlight %}

### Callbacks

Let us now suppose that we have a situation, where we have to use a particular library with no support for coroutines. The library we need to use, still uses the old callback mechanism to perform an asynchronous operation.

How can we add this little magic of coroutines, so that, using this library will not look different from all the other beautiful pieces of suspending code in our app?

Before we answer that, let us take a look at how callbacks look like.

Generally, we have an interface which will provide contract for success and error scenarios in a callback:

{% highlight kotlin %}
interface Callback<T> {
  fun onComplete(result: T)
  fun onError(e: Exception)
}
{% endhighlight %}

And we endup using this contract as:

{% highlight kotlin %}
val callback = object: Callback<Foo> {
  override onComplete(result: Foo) {
    onFetchingComplete.invoke(result)
  }
  
  override onError(e: Exception) {
    onFetchingError.invoke(e)
  }
}

SomeLibrary.setCallback(callback)
SomeLibrary.getResult()
{% endhighlight %}
    
Yuck! This will lead further and further into an exhausting loop of callbacks. We do not have a mechanism to make it _"behave in a synchronous way"_.
We need to do better. And the answer to this problem is `suspendCoroutine`.

### suspendCoroutine

We will leverage `suspendCoroutine` extension function, to return the callback result from a suspend function. This will look something like this:

{% highlight kotlin %}
suspend fun getResult() : Result<Foo> {
  return suspendCoroutine { continuation ->
    val callback = object: Callback<Foo> {
      override onComplete(result: Foo) {
        continuation.resume(Result.Success(result))
      }

      override onError(e: Exception) {
        continuation.resumeWithException(e as Throwable)
      }
    }

    SomeLibrary.setCallback(callback)
    SomeLibrary.getResult()
  }
}
{% endhighlight %}

As you can see, we have now made a _suspend function_ named - `getResult()`, which is enclosing a callback within it. It uses the **suspendCoroutine** extension function to resume this coroutine, and will eventually return the result of this computation, just like any other function.

Infact, this is how _suspend functions_ work internally. They _**suspend**_ the execution of a coroutine and _**resume**_ it when the enclosed callback is either a success or failure.
They abstract this callback flow and help in making a function behave as a normal sequential block of code. This is called **Continuation Passing Style (CPS)**, which is just another [fancy](https://youtu.be/YrrUCSi72E8?t=200) term for callbacks.

There is another extension function called - `suspendCancellableCoroutine` which adds support for cancellation.

Retrofit internally also encloses the Callable objects using **suspendCancellableCoroutine** for coroutines support.

You can refer to this great talk by [Roman Elizarov](https://github.com/elizarov), explaining all of this in detail.

<iframe width="560" height="315" src="https://www.youtube.com/embed/YrrUCSi72E8" frameborder="0" allowfullscreen></iframe>

Well, that counts for all the sorcery behind suspend functions.
I hope you learned something new today! See you in the next one. _Ciao!_