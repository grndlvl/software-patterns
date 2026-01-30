# Iterator

## Intent

Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

## Also Known As

- Cursor
- Enumerator

## Motivation

Consider a social network application that needs to traverse a user's connections in different ways: friends, friends of friends, colleagues, or followers. The underlying data might be stored in various structures—an adjacency list for close friends, a graph database for extended networks, or a remote API for followers. Client code that wants to iterate over these connections shouldn't need to know or care about these implementation details.

Without the Iterator pattern, each collection type would expose its internal structure. Code iterating over an array differs from code iterating over a linked list, tree, or graph. This couples clients to specific implementations and makes it impossible to switch data structures without rewriting all traversal code.

The Iterator pattern solves this by extracting traversal behavior into separate iterator objects. Each iterator encapsulates the traversal logic for a particular collection type and presents a uniform interface: `hasNext()` to check for more elements and `next()` to retrieve the next element. The client code works with this interface, blissfully unaware of whether it's traversing an array, tree, or remote API.

This separation provides multiple benefits. You can add new collections or traversal algorithms without modifying existing code. The same collection can support multiple simultaneous traversals. And because iterators encapsulate their position, you can pause and resume traversal as needed.

## Applicability

Use the Iterator pattern when:

- You want to access an aggregate object's contents without exposing its internal representation.
- You need to support multiple simultaneous traversals of aggregate objects.
- You want to provide a uniform interface for traversing different aggregate structures (polymorphic iteration).
- You need different traversal algorithms over the same collection (forward, backward, filtered, depth-first, breadth-first).
- You want to decouple algorithms from collections—algorithms work with iterators, not specific collection types.
- You need lazy evaluation—elements are computed or fetched only when requested.
- You want to traverse remote or infinite collections where loading everything into memory is impractical.

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌────────┐         ┌───────────────────────────────────────────┐        │
│    │ Client │         │            <<interface>>                  │        │
│    └───┬────┘         │              Iterator                     │        │
│        │              ├───────────────────────────────────────────┤        │
│        │              │ + hasNext(): boolean                      │        │
│        │              │ + next(): Element                         │        │
│        │              │ + reset(): void                           │        │
│        │              └───────────────────────────────────────────┘        │
│        │                               △                                    │
│        │                               │                                    │
│        │              ┌────────────────┴────────────────────┐              │
│        │              │                                     │              │
│        │     ┌────────┴─────────────┐          ┌───────────┴───────────┐  │
│        │     │   ConcreteIteratorA  │          │   ConcreteIteratorB   │  │
│        │     ├──────────────────────┤          ├───────────────────────┤  │
│        │     │ - collection: ref    │          │ - collection: ref     │  │
│        │     │ - position: int      │          │ - currentNode: Node   │  │
│        │     ├──────────────────────┤          ├───────────────────────┤  │
│        │     │ + hasNext()          │          │ + hasNext()           │  │
│        │     │ + next()             │          │ + next()              │  │
│        │     └──────────────────────┘          └───────────────────────┘  │
│        │              │                                     │              │
│        │              │ traverses                           │ traverses   │
│        │              ▼                                     ▼              │
│        │     ┌──────────────────────┐          ┌───────────────────────┐  │
│        │     │  ConcreteAggregateA  │          │  ConcreteAggregateB   │  │
│        │     └──────────────────────┘          └───────────────────────┘  │
│        │              △                                     △              │
│        │              │                                     │              │
│        │              └──────────────┬──────────────────────┘              │
│        │                             │                                      │
│        │              ┌──────────────┴───────────────────────┐             │
│        │              │            <<interface>>             │             │
│        └─────────────>│             Iterable                 │             │
│                       ├──────────────────────────────────────┤             │
│                       │ + createIterator(): Iterator         │             │
│                       └──────────────────────────────────────┘             │
│                                                                             │
│   Iteration Sequence:                                                       │
│                                                                             │
│   ┌────────┐    ┌──────────┐    ┌──────────┐                              │
│   │ Client │    │ Iterable │    │ Iterator │                              │
│   └───┬────┘    └────┬─────┘    └────┬─────┘                              │
│       │              │               │                                      │
│       │ createIterator()             │                                      │
│       │─────────────>│               │                                      │
│       │              │ new Iterator  │                                      │
│       │              │──────────────>│                                      │
│       │<─────────────│───────────────│                                      │
│       │                              │                                      │
│       │ while hasNext()              │                                      │
│       │─────────────────────────────>│                                      │
│       │<─────────────────────────────│ true/false                          │
│       │                              │                                      │
│       │ next()                       │                                      │
│       │─────────────────────────────>│                                      │
│       │<─────────────────────────────│ element                             │
│       │                              │                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Iterator**: Defines an interface for accessing and traversing elements. The minimal interface includes `hasNext()` and `next()`. May also include `reset()`, `remove()`, or `peek()`.

