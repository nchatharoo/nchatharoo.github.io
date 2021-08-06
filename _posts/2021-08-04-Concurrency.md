## Part 1 : Grand Central Dispatch (GCD)

When starting developing, did you think of performance and/or responsiveness for your app ?
It's not easy, because there is a lot of way to improve your app. One of it is *Concurrency*.

Concurrency on iOS is a large topic, today let's focus on Grand Central Dispatch (GCD).

GCD is Apple’s implementation of C’s **libdispatch** library, it allow us to write multi-threaded code without manually creating the threads themselves. We do not need to worry about managing them, because
GCD's tasks (either a method or a closure) are placed into GCD-managed **first-in, first-out** (FIFO) queues fully handled by the system.

> We place tasks in queues, GCD will place these queues in a pool, and will execute them in a FIFO based procedure BUT the tasks we submit **does not finish in the order** we submit them. The FIFO procedure applies when the task *starts*, thats because the completion of the tasks depends on several factors.

Tasks can run synchronously or asynchronously :

- Synchronous tasks : the app will wait and block the current run loop until execution finishes.
- Asynchronous tasks : will start and return execution the app immediately.

When we need to perform long task (like networking call or computationally expensive work), we :
-create a queue
-attach a task to it to run asynchronously on a background thread 
-delegate the code back to the main thread, when the task is completed.

For example :

```swift
  let queue = DispatchQueue(label: "com.nchatharoo.work") // Create it in init()
	// Later in a function
  queue.async { _ in
      DispatchQueue.main.async { // UI related work
          // Update UI here
      }
  }
```



### Serial vs Concurrent queue

<u>Serial</u> : Serial queue only have a single thread associated with them and will complete each task in the order they are submitted to the queue. When creating a queue, it is serial by default.

<u>Concurrent</u> : Each task runs as soon as possible without waiting for other tasks in the queue. The advantage is that the concurrent queue time is shorter than the serial queue because all the tasks in the queue are executed simultaneously in separate threads.

Use a concurrent queue if the order of execution is not important.

> Even if we tell iOS that we want to use a concurrent queue, keep in mind that there is no *guarantee* that more than one task will run at a time. If the app is struggling for resources, it may only be able to perform one task at a time.



There is 3 kind of queues available for in our app :

- The Main dispatch queue (serial)
- Global (Background) queues (concurrent)
- Private queues (serial or concurrent)



##### **The Main dispatch queue**

It's created automatically when the app start, it's a serial queue, all the UI updates take place in here and the code involving UI changes are placed.
As it is heavily used, Apple has provided us the *DispatchQueue.main* to access it.

```Swift
DispatchQueue.main.async { // Asynchronous Main Queue call
  // Will not block the UI
}
```

 The code inside the `{ }` won't block the UI, since its executed in a Background thread.

Be aware calling *DispatchQueue.main.sync* can lead to **deadlock** (Main Queue waiting for itself), that's because the calling queue will wait until the work we dispatch in the block is finished.

So when to use `sync` ? If you are submitting a tiny task (for example, updating a value), consider doing it synchronously. For example : 

```swift
DispatchQueue.global().async { // Asynchronous Background thread call
// Here we do something
	DispatchQueue.main.sync { // Synchronous Main Queue call
    // Update UI
  }
// This part will execute after 'Update UI' has finished
}
```

`DispatchQueue.global().async` will perform using a Background Thread  and when task inside the block finish, `DispatchQueue.main.sync` will bring the work from Background Queue to the Main Queue, which will update the UI.



##### The Global (Background) queues

Global queues are always concurrent and first-in, first-out. Apple have provided 6 different global queues, they differ by the *Quality of service (QoS)* the queue should have.

We create a global queue with QoS like this :

```swift
let queue = DispatchQueue.global(qos: .userInteractive)
```

We can also give a *default priority* like this : 

```swift
let queue = DispatchQueue.global() // default QoS falls somewhere between *user initiated* and *utility*
```

To add task to a global queue we can write this :

```swift
DispatchQueue.global(qos: .utility).async { [weak self] in
  guard let self = self else { return }
  // Perform work here
  DispatchQueue.main.async {   // Switch back to the main queue to update your UI
    self.label.text = "New text here"
  	}
  }
```

Task refer to any block of code using the `sync` or `async` functions.

##### Private queues

Finally, we can create our own private( or custom) queue. By default, private queues are *serial*.
Here is how we can create it : 

```swift
let concurrent = DispatchQueue(label: "com.nchatharoo.concurrent-queue", qos: .userInitiated, attributes: .concurrent)
concurrent.sync {
    print("Private concurrent queue")
}
```

As we can see, we can also precise what the QoS is but be aware, it is not guaranteed that it's gonna stay that way !
In fact the OS will eventually change the QoS based on the type of task submitted to the queue and perform change as necessary.