---
id: dI8ZZxMwdC6iOpPdEfaIB
title: Observables
desc: ''
updated: 1634616464357
created: 1630999552309
---
# **`</> Observables`**

## **What is an observable?**
An Observable is a sequence emitting `events` asynchronously over a period of time. 

**`Events`** can be any values:
- Number;
- Instances of custom type or;
- System events

---

## **Lifecycle of an observable**
>1 . An observable emits _next_ events, containing elements.

>2 . It can continue to do this until a terminating event is emitted: an _Error_ event or _Completed_ event.

>3 . Once an observable is terminated, it can no longer emit events.

### Events implemented in the RxSwift source code:
```swift
// Represents a sequence event.
//
// Sequence grammar:
// **next\* (error | completed)**

public enum Event<Element> {
    // Next element is produced
    case next(Element)

    // Sequence terminated with an error
    case error(Swift.Error)

    // Sequence completed successfully
    case completed
}
```
---
## **Creating observables**
### With _`just`_ or _`of`_  operator:
Create an observable sequence containing `just a single element`.

```swift
let one = 1
let two = 2
let three = 3

let observable = Observable<Int>.just(one)

let observable = Observable.of(one, two, three)

// It creates the array of [one, two, three] as a single element
let observable 2 = Observable.of([one, two, three])
```

### With _`from`_ operator:
Create an observable of `individual element from an array` of typed elements. It only takes `array`.

```swift
let observable4 = Observable.from([one, two, three])
```

### With _`empty`_ operator:
What use is an `empty` observable? They’re handy when you want to return an observable that immediately terminates or intentionally has zero values.

```swift
let observable = Observable<Void>.empty()
```

### With _`never`_ operator:
As opposed to the empty operator, the `never` operator creates an observable that doesn't emit anything and never terminates. It can be used to represent an infinite duration.

```swift
let observable = Observable<Void>.never()
```

### With _`range`_ operator:
Creating an observable with range operator takes a _start_ integer value and a _count_ of sequential integers to generate.

```swift
let observable = Observable<Int>.range(start: 1, count: 10)
```
---

### With `create` operator:
Use `create` operator to create an observable and at the same time specify all the events the observable will emit to subscribers.

```swift
Observable<String>.create { observer: AnyObserver<String> in 

    observer.onNext("1")

    observer.onCompleted()

    observer.onNext("?")
    
    return Disposables.create()
}
```

> `AnyObserver<T>` is a generic type that facilitates adding values onto an observable sequence.

## **Subscribing to observables**
> Note: an observable `won't send` events, or perform any work, until it has a subscriber.

Example:
```swift
let one = 1
let two = 2
let three = 3

let observable = Observable.of(one, two, three)

// Subscribe to the observable
observable.subscribe(
    onNext: { event in 
        print(event)
        if let element = event.element {
            print(event.element)
        }
    },

    onCompleted: {
        print("Completed")
    }
)
```

Result:
```swift
next(1)
1
next(2)
2
next(3)
3
completed
```
---

## **Disposing and terminating**
To manually terminate an observable, we use _**`dispose`**_ operator.

```swift
let observable = Observable.of("A", "B", "C")
let subscription = observable.subscribe { event in
    print(event)
}

subscription.dispose()
```

### With _`DisposeBag`_ type:
A dispose bag holds disposables. Simply add each disposable to a dispose bag so that it can automatically dispose each of them resided within the bag.

```swift
let disposeBag = DisposeBag()

Observable.of("A", "B", "C")
    .subscribe {
        print($0)
    }
    .diposed(by: disposeBag)
```

---
## **Creating observable factories**
> Create a flexible observable with _**`deferred`**_ operator.

```swift
let disposeBag = DisposeBag()
var flip = false
let factory: Observable<Int> = Observable.deferred {

    flip.toggle()

    if flip {
      return Observable.of(1, 2, 3)
    } else {
      return Observable.of(4, 5, 6)
    }
}

for _ in 0...3 {
    factory.subscribe(onNext: {
        print($0, terminator: "")
    })
    .disposed(by: disposeBag)
    print() 
}
```
Result
```swift
123
456
123
456
```