- **ConcreteIterator**: Implements the Iterator interface. Keeps track of the current position in the traversal. Knows how to compute the next element based on the aggregate's structure.

- **Iterable (Aggregate)**: Defines an interface for creating an Iterator object. This is often called `Iterable`, `IterableCollection`, or simply has a `createIterator()` or `iterator()` method.

- **ConcreteAggregate**: Implements the Iterable interface. Returns an instance of the appropriate ConcreteIterator. May support multiple iterator types (forward, reverse, filtered).

- **Client**: Uses the Iterator interface to traverse elements. Doesn't need to know the concrete classes of either the iterator or the aggregate.

## Collaborations

1. A ConcreteAggregate creates a ConcreteIterator and provides a reference to itself, so the iterator can access its elements.

2. The client requests an iterator from the aggregate via `createIterator()`.

3. The client uses `hasNext()` to check if more elements exist, and `next()` to retrieve the next element.

4. The ConcreteIterator keeps track of the current element and can compute the next element when requested.

5. Multiple iterators can traverse the same aggregate simultaneously, each maintaining its own position.

## Consequences

### Benefits

1. **Supports variations in traversal**: Complex aggregates can be traversed in many ways. Iterators make it easy to change the traversal algorithm—just replace the iterator instance.

2. **Iterators simplify the aggregate interface**: The aggregate doesn't need traversal methods; the iterator handles everything. This keeps the aggregate focused on its primary responsibility: managing elements.

3. **Multiple simultaneous traversals**: Each iterator maintains its own traversal state. You can have multiple iterators active on the same aggregate without interference.

4. **Decouples algorithms from collections**: Algorithms written against the Iterator interface work with any collection that provides an iterator.

5. **Lazy evaluation**: Elements can be computed or fetched on demand. This is essential for infinite sequences, large datasets, or remote data.

6. **Single Responsibility Principle**: Traversal logic is extracted from the collection into iterator classes.

### Liabilities

1. **May be overkill for simple collections**: If you're only iterating over simple arrays or lists and don't need multiple traversals, built-in language features may suffice.

2. **Iterator invalidation**: If the aggregate is modified during iteration, the iterator may become invalid. This is a common source of bugs.

3. **Overhead**: For very small collections traversed frequently, the object creation overhead of iterators may be noticeable.

4. **Limited operations**: A basic iterator only moves forward. Operations like accessing by index or moving backward require extended interfaces.

## Implementation

### Implementation Considerations

1. **Who controls the iteration?**:
   - External iterator: Client controls iteration by calling `next()` explicitly.
   - Internal iterator: Client passes an operation; the iterator applies it to each element.

2. **Who defines the traversal algorithm?**:
   - In the iterator: Most common. The iterator knows how to traverse.
   - In the aggregate: The aggregate exposes a cursor API; the iterator wraps it.

3. **How robust is the iterator?**:
   - Fail-fast: Throw an exception if the aggregate is modified during iteration.
   - Snapshot: Iterate over a copy of the aggregate, immune to modifications.
   - Robust: Track modifications and adjust position accordingly.

4. **Additional iterator operations**:
   - `peek()`: Return next element without advancing.
   - `reset()`: Return to the beginning.
   - `remove()`: Remove the current element.
   - `skip(n)`: Skip n elements.

### Pseudocode: Basic Iterator Infrastructure

