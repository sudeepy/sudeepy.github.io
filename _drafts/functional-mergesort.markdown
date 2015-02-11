---
layout: post
title:  "Functional Merge Sort (in Swift)"
date:   2015-02-11 09:17:00
summary:    Over the past several months, I have been learning a lot about Swift and functional programming. As an exercise for myself, I tried implementing the classic merge sort algorithm in as much of a functional way as I could.
categories: functional programming swift sorting
---
Over the past several months, I have been learning a lot about Swift and functional programming. As an exercise for myself, I tried implementing the classic merge sort algorithm following functional patterns. It was a pretty fun experiment, and I've documented it below.

But before we go through this, I'd like to clarify what I mean by "functional." Functional programming is a loaded term at this stage and encompasses [many many concepts](http://www.slideshare.net/ScottWlaschin/fp-patterns-buildstufflt). The key aspects that I was concerned with for this experiment were no side effects, immutability, and reusable/composable functions.

With that, let's get started.

Below is the basic, imperative implementation of merge sort. It takes in an array of {% ihighlight swift %}Comparable{% endihighlight %} items and returns them in ascending order:

{% highlight swift %}
func merge<T : Comparable>(a: [T], b: [T]) -> [T] {
    var indexA = 0
    var indexB = 0
    var c = [T]()
    while indexA < a.count && indexB < b.count {
        if a[indexA] <= b[indexB] {
            c = c + [a[indexA]]
            indexA++
        } else {
            c = c + [b[indexB]]
            indexB++
        }
    }
    c = c + Array(a[indexA..<a.count]) + Array(b[indexB..<b.count])
    return c
}

func mergeSort<T : Comparable>(a: [T]) -> [T] {
    if a.count <= 1 {
        return a
    }

    let middle = a.count / 2
    let left = mergeSort(Array(a[0..<middle]))
    let right = mergeSort(Array(a[middle..<a.count]))
    return merge(left, right)
}
{% endhighlight %}

Looking at the {% ihighlight swift %}mergeSort{% endihighlight %} function, it is already fairly functional. It only uses immutable types and has no side effects. Let's instead focus on the {% ihighlight swift %}merge{% endihighlight %} function.

To refactor the {% ihighlight swift %}merge{% endihighlight %} function, let's first recognize what it is doing: it takes two arrays and interleaves them based on sort order. This can be expressed recursively as "takes arrays A and B, picks either A[0] or B[0] based on sort order, and prepends to the interleaving of the remainder of A and B." Here's the refactored {% ihighlight swift %}merge{% endihighlight %} function:

{% highlight swift %}
func merge<T : Comparable>(a: [T], b: [T]) -> [T] {
    if a.count == 0 {
        return b
    }
    if b.count == 0 {
        return a
    }

    if a[0] <= b[0] {
        return [a[0]] + merge(Array(a[1..<a.count]), b)
    } else {
        return [b[0]] + merge(a, Array(b[1..<b.count]))
    }
}
{% endhighlight %}

With this new {% ihighlight swift %}merge{% endihighlight %} function, we now have eliminated all mutable variables and have something with a more declarative implementation. With this, our merge sort has purely immutable types.

