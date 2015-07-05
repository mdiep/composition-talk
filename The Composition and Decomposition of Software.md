
^ Designed for Deckset's black Franziska theme

# The **_Composition_**
# and **_Decomposition_**
# of Software
# ~
### **Matt Diephouse @mdiep**

^ Thesis: We should be using functional composition, and that requires changing
the decomposition of our programs.

---

^ What do _composition_ and _decomposition_ mean?

^ Composition: How we combine our code—the assembly

^ Decomposition: The elements of our programs—the parts

# _Composition_: **Assembly**
# _Decomposition_: **Parts**

---

# **_Composition_**
# + **_Decomposition_**
# = **_Program_**

^ Together, they make up our programs. If you take the parts and assemble them, they make a program.

^ Composition and Decomposition are also interdependent. If you change one, you have to change the other.

---

![](http://brickextratest.files.wordpress.com/2013/05/may2013mmmb.jpg)

# **LEGO**
## **_Bricks_** designed for **_Interlocking Assembly_**
## **_Interlocking Assembly_** depends on **_Bricks_**

^ To thing about this a little more clearly, consider LEGOs. LEGO bricks and their assembly where designed together and depend on each other.  If you changed one, you'd have to change the other.

---

![](http://lego.brickinstructions.com/42000/42025/015.jpg)

# **LEGO** Technic
## _Different **Parts**_
## _Different **Assembly**_
## _Different **Qualities**_

^ And LEGO has a different type of parts and assembly called Technic. But you have to change both the parts _and_ the assembly.

^ Interestingly, the reason to change them and use a different system is because the different methods of assembly have different qualities.

---

# [fit] Composition **_Matters_**

^ In other words, _Composition Matters_. How we assemble affects the parts as well as the finished product. How we compose units of code into a program is important. This is what I want to convince you of today.

^ So let's talk about software.

---

## Types of Composition

```swift
// Primitive Composition
func f(a: A) { ... g(a) ... }

// Higher-Order Composition
func f(a: A, g: A -> B) { ... g(a) ... }

// Functional Composition
func f(b: B) { ... }
f(g(a))
```

^ Thinking in functions because they’re the smallest units of code. There are 3 ways to combine two functions. These are the basic forms of composition. You can write a function that does all 3.

^ Primitive
* Call one function from within the other
* Uses an implementation

^ Higher-order
* Pass one function to the other
* Uses an interface

^ Functional
* Pass the result of one to the other
* Uses a value

^ Objects are _worse_ than functions.
* Potentially multiple functions
* Also includes state
* Additional initialization problem
* Inheritance!

^ But in order to evaluate these, we need to have language to discuss software quality.

---

^ Cohesion and Coupling are two metrics of software quality.

# **_Cohesion_**
## How **_focused_** is this code?

^ Cohesion
* See also: _single responsibility principle_
* Want **high** cohesion

---

# **_Responding_** to user input
# is different than
# **_interpreting_** it

---

# **_Parsing_** a network response
# is different than
# **_presenting_** it

---

# **_Controlling_** a view
# is different than
# **_persisting_** data

---

# **_Deciding_** what to do
# is different than
# **_doing_** it

^ Command/Query Separation

---

# **_Coupling_**
## How **_interconnected_** is this code?

^ Coupling
* See also: _separation of concerns_
* Want **low**/**loose** coupling

---

# **_Side effects_**
# connect code to
# **_external state_**

^ They create a connection to the outside world.

---

# **_Global variables_**
# connect code to
# **_internal state_**

^ They create connections across different parts of your program.

---

# **_Objects_**
# connect code to
# **_object state_**

^ They create connections between anything that has the object.

---

# **_APIs_**
# connect code to
# **_interfaces or implementations_**

^ They create connections to those interfaces and implementations.

---

# **_Coupling_** isn't **_binary_**
## _Specificity brings higher coupling_

^ Hierarchy
  - High: Implementation
  - Large, Specific Interface
  - Low: Small, Generic Interface

---

# **_Coupling_** and **_Cohesion_**
# are _correlated_ but _distinct_

^ As code does more things, it tends to have more connections

^ As code has more connections, it tends to do more things

---

# **_Coupling_** and **_Cohesion_**
# are _contagious_

---

# **_Coupling_** and **_Cohesion_**
## Can `f` be **_tested_** without `g`?
## Can `f` be **_used_** without `g`?
## Can `f` be **_changed_** without `g`?

^ These are measures of Good Software™

---

# **_Coupling_** and **_Cohesion_**
# depend on
# **_Decomposition_**

^ They measure the quality of the parts.

^ Code inherits the coupling and cohesion of its parts.

---

# **_Decomposition_**
# depends on
# **_Composition_**

^ But ultimately the parts depend on how you plan to combine them.

---

# **_Coupling_** and **_Cohesion_**
# depend on
# **_Composition_**

^ So in the end, coupling and cohesion depend on the type of composition that you use.

^ Now we're ready to discuss the types of composition.

---

# Primitive Composition
## `func f(a: A) { ... g(a) ... }`

^ _Primitive_ because:
  1. it’s the most basic form
  2. it’s best used with primitives

---

# Primitive Composition
## `func f(a: A) { ... g(a) ... }`
## How high is the **_coupling_** of `f`?

^
* `f` depends on `g`
* It depends on what `f` and `g` do
* Low if `g` is an _essential_ part of `f`
* Low if `g` has low coupling
* High if `g` has high coupling:
  - If it has side effects
  - If it reads from the outside
  - etc.
* Coupled to `g`'s interface and implementation

---

# Primitive Composition
## `func f(a: A) { g(a) }`
## How high is the **_cohesion_** of `f`?

^ * `f` does what `f` _and_ `g` do
* High if `g` is an _essential_ part of `f`
* High if `g` has high cohesion
* High if `f` is narrowly focused
* Low if `g` has low cohesion

---

```swift
import Contacts

func fetchContactsJSON(block: [NSObject: AnyObject] -> Void) { ... }

extension CNContactStore {
    func contactWithGivenName(givenName: String, familyName: String) -> CNContact? { ... }
}

func importRemoteContacts() {
    fetchContactsJSON() { JSON in
        do {
            try importFromJSON(JSON)
        } catch {
            print("Error! «\(error)»")
        }
    }
}
```

---

```swift
func importFromJSON(JSON: [NSObject: AnyObject]) throws {
    guard let contacts = JSON["contacts"] as? [[String:String]] else {
        return
    }
    let store = CNContactStore()
    let saveRequest = CNSaveRequest()
    for data in contacts {
        guard let givenName = data["givenName"], let familyName = data["familyName"], let email = data["email"] else {
            continue
        }
        if let contact = store.contactWithGivenName(givenName, familyName: familyName) as? CNMutableContact {
            let emails = contact.emailAddresses.map { $0.value as! String }
            if !emails.contains(email) {
                contact.emailAddresses += [ CNLabeledValue(label: CNLabelHome, value: email) ]
                saveRequest.updateContact(contact)
            }
        } else {
            let contact = CNMutableContact()
            contact.givenName = givenName
            contact.familyName = familyName
            contact.emailAddresses = [ CNLabeledValue(label: CNLabelHome, value: email) ]
            saveRequest.addContact(contact, toContainerWithIdentifier: nil)
        }
    }
    try store.executeSaveRequest(saveRequest)
}
```

^
1. `importFromJSON` is tightly coupled
  * Contacts Framework
    - CNContactStore
    - CNSaveRequest
    - CNMutableContact
  * JSON
    - Decoding
    - Structure returned from the server
2. `importFromJSON` has low cohesion
  - Decodes JSON
  - Queries the store
  - Updates people
  - Adds people
  - Saves the store
3. Thank goodness `fetchContactsJSON` wasn't included!
4. Can this be tested?
  - Not really. It depends on device state.
5. Can it be reused?
  - Nope. It's tied to that server, that store, and task of importing.
6. Can it be changed?
  - Not easily. It's a mess!

---

# Higher-Order Composition
## `func f(a: A, g: A -> B) { g(a) }`

---

# Higher-Order Composition
## `func f(a: A, g: A -> B) { g(a) }`
## How high is the **_coupling_** of `f`?

^
* `f` depends on `g`'s _interface_
* It depends on what `f` does and what `g`'s interface does
* High if `g` is a large, specific interface
* Less coupling because it's less specific
* Coupled to `g`'s interface

---

# Higher-Order Composition
## `func f(a: A, g: A -> B) { g(a) }`
## How high is the **_cohesion_** of `f`?

^
* `f` does what `f` does _and_ what `g`'s interface does
* Similar to the Primitive case
* But the same or less cohesion

---

# **_Dependency Injection_**
## _Passing in_ a dependency
## instead of _calling it directly_.

^
  * Used to break dependencies (coupling)
  * Decreases the specificity of the coupling through abstraction
  * But it's usually a large, specific, and leaky abstraction
    - Is it the same interface that the class had?
  * So the coupling is often not substantially less
    - Can I change the behavior of `g`?
  * Does not really alter the cohesion
  * An improvement, but often not a substantial one
    - More testable
    - Not more reusable or changeable
  * Sometimes it's the best we can do

---

# **_Higher-Order Functions_**
## Passing a _function_
## to an _algorithm_.

^
  * Higher-Order composition that uses an actual _function_ (think Functional Composition)
    - Examples: `map`, `filter`, `reduce`, `sort`
  * Behaves like Functional Composition
  * Uses a small, often generic interface
  * BIG improvement
    - Testable
    - Reusable
    - Changeable
  * Talk more about this later

---

```swift
import Contacts

func fetchContactsJSON(block: [NSObject: AnyObject] -> Void) { ... }

protocol StoreType {
    func contactWithGivenName(givenName: String, familyName: String) -> CNContact?
    func executeSaveRequest(saveRequest: CNSaveRequest) throws
}

extension CNContactStore: StoreType {
    func contactWithGivenName(givenName: String, familyName: String) -> CNContact? { ... }
}

func importRemoteContacts() {
    fetchContactsJSON() { JSON in
        do {
            try importFromJSON(JSON, intoStore: CNContactStore())
        } catch {
            print("Error! «\(error)»")
        }
    }
}
```

---

```swift
func importFromJSON<S: StoreType>(JSON: [NSObject: AnyObject], intoStore: S) throws {
    guard let contacts = JSON["contacts"] as? [[String:String]] else {
        return
    }

    let saveRequest = CNSaveRequest()
    for data in contacts {
        guard let givenName = data["givenName"], let familyName = data["familyName"], let email = data["email"] else {
            continue
        }

        if let contact = intoStore.contactWithGivenName(givenName, familyName: familyName) as? CNMutableContact {
            let emails = contact.emailAddresses.map { $0.value as! String }
            if !emails.contains(email) {
                contact.emailAddresses += [ CNLabeledValue(label: CNLabelHome, value: email) ]
                saveRequest.updateContact(contact)
            }
        } else {
            let contact = CNMutableContact()
            contact.givenName = givenName
            contact.familyName = familyName
            contact.emailAddresses = [ CNLabeledValue(label: CNLabelHome, value: email) ]
            saveRequest.addContact(contact, toContainerWithIdentifier: nil)
        }
    }

    try intoStore.executeSaveRequest(saveRequest)
}
```

^
1. A Dependency-Injection approach
  * Rather than call `CNContactStore` directly, we pass it in
  * Abstracting it takes it a step further
  * We could also abstract away `CNSaveRequest` and `CNMutableContact`
  * Abstractions minimize the interface, which lowers coupling
  * But we're still tied to the style of API (unless we write an adapter)
2. `importFromJSON` is still tightly coupled
  * ContactStoreType
  * Contacts Framework
    - CNSaveRequest
    - CNMutableContact
  * JSON
    - Decoding
    - Structure returned from the server
3. `importFromJSON` has low cohesion—it's unchanged
4. Can this be tested?
  * _Yes_. You can write a different ContactStoreType.
  * But then your tests depend on other code.
5. Can it be reused?
  * To an extent, maybe.
  * Possible to write another ContactStoreType.
  * But you'd have to abstract away the rest of the Contacts Framework
  * And probably write an adapter layer
6. Can it be changed?
  * Still no.

---

# Functional Composition
## `func f(b: B) { ... }; f(g(a))`

^ _Functional_ in the mathematical sense
  * Inputs map to only one output: _Referential Transparency_
  * In other words, value(s) to value(s)
  * The loosest of coupling

---

# Functional Composition
## `func f(b: B) { ... }; f(g(a))`
## How high is the **_coupling_** of `f`?

^
* `f` does not depend on `g`
* Low: it does not depend on any state

---

# Functional Composition
## `func f(b: B) { ... }; f(g(a))`
## How high is the **_cohesion_** of `f`?

^
* `f` does not do what `g` does
* High: it's dedicated to a single purpose

^ Something still needs to call `f` and `g`
  * But something needed to call `f` anyway
  * You can easily give a name to `f(g(a))`

---

# Functional Composition
## _Chain_ functions
## `f(g(h(i(a))))`

^ _Composition_ chains transformations
  * Not unlike Unix™ pipelines
  * Easily understandable, reusable, testable components
  * The highest of cohesion
  * The lowest of coupling

---

# Functional Composition
## Create values that represent
## the work to be done.

^ Often capture work to be done as a value
  * Figuring out what to do isn't the same as doing it
  * Example: function that returns a `NSURLRequest`, but doesn’t make the request
    - Complex example because `NSURLRequest` is complex
    - Makes it possible to test the request without stubbing or making the request
    - Makes it possible to reuse the request but make changes to the work to be done

---

```swift
import Contacts

func fetchContactsJSON(block: [NSObject: AnyObject] -> Void) { ... }

extension CNContactStore {
    func contactWithGivenName(givenName: String, familyName: String) -> CNContact? { ... }
}

func importRemoteContacts() {
    fetchContactsJSON() { JSON in
        do {
            let contactInfo = contactsFromJSON(JSON)
            try importContacts(contactInfo, intoStore: CNContactStore())
        } catch {
            print("Error! «\(error)»")
        }
    }
}
```

---

```swift
func contactsFromJSON(JSON: [NSObject: AnyObject]) -> [ContactInfo] {
    guard let contacts = JSON["contacts"] as? [[String:String]] else {
        return []
    }
    var result: [ContactInfo] = []
    for data in contacts {
        if let givenName = data["givenName"], let familyName = data["familyName"], let email = data["email"] {
            result.append(ContactInfo(familyName: familyName, givenName: givenName, email: email))
        }
    }
    return result
}
func importContacts(contacts: [ContactInfo], intoStore store: CNContactStore) throws {
    let saveRequest = CNSaveRequest()
    for contactInfo in contacts {
        if let contact = store.contactWithGivenName(contactInfo.givenName, familyName: contactInfo.familyName) as? CNMutableContact {
            let emails = contact.emailAddresses.map { $0.value as! String }
            if !emails.contains(contactInfo.email) {
                contact.emailAddresses += [ CNLabeledValue(label: CNLabelHome, value: contactInfo.email) ]
                saveRequest.updateContact(contact)
            }
        } else {
            let contact = CNMutableContact()
            contact.givenName = contactInfo.givenName
            contact.familyName = contactInfo.familyName
            contact.emailAddresses = [ CNLabeledValue(label: CNLabelHome, value: contactInfo.email) ]
            saveRequest.addContact(contact, toContainerWithIdentifier: nil)
        }
    }
    try store.executeSaveRequest(saveRequest)
}
```

^
1. Here, we split the work into two
  * Transformation
  * Application
2. `contactsFromJSON` and `importContacts(intoStore:)` have lower coupling
  * The remaining coupling is mostly essential
  * You could apply the higher-order techniques to further reduce coupling
3. Cohesion is much improved! We've split the work that we can
4. Can this be tested?
  * _Yes_. You can test the parts individually.
  * You'll still want to use DI or mocking to test the actual importing.
5. Can it be reused?
  * Yes!
    - Can plug in a different service
    - Can plug in a different store
    - Can do something entirely different with the `ContactInfo`
6. Can it be changed?
  * Yes!
    - Can easily filter the contacts that are saved
    - Can easily transform their info
    - Can fix JSON decoding without touching importing
    - Can fix importing without touching JSON decoding

---

# **_Composition_**
# + **_Decomposition_**
# = **_Program_**

^ Now that you're hopefully convinced that functional composition has some nice benefits, we need to talk about decomposition. Because, as we discussed before, composition and decomposition are interdependent.

^ We’re used to writing functions that call other functions
* That means rethinking our architecture
* That’s not a bad thing, because we’re not great at writing software

^ If you keep writing the same classes, don't be surprised when you end up with the same coupled and cohesion.

---

# _Minimize_ **Coupling**
## and
# _Maximize_ **Cohesion**
## at every level

^ They're contagious. So you have to push the coupling to the edges.

---

# Use **Functional** composition
# for the **_innermost_** code

^ This is a challenge in the stateful world of apps
* Views and VCs are stateful objects, so you can’t operate on them with Functional
* But you can still feed them data with Functional
* And you can still implement them with Functional

---

# Use **Higher-Order** composition
# for the **_middle_** code

---

# Use **Primitive** composition
# for the **_outermost_** code

^ Integration

---

# Use **value types**
# to **_capture relevant data_**

^ You’ll end up with more value types
* We tend to resist creating new types
* But more classes/structs/enums aren’t a bad thing

---

# Use **value types**
# to separate
# **_decisions_** from **_doing_**

---

# **_Questions_**
# and
# **_Answers_**

---

# Further Reading/Watching
* “Why Functional Programming Matters”
* “Boundaries” by Gary Bernhardt
* “The Clean Architecture in Python” by Brandon Rhodes
* “Making Friends with Value Types” by Andy Matuschak

---

# **_Thank you!_**
# ~
### **Matt Diephouse @mdiep**