---
## **Using Traits**
Traits are observables with a narrower set of behaviors than regular observables. Using traits can help make your code more intuitive. 

>There are `three` kinds of traits in RxSwift: `Single`, `Maybe`, `Completable`.

### **`Single`**:
Single will emit just _once_ either a _`success(value)`_ or _`error(error)`_ event. _`success(value)`_ is actually a combination of the next and completed events. 

> <> It is useful for one-time processes that will either succeed and yield a value or fail such as when downloading data or loading it from disk.

### **`Completable`**: 
> A Completable will only emit a _`completed`_ or _`error(error)`_ event. It will not emit any values.

> <> It is useful when you only care that an operation completed successfully or failed, such as a file write.

### **`Maybe`**:
A Maybe is a mashup of a Single and Completable. It can either emit a _`success(value)`_, _`completed`_ or _`error(error)`_. 

> <> It is useful with an operation that could either succeed or fail, and optionally return a value on success.

---

# **`</> Subjects`**
## **What are subjects?**
Subjects act as both an observable and an observer. They can receive events and also be subscribed to.

> There are `four` types of subjects: `PublishSubject`, `BehaviorSubject`, `ReplaySubject`, and `AsyncSubject`.

### **`PublishSubject`**:
> Starts empty and only emits new elements to subscribers.

### **`BehaviorSubject`**:
> Starts with an initial value and replays it or the latest element to new subscribers.

### **`ReplaySubject`**:
> Initialized with a buffer size and will maintain a buffer of elements up to that size and replay it to new subscribers.

### **`AsyncSubject:`**:
> It emeits _only_ the last next event in the sequence, and only when the subject receives a completed event. 

Additional:
> RxSwift also provides a concept called `Relays`. RxSwift provides two of these, named `PublishRelay` and `BehaviorRelay`. These wrap their respective subjects, but only accept and relay next events. You cannot add a completed or error event onto relays at all, so they’re great for non-terminating sequences.
---

## **Publish Subjects**:
`Publish Subjects` come with in handy when you simply want subscribers to be notified of new events from the point at which they subscribed, until either they unscribe or the subject has terminated with a _completed_ or _error_ event.

![](/assets/images/2021-09-16-14-44-20.png)

Note:

> When a publish subject receives a `completed` or `error` event, also known as `a stop event`, it will emit that stop event to new subscribers and it will no longer emit next events. However, it will re-emit its stop event to future subscribers.

Example:
```swift
let subject = PublishSubject<String>()

// Add a .next event to the publish subject
subject.on(.next("add next event to the publish subject"))

// Another way of adding event to the publish subject
subject.onNext("Another way of adding event")

// Add a .completed event
subject.onCompleted()
```
---
## **Behavior Subjects**:
`Behavior subjects` work similaryly to _publish subjects_, except they will **_`replay`_** the latest next event to next subscribers.

![](/assets/images/2021-09-16-15-20-36.png)

Example 1:
```swift
let subject = BehaviorSubject(value: "initial value")

subject
    .subscribe {
        print("1) \(event)")
    }
```

Result:
```swift
1) initial value
```

Example 2:
```swift
let subject = BehaviorSubject(value: "initial value")

subject.onNext("Y")

subject
    .subscribe {
        print("1) \(event)")
    }
```

Result:
```swift
1) Y
```

Example 3:
```swift
let subject = BehaviorSubject(value: "initial value")

subject.onNext("Y")

subject
    .subscribe {
        print("1) \($0)")
    }

subject.onError(MyError.anError)

subject
  .subscribe {
    print(label: "2) \($0)")
  }
```

Result:
```swift
// updated value of the behavior subject; previous value was "Y"
1) anError 

// latest value of the behavior subject
2) anError 
```
---
## **Replay Subjects**
`Replay subjects` will temporarily _`cache`_, or _`buffer`_, the latest _`elements`_ they emit, _`up to a specified size`_ of your choosing. They will then **_`replay`_** that buffer to new subscribers.