```
// Iterator interface
interface Iterator<T> {
    method hasNext(): boolean
    method next(): T
    method reset(): void
}

// Extended iterator with more operations
interface ExtendedIterator<T> extends Iterator<T> {
    method peek(): T           // Look without advancing
    method skip(count: int): void
    method remaining(): int    // Count of remaining elements
    method toList(): list<T>   // Consume remaining into list
}

// Iterable interface (aggregate)
interface Iterable<T> {
    method iterator(): Iterator<T>
}

// Bidirectional iterator
interface BidirectionalIterator<T> extends Iterator<T> {
    method hasPrevious(): boolean
    method previous(): T
}
```

### Pseudocode: Array Iterator

```
class ArrayList<T> implements Iterable<T> {
    private elements: array<T>
    private size: int

    method add(element: T): void {
        // Add to array, resize if needed
    }

    method get(index: int): T {
        return this.elements[index]
    }

    method size(): int {
        return this.size
    }

    method iterator(): Iterator<T> {
        return new ArrayListIterator(this)
    }

    method reverseIterator(): Iterator<T> {
        return new ReverseArrayListIterator(this)
    }
}

class ArrayListIterator<T> implements Iterator<T> {
    private list: ArrayList<T>
    private currentIndex: int = 0

    constructor(list: ArrayList<T>) {
        this.list = list
    }

    method hasNext(): boolean {
        return this.currentIndex < this.list.size()
    }

    method next(): T {
        if not this.hasNext() {
            throw "No more elements"
        }
        element = this.list.get(this.currentIndex)
        this.currentIndex = this.currentIndex + 1
        return element
    }

    method reset(): void {
        this.currentIndex = 0
    }
}

class ReverseArrayListIterator<T> implements Iterator<T> {
    private list: ArrayList<T>
    private currentIndex: int

    constructor(list: ArrayList<T>) {
        this.list = list
        this.currentIndex = list.size() - 1
    }

    method hasNext(): boolean {
        return this.currentIndex >= 0
    }

    method next(): T {
        if not this.hasNext() {
            throw "No more elements"
        }
        element = this.list.get(this.currentIndex)
        this.currentIndex = this.currentIndex - 1
        return element
    }

    method reset(): void {
        this.currentIndex = this.list.size() - 1
    }
}
```

### Pseudocode: Tree Iterator

```
class TreeNode<T> {
    public value: T
    public children: list<TreeNode<T>> = []

    constructor(value: T) {
        this.value = value
    }

    method addChild(child: TreeNode<T>): void {
        this.children.add(child)
    }
}

class Tree<T> implements Iterable<T> {
    private root: TreeNode<T>

    constructor(root: TreeNode<T>) {
        this.root = root
    }

    method getRoot(): TreeNode<T> {
        return this.root
    }

    // Default iterator uses depth-first pre-order
    method iterator(): Iterator<T> {
        return new DepthFirstIterator(this.root)
    }

    method breadthFirstIterator(): Iterator<T> {
        return new BreadthFirstIterator(this.root)
    }

    method depthFirstIterator(): Iterator<T> {
        return new DepthFirstIterator(this.root)
    }
}

// Depth-first pre-order traversal
class DepthFirstIterator<T> implements Iterator<T> {
    private stack: Stack<TreeNode<T>>

    constructor(root: TreeNode<T>) {
        this.stack = new Stack()
        if root is not null {
            this.stack.push(root)
        }
    }

    method hasNext(): boolean {
        return not this.stack.isEmpty()
    }

    method next(): T {
        if not this.hasNext() {
            throw "No more elements"
        }

        node = this.stack.pop()

        // Push children in reverse order so leftmost is processed first
        for i = node.children.size() - 1 downto 0 {
            this.stack.push(node.children.get(i))
        }

        return node.value
    }

    method reset(): void {
        // Would need reference to root to implement
        throw "Reset not supported"
    }
}

// Breadth-first (level-order) traversal
class BreadthFirstIterator<T> implements Iterator<T> {
    private queue: Queue<TreeNode<T>>

    constructor(root: TreeNode<T>) {
        this.queue = new Queue()
        if root is not null {
            this.queue.enqueue(root)
        }
    }

    method hasNext(): boolean {
        return not this.queue.isEmpty()
    }

    method next(): T {
        if not this.hasNext() {
            throw "No more elements"
        }

        node = this.queue.dequeue()

        // Enqueue all children
        for child in node.children {
            this.queue.enqueue(child)
        }

        return node.value
    }

    method reset(): void {
        throw "Reset not supported"
    }
}
```

