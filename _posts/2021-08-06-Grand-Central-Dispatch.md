---

layout: post 
title: "Grand Central Dispatch"

---

## Part 2 : DispatchGroup, DispatchSemaphore



So we've seen how to use GCD with one task, but how to deal with multiple tasks ? Each one of them tasks running on differents threads ? This is where DispatchGroup is useful.

### DispatchGroup

We start by initializing a `DispatchGroup`, then provide it as an argument to the `async` method of our dispatch queue. For each task, we `enter` the group, and `leave` it for each completed task.

For example : 

```swift
let dispatchGroup = DispatchGroup()
someQueue.async(group: group) { /* some work */ }
someQueue.async(group: group) { /* some other work */ }
someOtherQueue.async(group: group) { /* another work */ }
```

We can see that `dispatchGroup` is not attached to a single dispatch. That mean we can submit multiple tasks to multiples queues.

When all tasks are done, `DispatchGroup` will notify us.

```swift
dispatchGroup.notify(queue: DispatchQueue.main) { [weak self ] in 
	print("Tasks commpleted !")
}
```

Note that `notify` take a dispatch queue as a parameter, that mean the closure will be executed in the provided one.