![](/assets/images/2021-09-16-16-00-12.png)

Note:
> When using a replay subject, the buffer is held in memory.

Example 1:
```swift
let subject = ReplaySubject<String>.create(bufferSize: 2)

subject.onNext("1")
subject.onNext("2")
subject.onNext("3")

subject
    .subscribe {
      print(label: "1) \($0)")
    }

subject
    .subscribe {
      print(label: "2) \($0)")
    }
```

Result:
```swift
1) 2
1) 3
2) 2
2) 3
```

Example 2:
```swift
let subject = ReplaySubject<String>.create(bufferSize: 2)

subject.onNext("1")
subject.onNext("2")
subject.onNext("3")

subject
    .subscribe {
      print(label: "1) \($0)")
    }

subject
    .subscribe {
      print(label: "2) \($0)")
    }

subject.onNext("4")

subject
  .subscribe {
    print(label: "3)", event: $0)
  }
```

Result:
```swift
1) 4 // already subscribed -> get updated value
2) 4 // already subscribed -> get updated value

// newly subscribed -> get last two buffered values
3) 3
3) 4
```

Example 3:
```swift
let subject = ReplaySubject<String>.create(bufferSize: 2)

subject.onNext("1")
subject.onNext("2")
subject.onNext("3")

subject
    .subscribe {
      print(label: "1) \($0)")
    }

subject
    .subscribe {
      print(label: "2) \($0)")
    }

subject.onNext("4")
subject.onError(MyError.anError)

subject
  .subscribe {
    print(label: "3) \($0)")
  }
```

Result:
```swift
1) 4 // already subscribed -> get updated value
2) 4 // already subscribed -> get updated value
1) anError // already subscribed -> get updated value
2) anError // already subscribed -> get updated value

// newly subscribed -> get last two buffered values & 
// stop event .onError(anError)
3) 3 
3) 4
3) anError
```

Example 4:
```swift
let subject = ReplaySubject<String>.create(bufferSize: 2)

subject.onNext("1")
subject.onNext("2")
subject.onNext("3")

subject
    .subscribe {
      print(label: "1) \($0)")
    }

subject
    .subscribe {
      print(label: "2) \($0)")
    }

subject.onNext("4")
subject.onError(MyError.anError)

// call on dispose() to prevent subject from broadcasting to new subscriber after stop event.
subject.dispose()

subject
  .subscribe {
    print(label: "3) \($0)")
  }
```

Result:
```swift
1) 4 // already subscribed -> get updated value
2) 4 // already subscribed -> get updated value
1) anError // already subscribed -> get updated value
2) anError // already subscribed -> get updated value

// newly subscribed after the replay subject was disposed
 3) Object `RxSwift...ReplayMany<Swift.String>` was already
disposed.
```
---
## **Working with Relays**
Unlike other subjects, we add a value onto a relay by using `accept(_:)` method because relays can only accept values; you `cannot add` an _`error`_ or _`completed`_ event onto them.

> `PublishRelay` and `BehaviorRelay` are just wrapper of `PublishSubject` and `BehaviorSubject`, respectively. Relays are guaranteed to never terminate.

Example - PublishRelay:
```swift
let relay = PublishRelay<String>()

let disposeBag = DisposeBag()

relay.accept("Hello world!")

relay
  .subscribe(onNext: {
    print($0) 
  })
  .disposed(by: disposeBag)

relay.accept("1")
```

Result:
```swift
1
```

Example 1 - BehaviorRelay:
```swift
let relay = BehaviorRelay(value: "Initial value")

let disposeBag = DisposeBag()

relay.accept("New initial value")

relay
  .subscribe {
    print(label: "1) \($0)")
  }
  .disposed(by: disposeBag)


relay.accept("1")

relay
  .subscribe {
    print(label: "2) \($0)")
  }
  .disposed(by: disposeBag)

relay.accept("2")

print("Current value of relay: \(relay.value))

```

Result:
```swift
1) New initial value 
1) 1
2) 1
1) 2
2) 2

Crrent value of relay: 2
```