### Pseudocode: Filtering Iterator

```
// Iterator decorator that filters elements
class FilterIterator<T> implements Iterator<T> {
    private source: Iterator<T>
    private predicate: function(T): boolean
    private nextElement: T = null
    private hasNextElement: boolean = false

    constructor(source: Iterator<T>, predicate: function(T): boolean) {
        this.source = source
        this.predicate = predicate
        this.advance()
    }

    method hasNext(): boolean {
        return this.hasNextElement
    }

    method next(): T {
        if not this.hasNext() {
            throw "No more elements"
        }
        result = this.nextElement
        this.advance()
        return result
    }

    private method advance(): void {
        this.hasNextElement = false
        while this.source.hasNext() {
            element = this.source.next()
            if this.predicate(element) {
                this.nextElement = element
                this.hasNextElement = true
                return
            }
        }
    }

    method reset(): void {
        this.source.reset()
        this.advance()
    }
}

// Usage
list = new ArrayList<int>()
list.add(1); list.add(2); list.add(3); list.add(4); list.add(5)

evenIterator = new FilterIterator(
    list.iterator(),
    (n) => n % 2 == 0
)

while evenIterator.hasNext() {
    print(evenIterator.next())  // Prints 2, 4
}
```

## Example

A complete example implementing iterators for a social network:

