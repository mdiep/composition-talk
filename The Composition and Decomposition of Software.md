# The Composition and Decomposition of Software
Thesis: We should be using functional composition, and that requires changing the decomposition of our programs.

# Overview
1. What are _Composition_ and _Decomposition_?
2. Why do they matter?
3. Decomposition
4. Types of Composition
5. Composition affects Decomposition
6. Changing Decomposition
7. Advanced Composition

# What do _composition_ and _decomposition_ mean?
1. _Composition_: Assembly
  * Not talking about “functional composition” (yet)
2. _Decomposition_: Parts
3. Composition + Decomposition = Program
  * (Assembly + Parts = Thing)
4. Composition and Decomposition are interdependent
  * Consider LEGO™
    - Brick structure affects what and how you can build
    - Technic exists to let you build other things
    - See also: Erector Set, Knex, etc.

# Why do they matter?
1. _Cohesion_: How related are the components of this code?
  * See also: _single responsibility principle_
  * Want **high** cohesion
  * Responding to touches isn't really related to how they're drawn
  * Figuring out _what to do_ isn't the same as _doing it_
    - See also: _Command/Query Separation_
2. _Coupling_: How interconnected is this code?
  * See also: _separation of concerns_
  * Want **low**/**loose** coupling
  * Connection isn't binary; Specificity brings higher coupling
    - High: Implementation
    - Large, Specific Interface
    - Low: Small, Generic Interface
  * Stateful interfaces have high coupling. They're highly connected.
    - Side effects: observable by the world
    - Reading outside data: observing the world
    - Global variables: observable by anything in the program
    - Object state: observable by anything that has that object
3. Coupling and Cohesion are related, but distinct:
  * As code does more things, it tends to have more connections
  * As code has more connections, it tends to do more things
4. Coupling and Cohesion affect:
  * Testability: Can I test `f` without `g`?
  * Reusability: Can I use `f` without `g`?
  * Changeability: Can I change `g` without changing `f`?
  * These are measures of Good Software™

# Decomposition
1. Coupling and Cohesion are a question of parts
2. Coupling of `f`: A set of the coupling of all its components
3. Cohesion of `f`: A set of the connections of all its components
4. Push your connections as far "out" as you can to minimize coupling
5. Composition affects Decomposition, so it also affects Coupling and Cohesion

# Types of Composition
Thinking in functions because they’re the smallest units of code. There are 3 ways to combine two functions:

1. _Primitive Composition_: `f(a) { ... g(a) ... }`
  * Call one function from within the other
  * Uses an implementation
2. _Higher-Order Composition_: `f(a, g: A -> B) { ... g(a) ... }`
  * Pass one function to the other
  * Uses an interface
3. _Functional Composition_: `f(g(a))`
  * Pass the result of one to the other
  * Uses a value

These are the basic forms of composition. You can write a function that does all 3.

Objects are equivalent to or worse than functions
  * Potentially multiple functions
  * Also includes state
  * Additional initialization problem
  * Inheritance! :scream:

## Primitive Composition
1. _Primitive_ because:
  1. it’s the most basic form
  2. it’s best used with primitives
2. Coupling and Cohesion
  1. Coupling: `f` depends on `g`
  2. Cohesion: `f` does what `f` _and_ `g` do
  3. How high is the coupling of `f`?
    * It depends on what `f` and `g` do
    * Low if `g` is an _essential_ part of `f`
    * Low if `g` has low coupling
    * High if `g` has high coupling:
      - If it has side effects
      - If it reads from the outside
      - etc.
    * Coupled to `g`'s interface and implementation
  4. How high is the cohesion of `f`?
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

## `Higher-Order` Composition
1. Coupling and Cohesion
  1. Coupling: `f` depends on `g`'s _interface_
  2. Cohesion: `f` does what `f` does _and_ what `g`'s interface does
  3. How high is the coupling of `f`?
    * It depends on what `f` does and what `g`'s interface does
    * High if `g` is a large, specific interface
    * Less coupling because it's less specific
    * Coupled to `g`'s interface
  4. How high is the cohesion of `f`?
    * Similar to the Primitive case
    * But the same or less cohesion
2. Often used for _Dependency Injection_
  * DI is used to break dependencies (coupling)
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
3. _Higher-Order Functions_
  * Higher-Order composition that uses an actual _function_ (think Functional Composition)
    - Examples: `map`, `filter`, `reduce`
  * Behaves like Functional Composition
  * Uses a small, often generic interface
  * BIG improvement
    - Testable
    - Reusable
    - Changeable
  * Talk more about this later
4. _Callbacks_, _Delegates_, _Data Sources_, and _Notifications_ are all forms of this
  * Lower the specificity of connections
  * Changes connections from 1 to 0, 1, or n

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

## Functional Composition
1. _Functional_ in the mathematical sense
  * Inputs map to only one output: _Referential Transparency_
  * In other words, value(s) to value(s)
  * The loosest of coupling
2. Coupling and Composition
  1. Coupling: `f` does not depend on `g`
  2. Cohesion: `f` does not do what `g` does
  3. How high is the coupling of `f`?
    * Low: it does not depend on any state
  4. How high is the cohesion of `f`?
    * High: it's dedicated to a single purpose
  5. Something still needs to call `f` and `g`
    * But something needed to call `f` anyway
    * You can easily give a name to `f(g(a))`
3. _Composition_ chains transformations
  * Not unlike Unix™ pipelines
  * Easily understandable, reusable, testable components
  * The highest of cohesion
  * The lowest of coupling
4. functional > higher order > primitive
  * Testability: easier to test
  * Reusability: easier to reuse
  * Changeability: easier to change
5. Often capture work to be done as a value
  * Figuring out what to do isn't the same as doing it
  * Example: function that returns a `NSURLRequest`, but doesn’t make the request
    - Complex example because `NSURLRequest` is complex
    - Makes it possible to test the request without stubbing or making the request
    - Makes it possible to reuse the request but make changes to the work to be done

---

```swift
import Contacts

func fetchContactsJSON(block: [NSObject: AnyObject] -> Void) { ... }

struct ContactInfo {
    var familyName: String
    var givenName: String
    var email: String
}

extension CNContactStore {
    func contactWithGivenName(givenName: String, familyName: String) -> CNContact? { ... }
}

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

# Composition affects decomposition
1. We’re used to writing functions that call other functions
  * That means rethinking our architecture
  * That’s not a bad thing, because we’re not great at writing software
2. We want to minimize the coupling and maximize the cohesion at every level
3. If you try to mostly write functional functions, then:
  1. Innermost layers will be Functional
  2. Middle layers will be Higher-Order
  3. Outermost layers will by Primitive
  4. You’ll end up with more value types
    - We tend to resist creating new types
    - But more classes/structs/enums aren’t a bad thing
4. This is a challenge in the stateful world of apps
  * Views and VCs are stateful objects, so you can’t operate on them with Functional
  * But you can still feed them data with Functional
  * And you can still implement them with Functional

# Higher Order Functions
* Power-ups for values and functions
* Apply functions in different patterns (map, grep/filter, reduce/fold, etc.)
* Pipelines for values
* Monads et al?

# Q&A

# Further Reading/Watching
* “Why Functional Programming Matters”
* “Boundaries” by Gary Bernhardt
* “The Clean Architecture in Python” by Brandon Rhodes
* “Making Friends with Value Types” by Andy Matuschak