> Note: `BehaviorRelay` let you directly access their current value through `.value` property.
---

# **`</> Operators`**
Operators are the building blocks of Rx, which you can use to transform, process, and react to events emitted by observables.

## **`Filtering Operators`**
## **Ignoring operators**: 
### With _`ignoreElements`_ operator:
It will ignore all next events and only allow stop events, such as completed or error events.

![](/assets/images/2021-09-21-09-53-05.png)

Example:
```swift
let strikes = PublishSubject<String>()
let disposeBag = DisposeBag()

strikes
    .ignoreElements()
    .subscribe { _ in
      print("You're out!")
    }
    .disposed(by: disposeBag)
    
strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")

strkes.onCompleted()
```

Result:
```swift
You're out!
```

### With _`elementAt`_ operator:
We can take a specific index of element we want to receive and ignore everything else.

![](/assets/images/2021-09-21-10-08-59.png)

Example:
```swift
let strikes = PublishSubject<String>()
let disposeBag = DisposeBag()

strikes
    .elementAt(2)
    .subscribe(onNext: { _ in
      print("You're out!")
    })
    .disposed(by: disposeBag)

strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")
```

Result:
```swift
You're out!
```

Note:
> An interesting fact about _`element(at:)`_. As soon as an element is emitted at the provided index, the subscription is _`terminated`_.

### With _`filter`_ operator:
We can apply a conditional constraint to the emitted elements.

![](/assets/images/2021-09-21-10-17-56.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6)
    .filter { $0.isMultiple(of: 2) }
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)
```

Result:
```swift
2
4
6
```

## **Skipping operators**:
### With _`skip`_ operator:
It let you ignore the first _n_ elements of the emitted elements, where _n_ is the number you pass as its parameter.

![](/assets/images/2021-09-21-10-51-02.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of("A", "B", "C", "D", "E", "F")
    .skip(3)
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)
```

Result:
```swift
D
E
F
```

### With _`skipWhile`_ operator:
Unlike filter, _`skipWhile`_ only skips up until something is not skipped, and then it lets everything else through from that point on.

- return `true` will cause the element to be skipped
- return `false` will let it through. 

![](/assets/images/2021-09-21-11-04-31.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of(2, 2, 3, 4, 4)
    // 2
    .skipWhile { $0.isMultiple(of: 2) }
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)
```

Result:
```swift
3
4
4
```

### With _`skipUntil`_ operator:
It skips elements from the source observable - the observable we're subscribing to - until some other trigger observable emits.

![](/assets/images/2021-09-21-11-16-01.png)

Example:
```swift
let disposeBag = DisposeBag()

let subject = PublishSubject<String>()
let trigger = PublishSubject<String>()

subject
    .skipUntil(trigger)
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)

subject.onNext("A")
subject.onNext("B")

trigger.onNext("X")
subject.onNext("C")
```

Result:
```swift
C
```
## **Taking operators**: 
### With _`take`_ operator:
It takes the first _n_ elements, where _n_ is the number you pass as its parameter.

![](/assets/images/2021-09-21-11-23-00.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6)
    .take(3)
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)
```

Result:
```swift
1
2
3
```

### With _`takeWhile`_ operator:
It works similarly to skipWhile, except we're taking instead of skipping.

![](/assets/images/2021-09-21-11-26-00.png)

> `enumerated()` operator helps extract the index of the element being emitted.

Example:
```swift
let disposeBag = DisposeBag()

Observable.of(2, 2, 4, 4, 6, 6)
    .enumerated()
    .takeWhile { index, integer in
      integer.isMultiple(of: 2) && index < 3
    }
    .map(\.element)
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
```

Result:
```swift
2
2
4
```

### With _`takeUntil`_ operator:
It will take elements until the predicate is met. It also takes a _behavior_ argument for its first parameter that specifies if you want to include or exclude the last element matching the predictate.