```
// User in the social network
class User {
    public id: string
    public name: string
    public email: string
    public friends: list<string> = []        // Friend user IDs
    public followers: list<string> = []       // Follower user IDs
    public following: list<string> = []       // Users this user follows

    constructor(id: string, name: string, email: string) {
        this.id = id
        this.name = name
        this.email = email
    }
}

// Profile iterator interface
interface ProfileIterator extends Iterator<User> {
    method hasNext(): boolean
    method next(): User
    method reset(): void
}

// Social network interface (aggregate)
interface SocialNetwork {
    method createFriendsIterator(userId: string): ProfileIterator
    method createFollowersIterator(userId: string): ProfileIterator
    method createFriendsOfFriendsIterator(userId: string): ProfileIterator
}

// Concrete social network implementation
class Facebook implements SocialNetwork {
    private users: map<string, User> = {}

    method addUser(user: User): void {
        this.users.set(user.id, user)
    }

    method getUser(userId: string): User {
        return this.users.get(userId)
    }

    method createFriendsIterator(userId: string): ProfileIterator {
        return new FacebookFriendsIterator(this, userId)
    }

    method createFollowersIterator(userId: string): ProfileIterator {
        return new FacebookFollowersIterator(this, userId)
    }

    method createFriendsOfFriendsIterator(userId: string): ProfileIterator {
        return new FacebookFriendsOfFriendsIterator(this, userId)
    }

    // Internal method for iterators to use
    method getUserFriendIds(userId: string): list<string> {
        user = this.users.get(userId)
        return user is not null ? user.friends : []
    }

    method getUserFollowerIds(userId: string): list<string> {
        user = this.users.get(userId)
        return user is not null ? user.followers : []
    }
}

// Iterator for direct friends
class FacebookFriendsIterator implements ProfileIterator {
    private network: Facebook
    private userId: string
    private friendIds: list<string>
    private currentIndex: int = 0

    constructor(network: Facebook, userId: string) {
        this.network = network
        this.userId = userId
        this.friendIds = network.getUserFriendIds(userId)
    }

    method hasNext(): boolean {
        return this.currentIndex < this.friendIds.size()
    }

    method next(): User {
        if not this.hasNext() {
            throw "No more friends"
        }
        friendId = this.friendIds.get(this.currentIndex)
        this.currentIndex = this.currentIndex + 1
        return this.network.getUser(friendId)
    }

    method reset(): void {
        this.currentIndex = 0
    }
}

// Iterator for followers
class FacebookFollowersIterator implements ProfileIterator {
    private network: Facebook
    private userId: string
    private followerIds: list<string>
    private currentIndex: int = 0

    constructor(network: Facebook, userId: string) {
        this.network = network
        this.userId = userId
        this.followerIds = network.getUserFollowerIds(userId)
    }

    method hasNext(): boolean {
        return this.currentIndex < this.followerIds.size()
    }

    method next(): User {
        if not this.hasNext() {
            throw "No more followers"
        }
        followerId = this.followerIds.get(this.currentIndex)
        this.currentIndex = this.currentIndex + 1
        return this.network.getUser(followerId)
    }

    method reset(): void {
        this.currentIndex = 0
    }
}

// Iterator for friends of friends (2 degrees of separation)
class FacebookFriendsOfFriendsIterator implements ProfileIterator {
    private network: Facebook
    private userId: string
    private visited: set<string> = {}
    private queue: Queue<string> = new Queue()
    private currentDepth: map<string, int> = {}

    constructor(network: Facebook, userId: string) {
        this.network = network
        this.userId = userId
        this.initialize()
    }

    private method initialize(): void {
        this.visited.clear()
        this.queue.clear()
        this.currentDepth.clear()

        // Mark the starting user as visited
        this.visited.add(this.userId)

        // Add direct friends at depth 1
        for friendId in this.network.getUserFriendIds(this.userId) {
            if not this.visited.contains(friendId) {
                this.visited.add(friendId)
                this.queue.enqueue(friendId)
                this.currentDepth.set(friendId, 1)
            }
        }

        // Expand to depth 2 (friends of friends)
        this.expandToNextLevel()
    }

    private method expandToNextLevel(): void {
        // Get all current depth-1 users and add their friends
        tempList = []
        while not this.queue.isEmpty() {
            tempList.add(this.queue.dequeue())
        }

        for friendId in tempList {
            this.queue.enqueue(friendId)  // Re-add to queue

            depth = this.currentDepth.get(friendId)
            if depth < 2 {
                // Add friends of this friend
                for fofId in this.network.getUserFriendIds(friendId) {
                    if not this.visited.contains(fofId) {
                        this.visited.add(fofId)
                        this.queue.enqueue(fofId)
                        this.currentDepth.set(fofId, depth + 1)
                    }
                }
            }
        }
    }

    method hasNext(): boolean {
        return not this.queue.isEmpty()
    }

    method next(): User {
        if not this.hasNext() {
            throw "No more users"
        }
        userId = this.queue.dequeue()
        return this.network.getUser(userId)
    }

    method reset(): void {
        this.initialize()
    }
}

// Lazy paginated iterator for remote data
class PaginatedFollowersIterator implements ProfileIterator {
    private network: Facebook
    private userId: string
    private pageSize: int
    private currentPage: list<User> = []
    private pageIndex: int = 0
    private globalIndex: int = 0
    private hasMorePages: boolean = true

    constructor(network: Facebook, userId: string, pageSize: int = 50) {
        this.network = network
        this.userId = userId
        this.pageSize = pageSize
        this.loadNextPage()
    }

    private method loadNextPage(): void {
        // Simulate loading a page from remote API
        allFollowers = this.network.getUserFollowerIds(this.userId)
        startIndex = this.globalIndex
        endIndex = min(startIndex + this.pageSize, allFollowers.size())

        this.currentPage = []
        for i = startIndex to endIndex - 1 {
            user = this.network.getUser(allFollowers.get(i))
            this.currentPage.add(user)
        }

        this.pageIndex = 0
        this.hasMorePages = endIndex < allFollowers.size()
    }

    method hasNext(): boolean {
        if this.pageIndex < this.currentPage.size() {
            return true
        }
        if this.hasMorePages {
            this.loadNextPage()
            return this.pageIndex < this.currentPage.size()
        }
        return false
    }

    method next(): User {
        if not this.hasNext() {
            throw "No more followers"
        }
        user = this.currentPage.get(this.pageIndex)
        this.pageIndex = this.pageIndex + 1
        this.globalIndex = this.globalIndex + 1
        return user
    }

    method reset(): void {
        this.globalIndex = 0
        this.currentPage = []
        this.pageIndex = 0
        this.hasMorePages = true
        this.loadNextPage()
    }
}

// Application that uses iterators
class SocialSpammer {
    private network: SocialNetwork

    constructor(network: SocialNetwork) {
        this.network = network
    }

    method sendSpamToFriends(userId: string, message: string): void {
        iterator = this.network.createFriendsIterator(userId)
        this.sendToAll(iterator, message)
    }

    method sendSpamToFollowers(userId: string, message: string): void {
        iterator = this.network.createFollowersIterator(userId)
        this.sendToAll(iterator, message)
    }

    method sendSpamToNetwork(userId: string, message: string): void {
        iterator = this.network.createFriendsOfFriendsIterator(userId)
        this.sendToAll(iterator, message)
    }

    // Works with any ProfileIterator - doesn't care about the source
    private method sendToAll(iterator: ProfileIterator, message: string): void {
        while iterator.hasNext() {
            user = iterator.next()
            this.sendEmail(user.email, message)
        }
    }

    private method sendEmail(email: string, message: string): void {
        print("Sending to " + email + ": " + message)
    }
}

// Utility functions for iterators
class IteratorUtils {
    // Count elements
    static method count<T>(iterator: Iterator<T>): int {
        count = 0
        while iterator.hasNext() {
            iterator.next()
            count = count + 1
        }
        return count
    }

    // Collect into list
    static method toList<T>(iterator: Iterator<T>): list<T> {
        result = []
        while iterator.hasNext() {
            result.add(iterator.next())
        }
        return result
    }

    // Find first matching element
    static method find<T>(iterator: Iterator<T>, predicate: function(T): boolean): T {
        while iterator.hasNext() {
            element = iterator.next()
            if predicate(element) {
                return element
            }
        }
        return null
    }

    // Apply function to each element
    static method forEach<T>(iterator: Iterator<T>, action: function(T): void): void {
        while iterator.hasNext() {
            action(iterator.next())
        }
    }

    // Transform elements
    static method map<T, R>(iterator: Iterator<T>, transform: function(T): R): Iterator<R> {
        return new MappingIterator(iterator, transform)
    }

    // Filter elements
    static method filter<T>(iterator: Iterator<T>, predicate: function(T): boolean): Iterator<T> {
        return new FilterIterator(iterator, predicate)
    }

    // Take first n elements
    static method take<T>(iterator: Iterator<T>, count: int): Iterator<T> {
        return new TakeIterator(iterator, count)
    }
}

// Mapping iterator (transforms elements)
class MappingIterator<T, R> implements Iterator<R> {
    private source: Iterator<T>
    private transform: function(T): R

    constructor(source: Iterator<T>, transform: function(T): R) {
        this.source = source
        this.transform = transform
    }

    method hasNext(): boolean {
        return this.source.hasNext()
    }

    method next(): R {
        return this.transform(this.source.next())
    }

    method reset(): void {
        this.source.reset()
    }
}

// Take iterator (limits elements)
class TakeIterator<T> implements Iterator<T> {
    private source: Iterator<T>
    private limit: int
    private taken: int = 0

    constructor(source: Iterator<T>, limit: int) {
        this.source = source
        this.limit = limit
    }

    method hasNext(): boolean {
        return this.taken < this.limit and this.source.hasNext()
    }

    method next(): T {
        if not this.hasNext() {
            throw "No more elements"
        }
        this.taken = this.taken + 1
        return this.source.next()
    }

    method reset(): void {
        this.source.reset()
        this.taken = 0
    }
}

// Usage example
function main() {
    // Set up social network
    facebook = new Facebook()

    alice = new User("1", "Alice", "alice@example.com")
    bob = new User("2", "Bob", "bob@example.com")
    charlie = new User("3", "Charlie", "charlie@example.com")
    diana = new User("4", "Diana", "diana@example.com")

    alice.friends = ["2", "3"]  // Alice is friends with Bob and Charlie
    bob.friends = ["1", "4"]    // Bob is friends with Alice and Diana
    charlie.friends = ["1"]     // Charlie is friends with Alice

    facebook.addUser(alice)
    facebook.addUser(bob)
    facebook.addUser(charlie)
    facebook.addUser(diana)

    // Use the spammer with different iterators
    spammer = new SocialSpammer(facebook)

    print("--- Spamming Alice's friends ---")
    spammer.sendSpamToFriends("1", "Check out this deal!")
    // Sends to Bob, Charlie

    print("--- Spamming Alice's network ---")
    spammer.sendSpamToNetwork("1", "Join our MLM!")
    // Sends to Bob, Charlie, Diana (friend of friend)

    // Using iterator utilities
    friendsIterator = facebook.createFriendsIterator("1")
    emails = IteratorUtils.map(friendsIterator, (user) => user.email)
    print(IteratorUtils.toList(emails))  // ["bob@example.com", "charlie@example.com"]
}
```

