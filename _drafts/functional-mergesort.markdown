---
layout: post
title:  "Functional Merge Sort (in Swift)"
date:   2015-02-10 01:48:54
summary:    Merge sort
categories: jekyll update
---
Learn about functional merge sort.

This is great!

{% highlight swift %}
enum Choice {
    case Left
    case Right
}

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

func mergeSort<T : Comparable>(a: [T]) -> [T] {
    if a.count <= 1 {
        return a
    }

    let middle = a.count / 2
    let left = mergeSort(Array(a[0..<middle]))
    let right = mergeSort(Array(a[middle..<a.count]))
    return merge(left, right) { $0 <= $1 ? .Left : .Right }
}
{% endhighlight %}

More please :)
