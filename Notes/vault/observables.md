---
id: dI8ZZxMwdC6iOpPdEfaIB
title: Observables
desc: ''
updated: 1631784148634
created: 1630999552309
---
# **`</> Observables`**

## **What is an observable?**
An Observable is a sequence emitting`events`asynchronously over a period of time. 

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
Single will emit either a _`success(value)`_ or _`error(error)`_ event. _`success(value)`_ is actually a combination of the next and completed events. 

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