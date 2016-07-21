# ScriptKit - Convenient Scripts in Swift (WIP)

This framework stems from not wanting to get into complicated stuff when you just want to write a short script in Swift. By script I am not referring to a single interpreted file, but rather a small application that does useful things, mostly without UI.

###Script protocol (mostly done)

This is the part of this framework that's mostly done. It handles things like restarts of the script, waiting for a certain amount of time before terminating and the Apps run loop. To use it for your script:

 1. Create a new OSX application project
 2. Delete every file except `Info.plist`
 3. In the `Info.plist`, set the `Application is background only` key to `YES`
 4. Add a new Swift file called `main.swift` (If Xcode asks, you don't need a bridging header)
 5. Implement a class conforming to `Script`. Example: 
 
```swift
import ScriptKit

final class MyScript : Script {
    func setUp(manager: Manager<MyScript>) {
        manager.terminationDelay = 5
        
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 3_000_000_000), manager.metaQueue) {
            manager.invokeMain(context: "This is three seconds later")
        }
    }
    
    func main(manager: Manager<MyScript>, index: Int32, group: dispatch_group_t, context: String?) {
        if index == 0 {
            NSLog("First start of the script")
        }
        
        if let message = context {
            NSLog(message)
        }
        
        NSLog("Starting work on \(index). invocation")
        NSThread.sleepForTimeInterval(5)
        NSLog("Finished work on \(index). invocation")
    }
    
    func tearDown(manager: Manager<MyScript>) {
        NSLog("Finished")
    }
}

MyScript.run()
```

If you run this application and double-click again after 10 seconds, this is the output of Console:

``` 
02:29:12 Example: First start of the script
02:29:12 Example: Starting work on 0. invocation
02:29:16 Example: This is three seconds later
02:29:16 Example: Starting work on 1. invocation
02:29:17 Example: Finished work on 0. invocation
02:29:21 Example: Finished work on 1. invocation
02:29:22 Example: Starting work on 2. invocation
02:29:27 Example: Finished work on 2. invocation
02:29:33 Example: Finished
```

The documentation describes pretty well what everything does.

###Hotkey class (about done)

The `Hotkey` class is a very simple way of registering a new global Hotkey and execute some code when it gets triggered. The documentation is inexistent atm. It's useable like this:

```swift
import Carbon // Needed for kVK... definitions, maybe this'll be an enum later

HotkeyManager.register([.Command, .Shift] + kVK_ANSI_0)) { hotkey in
    print("⌘⇧-0 was pressed!")
}
```

This has effect as long as the application is running (unless you use the `unregister` method).

###Console (not done at all)

Console is a class representing a running console. The standard initializer uses `bash` (should probably be changed to `$SHELL`), but in theory it's possible to use any shell/command that supports interaction (`fish` doesn't want to work for some reason though). It's for example possible to use the Swift REPL (Swiftception):

```swift
let swiftREPL = try! Console(shell: "swift")
swiftREPL.input("let x = 10; print(x * 2)")
```

###File utilities (so not done it's not even worth mentioning)

##Ideas and stuff to do

- Xcode template for a script that starts at login, supporting the `Script` protocol
- Convenient `NSStatusItem` support (the little icon it the menu bar)
- Add tests