![](/assets/images/2021-09-21-11-34-50.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of(1, 2, 3, 4, 5)
    .takeUntil(.inclusive) { $0.isMultiple(of: 4) }
    .subscribe(onNext: {
      print($0) 
    })
  .disposed(by: disposeBag)
```

Result:
```swift
// takeUntil(.inclusive)
1
2
3
4

// takeUntil.(.exclusive)
1
2
3
```

> _`takeUntil`_ also works with trigger observable.

![](/assets/images/2021-09-21-11-42-02.png)

Example:
```swift
let disposeBag = DisposeBag()

let subject = PublishSubject<String>()
let trigger = PublishSubject<String>()

subject
    .takeUntil(trigger)
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)

subject.onNext("1")
subject.onNext("2")

trigger.onNext("X")
subject.onNext("3")
```

Result:
```swift
1
2
```

## **Distinct operators**:
### With _`distinctUntilChanged`_ operator:
It prevents duplicate contiguous items from getting through, but only those right next to each other.

![](/assets/images/2021-09-21-13-52-11.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of("A", "A", "B", "B", "A")
    .distinctUntilChanged()
    .subscribe(onNext: {
      print($0) 
    })
    .disposed(by: disposeBag)
```

Result:
```swift
A
B
A
```

> _`distinctUntilChanged(_:)`_ operator let us provide our own custom equatable logic. We must pass a comparer as a paramerter.

![](/assets/images/2021-09-21-13-58-48.png)

Example:
```swift
let disposeBag = DisposeBag()

let formatter = NumberFormatter()
formatter.numberStyle = .spellOut

Observable<NSNumber>.of(10, 110, 20, 200, 210, 310)
    .distinctUntilChanged { a, b in
      guard
        let aWords = formatter
          .string(from: a)?
          .components(separatedBy: " "),
        let bWords = formatter
          .string(from: b)?
          .components(separatedBy: " ")
        else {
          return false
      }

      var containsMatch = false
      for aWord in aWords where bWords.contains(aWord) {
        containsMatch = true
        break
      }
      return containsMatch
    }
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
```

Result
```swift
10
20
200
```

## **Other operators**:
### With _`share()`_ operator:
It ensures that an observable does not produce a new subscription every time a new observer subscribes to it. It'd rather shares an existing subscription to all the observers.

### With _`throttle(_:scheduler:)`_ operator:
The operator filters any elements followed by another element within the specified time interval. It makes sure that _no two elements_ are emitted in less then dueTime.

---
## **`Transforming Operators`**
### With _`toArray()`_ operator:
It _converts_ an observable _sequence of elements_ into _an array_ of those elements once the observable completes and _return a Single_.

![](/assets/images/2021-09-22-14-29-59.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of("A", "B", "C")
    .toArray()
    .subscribe(onSuccess: {
      print($0) 
    })
    .disposed(by: disposeBag)
```

Result
```swift
["A", "B", "C"]
```

### With _`map`_ operator:
Like _map_ in Swift's standard library, excepts that this operator works with observables, it converts the elements emitted by observables.

![](/assets/images/2021-09-22-14-47-11.png)

Example:
```swift
let disposeBag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6)
  .enumerated()
  .map { index, integer in
    index > 2 ? integer * 2 : integer
  }
  .subscribe(onNext: {
    print($0)
  })
  .disposed(by: disposeBag)
```

Result:
```swift
1
2
3
8
10
12
```

### With _`compactMap`_ operator:
The operator is a combination of the _map_ and _filter_ operators that specifically filters out _`nil`_ values. It helps retrieve unwrapped value and filter out nil.

Example:
```swift
let disposeBag = DisposeBag()

Observable.of("To", "be", nil, "or", "not", "to", "be", nil)
    .compactMap { $0 }
    .toArray()
    .map { $0.joined(separator: " ") }
    .subscribe(onSuccess: {
      print($0)
    })
    .disposed(by: disposeBag)
```

Result:
```swift
To be or not to be
```

### With _`flatMap`_ operator:
The operator projects and transforms an observable value of an observable, and then flattens it down to a target observable.

![](/assets/images/2021-09-22-15-14-13.png)

Example: 
```swift
struct Student {
  let score: BehaviorSubject<Int>
}

