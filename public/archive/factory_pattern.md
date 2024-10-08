---
title: "Factory Pattern"
category: 'Design Pattern'
pubDate: 'Feb 13 2024 16:23'
---


- Factory Pattern provides an interface to create an object and delegates object construction from a super class to a special class known as a _factory class_.
- Factory Pattern provides encapsulation through composition.
- Factory Pattern makes the system scalable.

//

The Factory Pattern is employed to abstract and separate object construction within a superclass or base class. This separation is achieved by delegating the responsibility of object creation to a distinct class, commonly referred to as the _Factory class_. The Factory class is designed to create and return objects based on specific conditions or parameters provided during the instantiation process.

## Problem

Consider the scenario where we are developing a mini computer that initializes the _Windows OS_. Initially, since there is only one OS specified, there's no requirement for conditional logics. Consequently, we directly initiate the Windows boot-up process as follows:

```js
class MiniComputer {
	boot() {
		console.log("Booting up Windows...")
	}
}

const mini_computer = new MiniComputer()
mini_computer.boot()
```

However, the following day, we are instructed to incorporate support for the _Mac OS_. In response to this, we modify our codebase to accommodate these changes.

```js
class MiniComputer {
	#os;
	constructor(os) {
		this.#os = os.toLowerCase()
	}
	
	boot() {
		if(this.#os === 'windows') {
			console.log("Booting up Windows OS...")
		} else if (this.#os === 'mac') {
			console.log("Booting up Mac OS...")
		} else {
			console.error("Unsupported OS...")
		}
	}
}

const mini_computer = new MiniComputer("windows")
mini_computer.boot()
```

We already spotted an issue here. The client code has changed. Previously, they simply called the constructor `new MiniComputer()`, but now, they have to specify the OS with `new MiniComputer("Windows")`. As soon as people update the version of our system, they are likely to encounter an error.

And we only had one function here, `boot()`. Imagine if we had more functions. That means as we add support for different OS, we'll have to modify every one of those methods in the class. You can see how things can get messy pretty quickly.

## Solution

We can apply factory patterns here by creating an interface that includes a shared method, such as `boot()`. Subsequently, each concrete class (e.g., `WindowsOS`, `MacOS`, etc.) will inherit this interface. Now, our superclass is freed from the responsibility of creating and returning an object. This approach ensures that we don't have to modify anything whenever there's a need to add support for new OS logics.

## Pseudocode

First, let's establish the interface for our system. Every mini computer (PC) within our system will share two operations: `boot()` and `openBrowser()`.

```js
// interface
class PC {
	boot() {}
	openBrowser() {}
}
```

Now, we can proceed to implement concrete classes that inherit the `PC` interface, ensuring that each class implements all the shared methods.

```js
// concrete classes
class WindowsOS extends PC {
	boot() {
		console.log("Booting Windows...")
	}
	openBrowser() {
		console.log("Edge is ready!")
	}
}

class MacOS extends PC {
	boot() {
		console.log("Booting Mac...")
	}
	openBrowser() {
		console.log("Safari is ready!")
	}
}
```

Now, introducing the factory class. This class is designed to return one of the supported machines based on a given condition. In this particular case, the condition is the client's operating system (OS).

```js
// factory class
class PCFactory {
	static createPC(userAgent) {
		if(userAgent.indexOf("Win") !== -1) {
			return new WindowsOS()
		} else if(userAgent.indexOf("Mac") !== -1) {
			return new MacOS()
		} else {
			return null	
		}
	}
}
```

And finally, revisiting our initial `MiniComputer` class, you'll notice that it remains unchanged from our initial code, except for the addition of more functionality, `openBrowser()`.

```js
class MiniComputer {
	#pc;
	constructor() {
		const userAgent = navigator.userAgent
		this.#pc = PCFactory.createPC(userAgent)
	}

	boot() {
		this.#pc.boot()
	}

	openBrowser() {
		this.#pc.openBrowser()
	}
}

const mini_computer = new MiniComputer()
mini_computer.boot()
mini_computer.openBrowser()
```

Let's consider a scenario where we don't have the `MiniComputer` class, and we're not creating a PC based on the client's OS. In this case, we can directly call the `PCFactory` like this.

```js
const win = PCFactory.createPC('Windows')
const mac = PCFactory.createPC('Mac')

win.boot() // 'Booting WindowsOS...'
win.openBrowser() // 'Opening Edge'

mac.boot() // 'Booting MacOS...'
mac.openBrowser() // 'Opening Safari'
```


## References
- Refactoring.Guru. (2024, January 1). Factory method. https://refactoring.guru/design-patterns/factory-method