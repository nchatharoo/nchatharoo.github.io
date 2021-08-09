---

layout: post 
title: "Grand Central Dispatch"

---

## Part 2 : DispatchGroup, DispatchSemaphore



So we've seen how to use GCD with one task, but how to deal with multiple tasks ? This is where DispatchGroup is useful.

### DispatchGroup

We start by initializing a `DispatchGroup`, then provide it as an argument to the `async` method of our dispatch queue.

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
	print("Tasks completed !")
}
```

Note that `notify` take a dispatch queue as a parameter, that mean the closure will be executed in the specified one.

Let's see a more concrete example, for each task, we `enter` the group, and `leave` it for each completed task.

```swift
import UIKit
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let dispatchGroup = DispatchGroup() //Create a group for the tasks.
        let session: URLSession = URLSession.shared
      
        dispatchGroup.enter() //Enter the group
        let firstTask = session.dataTask(with: URLRequest(url: URL(string:
"a url")!)) { (data, response, error) in
            //Process Response..
            dispatchGroup.leave() //Leave the group 
        }
      
        dispatchGroup.enter()  //Enter the group
        let secondTask = session.dataTask(with: URLRequest(url: URL(string: "another url")!))
{ (data, response, error) in
            //Process Response..
            dispatchGroup.leave()  //Leave the group
        }
      
        //When all of the tasks listed above have been done, we get a notification on the Main Thread.
        dispatchGroup.notify(queue: DispatchQueue.main) {
        	print("Every task is complete")
        }
      
        //Resume the tasks.
        firstTask.resume()
        secondTask.resume()
    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    	} 
    }
```

See how we use the `enter` and `leave`, if we forgot to leave after entering, the app will hang forever !



### DispatchSemaphore