## Known Uses

- **Java Collections Framework**: `java.util.Iterator` is the standard interface. All collections implement `Iterable` and provide iterators.

- **C++ Standard Template Library (STL)**: Iterators are fundamental to STL algorithms. Types include input, output, forward, bidirectional, and random access iterators.

- **Python Iterators**: Python's `__iter__` and `__next__` methods implement the Iterator pattern. Generator functions create iterators automatically.

- **JavaScript Iterators**: The Symbol.iterator protocol and for...of loops use iterators. Generators (`function*`) create iterators.

- **LINQ (C#)**: Language Integrated Query uses iterators extensively for lazy evaluation of query chains.

- **Ruby Enumerators**: Ruby's Enumerable module and Enumerator class implement iterator functionality.

- **Database Cursors**: JDBC ResultSet, MongoDB cursors, and similar database APIs are iterators over query results.

- **File System APIs**: Directory iteration in most languages uses iterators to avoid loading entire directory listings into memory.

- **Stream APIs**: Java 8 Streams, Reactive Streams, and similar APIs use iterator-like patterns with additional operations.

## Related Patterns

- **Composite**: Iterators are often used to traverse Composite structures. The iterator handles the recursive traversal transparently.

- **Factory Method**: Polymorphic iterators rely on factory methods. The `createIterator()` method is a factory method that returns an iterator.

- **Memento**: An iterator can use a Memento to capture its state, allowing traversal to be saved and restored.

- **Visitor**: Both traverse structures. Iterator gives you elements one at a time; Visitor applies operations to each element. They can be combined: iterator provides elements, visitor processes them.

- **Null Object**: A NullIterator that always returns `hasNext() = false` can represent an empty collection.

## When NOT to Use

1. **Simple indexed access is sufficient**: If you're just looping through an array with `for (i = 0; i < array.length; i++)`, adding an iterator layer doesn't help.

```
// Overkill: Iterator for simple array access
iterator = array.iterator()
while iterator.hasNext() {
    process(iterator.next())
}

// Just use the array directly
for i = 0 to array.length - 1 {
    process(array[i])
}
```

2. **Random access is required**: If you need to jump to arbitrary positions, iterators' sequential nature is limiting. Use indexed access instead.

3. **Modification during iteration**: If you need to modify the collection while iterating (beyond simple removal), iterators become problematic. Consider building a new collection instead.

4. **Performance-critical tight loops**: Iterator method calls add overhead. For extremely hot loops, direct array access may be measurably faster.

5. **When language provides better alternatives**: Modern languages often have comprehensions, map/filter/reduce, or LINQ-style queries that are more expressive than explicit iteration.

```
// Less expressive: Explicit iterator
result = []
iterator = list.iterator()
while iterator.hasNext() {
    item = iterator.next()
    if item.isActive() {
        result.add(item.name)
    }
}

// More expressive: Functional style (if language supports it)
result = list.filter(item => item.isActive())
             .map(item => item.name)
```

6. **Trivial collections**: For collections you know will always have 0-3 elements, the iterator infrastructure is overkill.

The Iterator pattern is essential for abstracting traversal across diverse collection types and enabling lazy evaluation. It's less necessary when dealing with simple, homogeneous arrays or when the language provides higher-level iteration constructs.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
