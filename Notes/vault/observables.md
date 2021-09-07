---
id: dI8ZZxMwdC6iOpPdEfaIB
title: Observables
desc: ''
updated: 1631008042781
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
### With _`just`_ method:

```swift
let one = 1
let observable = Observable<Int>.just(one)
```