let disposeBag = DisposeBag()

let laura = Student(score: BehaviorSubject(value: 80))
let charlotte = Student(score: BehaviorSubject(value: 90))

let student = PublishSubject<Student>()

student
  .flatMap {
    $0.score 
  }
  .subscribe(onNext: {
    print($0)
  })
  .disposed(by: disposeBag)

// 1
student.onNext(laura)

// 2: Change laura's score
laura.score.onNext(85)

// 3
student.onNext(charlotte)

// 4
charlotte.score.onNext(100)
```

Result
```swift
// 1
80
// 2
85
// 3
90
// 4
100
```

### With _`flatMapLatest`_ operator:
Similar to flatMap, it projects and transforms changes from the most recent observable and unsubscribes to previous observables.

![](/assets/images/2021-09-22-15-36-40.png)

Example:
```swift
let disposeBag = DisposeBag()

let laura = Student(score: BehaviorSubject(value: 80))
let charlotte = Student(score: BehaviorSubject(value: 90))

let student = PublishSubject<Student>()

student
  .flatMapLatest {
    $0.score
  }
  .subscribe(onNext: {
    print($0)
  })
  .disposed(by: disposeBag)

student.onNext(laura)
laura.score.onNext(85)
student.onNext(charlotte)

// 1
laura.score.onNext(95)
charlotte.score.onNext(100)
```

Result
```swift
80
85
90
100
```

### With _`materialize`_ operator:
Use the operator to wrap each event emitted by an observable in an observable.

![](/assets/images/2021-09-22-16-14-50.png)

### With _`dematerialize`_ operator:
The operator converts a materialized observable back into its original form.

![](/assets/images/2021-09-22-16-17-18.png)

---
## **`Combining Operators`**
### With _`startWith`_ operator:
The operator guarantee that an observer receives an initial value.

![](/assets/images/2021-09-29-10-17-42.png)

Example:
```swift
let numbers = Observable.of(2, 3, 4)

let observable = numbers.startWith(1)
_ = observable.subscribe(onNext: { value in
  print(value)
})
```

Result:
```swift
1
2
3
4
```

### With _`concat`_ operator:
The operator chains two sequences.

![](/assets/images/2021-09-29-10-25-03.png)

Static function: Observable.concat(_ :)

Example:
```swift
let first = Observable.of(1, 2, 3)
let second = Observable.of(4, 5, 6)

let observable = Observable.concat([first, second])

observable.subscribe(onNext: { value in
    print(value)
})
```

Instance function: .concat(_ :)

Example:
```swift
let germanCities = Observable.of("Berlin", "Münich",
"Frankfurt")
  let spanishCities = Observable.of("Madrid", "Barcelona",
"Valencia")

let observable = germanCities.concat(spanishCities)
_ = observable.subscribe(onNext: { value in
    print(value)
})
```

Result:
```swift
Berlin,
Münich,
Frankfurt,
Madrid,
Barcelona,
Valencia
```

> `Note:` Observable sequences are strongly typed. You can only concatenate sequences whose elements are of the same type!

### With _`concatMap(_:)`_ operator:
Closely related to flatMap(_:), the operator concatenates two observable sequences and relays the values each sequence emits into the resulting observable sequence and gurantee sequential order.

Example:
```swift
let sequences = [
    "German cities": Observable.of("Berlin", "Münich", "Frankfurt"),
    "Spanish cities": Observable.of("Madrid", "Barcelona", "Valencia")
]

let observable = Observable.of("German cities", "Spanish cities")
  .concatMap { country in sequences[country] ?? .empty() }

_ = observable.subscribe(onNext: { string in
      print(string)
})
```

### With _`merge`_ operator:
The merge operator subscribes to each of the sequences it receives and emits the elements as soon as they arrive - there is no predefined order.

![](/assets/images/2021-09-30-10-29-30.png)

Example:
```swift
let left = PublishSubject<String>()
let right = PublishSubject<String>()

let source = Observable.of(left.asObservable(), right.asObservable())
let observable = source.merge()

