---
title: "Understanding Swift by example - Part 1: Structs"
slug: structs-in-swift
---     

At MakeSchool we have decided to switch from Objective-C to Swift for all of our new online content and courses. We get a lot of questions from the community regarding Swift:

* Should I learn Swift or Objective-C?
* Which of the two languages is easier to learn?
* What are the differences between Objective-C and Swift?

We will address all of these questions in a series of articles. Today I want to discuss some unique characteristics of the Swift programming language. Specifically we will be discussing structs. We will add further parts to this series until we have covered most of Swift's language features. As always we like to teach by example so you will be working with interactive playgrounds throughout this tutorial.

Why are we starting this series with structs? It turns out that, unlike in Objective-C, structs are a very powerful tool in Swift and can be used instead of classes in many cases. Throughout this first tutoiral part we will discuss the most important characteristics of structs and when and how they can be used instead of classes.

##How is this different than the official Apple documentation?
Think of this tutorial series as a more accessible version of the [Apple language documentation](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/index.html). For each topic that we discuss we will have a bunch of examples which you can use in playground. We will modify these examples as we work through the tutorial. We discuss underlying principles in a little more detail to make this series more beginner friendly. We also prefer repeating some information multiple times if that makes it easier for you to learn Swift.

##What you need for this tutorial series
* Knowledge in any object oriented programming language
* (Optionally: Objective-C; we will use Objective-C for comparison reasons)
* MacOS with an Xcode Version of 6.1.1 or higher

##What you're going to learn in this part

* Some basic Swift syntax and language concepts
* What is new about structs in Swift? What is the same as in Objetive-C?
* Difference between value and reference types

#The Basics

If you have programmend in C or Objective-C before you might be familiar with structs. However, structs in Swift are far more powerful than in C based languages. In Swift a struct is very similar to a class, you will learn the differences shortly. 

##Defining a basic struct

Let's start by defining a first struct in our playground:

	struct TodoItem {
	  var title: String
	  var content: String
	  var dueDate: NSDate
	  let owner: String
	}

First and foremost a struct is a collection of variables. This struct models a todo entry. The general syntax for declaring variables in Swift is:

* `var` or `let` keyword
* variable name + `:`
* variable type 

We will take about variables and types in detail in a later part of this tutorial series. Then you will learn that defining the type of a variable is not always necessary. For now however we will focus on the details of this struct.

Variables that belong to a struct are called *members*. The struct has three `var` members: `title`, `content` and `dueDate`. The values of these members can be changed throughout the lifetime of an instance of `TodoItem`. The struct also has one `let` member. The `let` keyword marks constants in Swift. This means that the owner string cannot be changed after a `TodoItem` has been created. 

##Initializing a basic struct 

Here's how you create an instance of this `TodoItem` struct:

	var todoItem = TodoItem(title: "Get Milk", content: "really urgent!", dueDate: NSDate(), owner:"User1")

If your struct does not define its own initializer Swift provides a default *memberwise initializer*. That default initializer takes each of the fields of the struct as a parameter. This is also the default behavior for structs in Objective-C.

##Modifying a struct

By default modifying struct instances in Swift works the same way as modifying class instances.
Assume a user would want to change the title of a todo item. We can use this straightforward syntax to modify members:

	todoItem.title = "Get 2 Milk"
	
The playground should visualize that your struct has been updated correctly:

![](modify_title.png) 

Modifying the owner on the other hand will not work because the `owner` member was marked as constant. Try this:

	todoItem.owner = "User25"

And you will see an error:
![](modify_owner.png) 

You can also create immutable instances of a struct. This means that once the instance is created, none of the fields can be modified, not even the fields declared as `var`. This .
Using the `let` keyword you can create an instance of `TodoItem` that cannot ever be modified:

	let unchangeableTodoItem = TodoItem(title: "Get Milk", content: "really urgent!", dueDate: NSDate()	owner:"User1") 
	unchangeableTodoItem.title = "New Title"

![](immutable_enum.png)

#New in Swift

So far we have looked at the basics of structs in Swift, all of which are available in Objetive-C as well. Now we will dive into the more exciting features. In this section we will discuss features that make structs a great replacement of classes in many cases.

##Structs can have custom initializers

Earlier we mentioned the default *memberwise* initializer which is very similar to initializing structs in Objetive-C. In Swift structs can have custom initializers. This is a broad topic because adding a custom initializer has a lot of side effects that we will discuss.

Let's start by adding an initializer. We want the user to be able to create a `TodoItem` only providing an `owner` but not a `title`, `content` or a `dueDate`:

	struct TodoItem {
	  var title: String
	  var content: String
	  var dueDate: NSDate
	  let owner: String
	  
	  init(owner:String) {
	    self.owner = owner
	  }
	}
	
	var todoItem = TodoItem(title: "Get Milk", content: "really urgent!", dueDate: NSDate(), owner:"User1")

Initializers can be created with the `init` keyword. Like regular methods they take a list of parameters, however they don't have a return type. All we need to do in the body of the initializer is map the parameters to member variables. In Swift the `self` keyword is optional. You only are forced to use it in situations as shown above where a parameter name and a member name are the same.

You will realize that the above code does not run in the playground:

![](custom_init.png)

