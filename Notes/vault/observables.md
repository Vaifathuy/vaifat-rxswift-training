---
id: dI8ZZxMwdC6iOpPdEfaIB
title: Observables
desc: ''
updated: 1631698970481
created: 1630999552309
---

## **What is an observable?**
> An Observable is a sequence emitting`events`asynchronously over a period of time. 

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
> Create an observable sequence containing `just a single element`.

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
>Create an observable of `individual element from an array` of typed elements. It only takes `array`.

```swift
let observable4 = Observable.from([one, two, three])
```

### With _`empty`_ operator:
> What use is an `empty` observable? They’re handy when you want to return an observable that immediately terminates or intentionally has zero values.

```swift
let observable = Observable<Void>.empty()
```

### With _`never`_ operator:
> As opposed to the empty operator, the `never` operator creates an observable that doesn't emit anything and never terminates. It can be used to represent an infinite duration.

```swift
let observable = Observable<Void>.never()
```

### With _`range`_ operator:
> Creating an observable with range operator takes a _start_ integer value and a _count_ of sequential integers to generate.

```swift
let observable = Observable<Int>.range(start: 1, count: 10)
```
---

### With `create` operator:
> Use `create` operator to create an observable and at the same time specify all the events the observable will emit to subscribers.

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
> To manually terminate an observable, we use _**`dispose`**_ operator.

```swift
let observable = Observable.of("A", "B", "C")
let subscription = observable.subscribe { event in
    print(event)
}

subscription.dispose()
```

### With _`DisposeBag`_ type:
> A dispose bag holds disposables. Simply add each disposable to a dispose bag so that it can automatically dispose each of them resided within the bag.

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
> Traits are observables with a narrower set of behaviors than regular observables. Using traits can help make your code more intuitive. 

>There are `three` kinds of traits in RxSwift: `Single`, `Maybe`, `Completable`.

### **`Single`**:
> Single will emit either a _`success(value)`_ or _`error(error)`_ event. 

>_`success(value)`_ is actually a combination of the next and completed events. 

> <> It is useful for one-time processes that will either succeed and yield a value or fail such as when downloading data or loading it from disk.

### **`Completable`**: 
> A Completable will only emit a _`completed`_ or _`error(error)`_ event. It will not emit any values.

> <> It is useful when you only care that an operation completed successfully or failed, such as a file write.

### **`Maybe`**:
> A Maybe is a mashup of a Single and Completable. It can either emit a _`success(value)`_, _`completed`_ or _`error(error)`_. 

> <> It is useful with an operation that could either succeed or fail, and optionally return a value on success.