_ = observable.subscribe(onNext: { value in
  print(value)
})

var leftValues = ["Berlin", "Munich", "Frankfurt"]
var rightValues = ["Madrid", "Barcelona", "Valencia"]
repeat {
  switch Bool.random() {
    case true where !leftValues.isEmpty:
        left.onNext("Left:  " + leftValues.removeFirst())
    case false where !rightValues.isEmpty:
        right.onNext("Right: " + rightValues.removeFirst())
    default:
        break
    }
} while !leftValues.isEmpty || !rightValues.isEmpty

left.onCompleted()
right.onCompleted()

```

Result
```swift
Right: Madrid
Left:  Berlin
Right: Barcelona
Right: Valencia
Left:  Munich
Left:  Frankfürt
```

> When and how .merge() completes ?
- merge() completes after its source sequence completes and all inner sequences have completed.

- If any of the sequences emit and error, the merge() observable immediately relay the error, then terminates.

### With _`combineLatest`_ operator:
The operator combines _`values`_ from serveral observable sequences, whose types could possibly be the same or different, and it then returns an observable whose type is the closure return type.

![](/assets/images/2021-09-30-11-27-52.png)

Example:
```swift
let left = PublishSubject<String>()
let right = PublishSubject<String>()

let observable = Observable.combineLatest(left, right) {
  lastLeft, lastRight in
  "\(lastLeft) \(lastRight)"
}

_ = observable.subscribe(onNext: { value in
  print(value)
})

print("> Sending a value to Left")
left.onNext("Hello,")

print("> Sending a value to Right")
right.onNext("world")

print("> Sending another value to Right")
right.onNext("RxSwift")

print("> Sending another value to Left")
left.onNext("Have a good day,")

left.onCompleted()
right.onCompleted()
```

> Note: The operators waits for all its observables to emit one element before starting to call the closure we specify. 

Example 2:
```swift
let choice: Observable<DateFormatter.Style> =
Observable.of(.short, .long)

let dates = Observable.of(Date())

let observable = Observable.combineLatest(choice, dates) {
    format, when -> String in
    let formatter = DateFormatter()
    formatter.dateStyle = format
    return formatter.string(from: when)
}

_ = observable.subscribe(onNext: { value in
    print(value)
})

// This example demonstrates automatic updates of on-screen values when the user settings change. Think about all the manual updates you’ll remove with such patterns!
```
> Note: combineLatest(_:) operator completes only when the last of its inner sequences completes.

### With _`zip`_ operator:
What makes zip operator different from the combineLatest is that zip emits values by pariing each _`next value`_ of each observable at the same logical position.

![](/assets/images/2021-09-30-16-17-53.png)

Example:
```swift
enum Weather {
  case cloudy
  case sunny 
}

let left: Observable<Weather> = Observable.of(.sunny, .cloudy, .cloudy, .sunny)
let right = Observable.of("Lisbon", "Copenhagen", "London", "Madrid", "Vienna")

let observable = Observable.zip(left, right) { weather, city in
    return "It's \(weather) in \(city)"
}

_ = observable.subscribe(onNext: { value in
  print(value)
})
```

Result:
```swift
It's sunny in Lisbon
It's cloudy in Copenhagen
It's cloudy in London
It's sunny in Madrid
```
---

## **`Triggers`**
### With _`withLatestFrom(_:)`_ operator:
The operator emtis the latest values of an observable but only when a particular triggers occurs.

![](/assets/images/2021-09-30-16-37-31.png)

Example:
```swift
let button = PublishSubject<Void>()
let textField = PublishSubject<String>()

let observable = button.withLatestFrom(textField)
_ = observable.subscribe(onNext: { value in
  print(value)
})

textField.onNext("Par")
textField.onNext("Pari")
textField.onNext("Paris")
button.onNext(())
button.onNext(())
```

Result:
```swift
Paris
Paris
```

### With _`sample(_:)`_ operator:
Like _withLatestFrom(_:) operator, the operator emits the latest value from the other observable, but only if it arrived since the last trigger. If no new data arrived, the operator won't emit anything.

![](/assets/images/2021-09-30-16-46-46.png)

Example:
```swift
let button = PublishSubject<Void>()
let textField = PublishSubject<String>()