(Note: this post isn't concerned with performance; the above implementation is certainly susceptible to stack overflows. This might be a topic for a future post.)

The next issue to tackle is our merge sort's reusability. Right now, if you want to sort something in ascending order, it works great, but if you want descending, you're out of luck. What we need to do is extract the comparison into a block that can be passed in. This way our merge sort is not tied to a particular sort order.

{% highlight swift %}
func merge<T : Comparable>(a: [T], b: [T], eval: (T, T) -> Bool) -> [T] {
    if a.count == 0 {
        return b
    }
    if b.count == 0 {
        return a
    }

    if eval(a[0], b[0]) {
        return [a[0]] + merge(Array(a[1..<a.count]), b, eval)
    } else {
        return [b[0]] + merge(a, Array(b[1..<b.count]), eval)
    }
}

func mergeSort<T : Comparable>(a: [T], eval: (T, T) -> Bool) -> [T] {
    if a.count <= 1 {
        return a
    }

    let middle = a.count / 2
    let left = mergeSort(Array(a[0..<middle]), eval)
    let right = mergeSort(Array(a[middle..<a.count]), eval)
    return merge(left, right, eval)
}
{% endhighlight %}

Now that the comparison has been abstracted, we can make another tweak: we can drop the {% ihighlight swift %}Comparable{% endihighlight %} requirement on T. Now we can sort any arbitary array of items. This makes our merge sort even more reusable.

One aspect of functional programming I really enjoy is its emphasis on expressive syntax. Each line of code is meant to be self-contained and declarative, which should make the code more readable. One important aspect of expressive syntax is the careful use of types. This subject alone is worthy of another post, as [others have done](https://swiftcast.tv/articles/the-design-of-types).

In our case, the issue is with the return type of the {% ihighlight swift %}eval{% endihighlight %} block. If you look at our {% ihighlight swift %}merge{% endihighlight %} function, we call {% ihighlight swift %}eval(a[0], b[0]){% endihighlight %}. If it returns true, we pick {% ihighlight swift %}a[0]{% endihighlight %} as the first element, and if it returns false, we pick {% ihighlight swift %}b[0]{% endihighlight %}. This seems arbitrary. Why can't it be the other way around? And also, what guarantees the caller who defined {% ihighlight swift %}eval{% endihighlight %} followed this contract? The problem is that we're using documentation to express a requirement instead of using the compiler to enforce it.

Instead, we'll introduce a new type called {% ihighlight swift %}Choice{% endihighlight %}. All {% ihighlight swift %}Choice{% endihighlight %} does is describe a choice between one or the other, a or b, left or right:

{% highlight swift %}
enum Choice {
    case Left
    case Right
}
{% endhighlight %}

Now, we'll just make {% ihighlight swift %}eval{% endihighlight %} return a {% ihighlight swift %}Choice{% endihighlight %} rather than a {% ihighlight swift %}Bool{% endihighlight %} and update our {% ihighlight swift %}merge{% endihighlight %} function accordingly:

{% highlight swift %}
func merge<T>(a: [T], b: [T], eval: (T, T) -> Choice) -> [T] {
    if a.count == 0 {
        return b
    }
    if b.count == 0 {
        return a
    }

    switch eval(a[0], b[0]) {
    case .Left:
        return [a[0]] + merge(Array(a[1..<a.count]), b, eval)

    case .Right:
        return [b[0]] + merge(a, Array(b[1..<b.count]), eval)
    }
}
{% endhighlight %}

By introducing a new type, there is a much stronger connection between {% ihighlight swift %}eval{% endihighlight %}'s return type and {% ihighlight swift %}merge{% endihighlight %}'s logic flow. In addition, the return type's meaning is much clearer to callers.

With that, we have made both our functions very functional. They have no side effects, use immutable types, and are reusable. In fact, the {% ihighlight swift %}merge{% endihighlight %} function is reusable even outside the context of merge sort. If you look at it again, all it's doing is taking two arrays and interleaving their contents based off of some external criteria. For example, outside of merge sort, another conceivable use case would be to simulate a [riffle shuffle](http://en.wikipedia.org/wiki/Shuffling#Riffle) for a deck of cards.

Here is a sample implementation of a riffle shuffle using our {% ihighlight swift %}merge{% endihighlight %} function:

{% highlight swift %}
struct Card {
    enum Suit {
        case Heart
        case Diamond
        case Club
        case Spade
    }

    enum Value {
        case Two
        case Three
        case Four
        case Ace
    }

    let value: Value
    let suit: Suit
}

func riffleShuffle(cards: [Card]) -> [Card] {
    let middle = cards.count / 2
    let left = Array(cards[0..<middle])
    let right = Array(cards[middle..<cards.count])
    return merge(left, right) { _ in
        return arc4random_uniform(2) == 0 ? .Left : .Right
    }
}
{% endhighlight %}

With the advent of Scala and Swift in front-end development, functional programming concepts are becoming more and more popular. I encourage you to try revisiting these sorts of tried-and-true imperative implementations and rethinking them with a more functional style.