When we implement a custom initializer Swift no longer provides the default *memberwise* initializer. In many cases this OK. If you however want to keep the memberwise initializer and simply provide an additional initializer you can add the initializer to an extension instead of adding it to the struct directly:

	struct TodoItem {
	  var title: String
	  var content: String
	  var dueDate: NSDate
	  let owner: String
	}

	extension TodoItem {
	  init(owner:String) {
	    self.owner = owner
	  }
	}

	var todoItem = TodoItem(title: "Get Milk", content: "really urgent!", dueDate: NSDate(), owner:"User1")

With this solution you will still run into a compiler error:

![](custom_init_error.png)

What's the problem here? [Apple's documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html) states:
> Classes and structures must set all of their stored properties to an appropriate initial value by the time an instance of that class or structure is created. Stored properties cannot be left in an indeterminate state.

With the initializer we are currently providing we are only setting one out of four values. The values for `title`, `content` and `dueDate` are indetermined when the initializer completes. As shown in the quote this is not allowed in Swift. How can we change the code to compile? There are three options:

* assign some value for the other three members inside of the initializer
* set a default value for each of the three members
* mark the three members as *Optional* so that they are allowed to have no value after initialization

The first solution is pretty easy, simply change the initializer to look like this:

	extension TodoItem {
	  init(owner:String) {
	    self.owner = owner
	    self.content = "Default content"
	    self.title = "Default title"
	    self.dueDate = NSDate()
	  }
	}
	
Now we are assigning values to all members and the Swift compiler is satisfied. Finally we can use our new initializer that only takes one parameter:

	var todoItem2 = TodoItem(owner: "User29")


The second solution is assigning default values outside of the initializer, direcltly as part of the variable declaration. With the second solution our new struct and the extension with the initializer look like this:

	struct TodoItem {
	  var title     = "Default title"
	  var content   = "Default content"
	  var dueDate   = NSDate()
	  let owner: String
	}

	extension TodoItem {
	  init(owner:String) {
	    self.owner = owner
	  }
	}
	
This second solution has a big advantage when you add multiple different initializers. You won't have to repeat the default values inside each of these initializers. Another side effect of assiging default values is that we no longer need to declare the type of variables. This Swift feature is called *type inference*. The compiler automatically detects the type of the value that we are assigning to a variable and uses that type for the variable declaration.

The third possible solution for implementing an initializer that does not receive values for all members is using *Optionals*. Optionals are a new feature in Swift. They are essential and we will discuss them in detail in  a separate tutorial. For now it is enough to know that Optionals are a way to represent values that can contain nothing. Here's what our struct could look like when using Optionals:

	struct TodoItem {
	  var title: String?
	  var content: String?
	  var dueDate: NSDate?
	  let owner: String
	}

	extension TodoItem {
	  init(owner:String) {
	    self.owner = owner
	  }
	}

When we mark the type of a variable as optional we tell the compiler that it is OK if this variable contains no value. Since it is OK for Optionals to contain nothing we are not forced to set an initial value in the initializer. As mentioned earlier we will discuss Optionals separately, for now we'll just remember how they can be used when initializing structs.

##Structs can have methods
This is another big difference between C language structs and Swift structs. Formerly structs could only store data, now they can also perform actions. Let's assume we want to present a string in an app that summarizes some information about a todo item. We can add a method to compute that string to our `TodoItem`:

	struct TodoItem {
	  var title     = "Default title"
	  var content   = "Default content"
	  var dueDate   = NSDate()
	  let owner: String
	  
	  func summary() -> String {
	    return "\(title) belongs to \(owner)"
	  }
	}
	
The `func` keyword is used for fuctions and methods in Swift, it is followed by the method name and a parameter list in parantheses. This function takes no parameters since it only operates on the members of the struct. The `->` symbol is placed in front of the return type of Swift methods, this method returns a String.

We can call this method the same way as we would on a class:

	todoItem.summary()
	
You should see the following output:

![](function_output.png)

There's a special rule for methods that modify structs. Assume you want a function that sets the due date of a todo item to today. The straightforward implementation would be adding the following function to your struct:

	func makeDueToday() {
	  dueDate = NSDate()
	}
	
If you do that you will once again see a compiler error:

![](mutating_func_error.png)

Why? By default instance methods of structs cannot modify instance variables. In the last part of this tutorial we will see why this default behavior makes sense for the most ways we use structs in Swift. 
However, this default behavior can be changed with the `mutating` keyword. By adding the keyword `mutating` to a function we indicate that this function wants to change instance variables of the struct:

	mutating func makeDueToday() {
     dueDate = NSDate()
	}
	
Now we can call the function and change the `dueDate`:

	var todoItem = TodoItem(title: "Get Milk", content: "really urgent!", dueDate: NSDate(timeIntervalSinceNow: 60000), owner:"User1")
	todoItem.makeDueToday()
	
You might have guessed it: mutating functions can only be called on structs that are stored in mutable variables (keyword `var`)! Remember, if we store a struct in a variable marked as immutable (keyword `let`) then the struct itself can never be modified. Therefore we also can't call methods that would modify the struct:

	let todoItem = TodoItem(title: "Get Milk", content: "really urgent!", dueDate: NSDate(timeIntervalSinceNow: 60000), owner:"User1")
	todoItem.makeDueToday()
	
You will see this compiler error:

![](constant_mutating_error.png)

##Structs can implement protocols

Since structs in Swift can implement methods they can also confirm to protocols! In practice however this functionality is still pretty limited. Most of the protocols you use when writing iOS apps are currently defined as *Class-Only Protocols* which means that they cannot be implemented by structs. We will discuss protocols in Swift in a later tutorial post and come back to discuss how they can be used with structs.

#Conclusion