let observable = textField.sample(button)
```

## **`Switches`**
### With _`amb(_:)`_ operator:
The operator subscribes to each observables. It waits for any of them to emit an element, then unsubscribes from the other one. After that, it only relyas elements from the first active observable.

![](/assets/images/2021-09-30-16-58-53.png)

Example: 
```swift
let left = PublishSubject<String>()
let right = PublishSubject<String>()

let observable = left.amb(right)
_ = observable.subscribe(onNext: { value in
  print(value)
})

left.onNext("Lisbon")
right.onNext("Copenhagen")
left.onNext("London")
left.onNext("Madrid")
right.onNext("Vienna")
left.onCompleted()
right.onCompleted()
```

### With _`switchLatest()`_ operator:
The operator only emits the latest value of the most recently active observable among others.

![](/assets/images/2021-09-30-17-01-03.png)

Example:
```swift
let one = PublishSubject<String>()
let two = PublishSubject<String>()
let three = PublishSubject<String>()
let source = PublishSubject<Observable<String>>()

let observable = source.switchLatest()
let disposable = observable.subscribe(onNext: { value in
  print(value)
})

source.onNext(one)
one.onNext("Some text from sequence one")
two.onNext("Some text from sequence two")

source.onNext(two)
two.onNext("More text from sequence two")
one.onNext("and also from sequence one")

source.onNext(three)
two.onNext("Why don't you see me?")
one.onNext("I'm alone, help me")
three.onNext("Hey it's three. I win.")

source.onNext(one)
one.onNext("Nope. It's me, one!")

disposable.dispose()
```

Result:
```swift
Some text from sequence one
More text from sequence two
Hey it's three. I win.
Nope. It's me, one!
```
---
## **`Combining element within a sequence`**
### With _`reduce(_:_:)`_ operator:
The operator combine each element within a observable sequence with specified criteria set up in its closure.

![](/assets/images/2021-10-04-09-51-23.png)

Example:
```swift
let source = Observable.of(1, 3, 5, 7, 9)

let observable = source.reduce(0) { summary, newValue in
  return summary + newValue
}

// The operator “accumulates” a summary value. It starts with the initial value you provide (in this example, you start with 0). Each time the source observable emits an item, reduce(_:_:) calls your closure to produce a new summary. When the source observable completes, reduce(_:_:) emits the summary value, then completes.
```

> `Note`: reduce(_: _:) produces its summary value only when the source observable completes.

### With _`scan(_:accumulator:)`_ operator:
Closely related to reduce(), the operator invokes the closure each time the source observable emits an element. It passes the running value along with the new element, and the closure returns the new accumulated value.

![](/assets/images/2021-10-04-09-58-10.png)

Example:
```swift
let source = Observable.of(1, 3, 5, 7, 9)

let observable = source.scan(0, accumulator: +)
_ = observable.subscribe(onNext: { value in
  print(value)
})
```

Result:
```swift
1
4
9
16
25
```
---
# **`</> Time-based Operators`**
## **`Buffering operators`**

The operator will either replay past elements to new subscribers, or buffer them and deliver them in bursts. They allow you to control how and when past and new elements get delivered.

### With _`.replay()`_ operator:
This operator creates a new sequence which records the last replayedElements emitted by the source observable. Every time a new observer subscribes, it immediately receives the buffered elements, if any, and keeps receiving any new element like a normal observer does.

### With _`.delaySubscription(_:scheduler:)`_ operator:
It delay the time a subscriber starts receiving elements from its subscription.

### With _`.delay(_:scheduler:)`_ operator:
The operator delays the receiption of the emitted items. You simply see the items arrive with a delay.

### With _`.timeout()`_ operator:
The operator semantically distinguish an acutal timer from a timeout condition. Therefore, when a timeout operator fires, it emits an RxError.TimeoutError error event; if not caught, it terminates the sequence.