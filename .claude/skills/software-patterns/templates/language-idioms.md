# Language Idioms for Pattern Implementation

## Core Principles

Each language has idiomatic ways to express design patterns. Direct translation often feels unnatural. This guide shows how to adapt patterns to language strengths.

---

## TypeScript/JavaScript

### Key Characteristics
- Prototype-based with class syntax sugar
- First-class functions
- Duck typing (JS) / structural typing (TS)
- Async-first with promises

### Pattern Adaptations

#### Singleton
```typescript
// Anti-pattern (mutable global)
class Singleton {
  private static instance: Singleton;
  static getInstance() { /* ... */ }
}

// Idiomatic (module scope + DI)
export const config = createConfig();

// Or with lazy initialization
let config: Config | null = null;
export const getConfig = () => config ??= createConfig();
```

#### Strategy
```typescript
// Classical (objects)
interface Strategy { execute(): void; }
class Context {
  constructor(private strategy: Strategy) {}
}

// Idiomatic (functions)
type Strategy = () => void;
class Context {
  constructor(private strategy: Strategy) {}
}

// Or just plain functions
const strategies = {
  fast: (data) => { /* ... */ },
  thorough: (data) => { /* ... */ }
};
```

#### Factory Method
```typescript
// Classical (abstract method)
abstract class Creator {
  abstract createProduct(): Product;
}

// Idiomatic (function returning function)
const createFactory = (config: Config) => {
  return (type: string) => {
    // Factory logic
  };
};
```

#### Observer
```typescript
// Classical (manual observer list)
class Subject {
  private observers: Observer[] = [];
  attach(o: Observer) { /* ... */ }
}

// Idiomatic (EventEmitter or RxJS)
import { EventEmitter } from 'events';
const subject = new EventEmitter();
subject.on('change', handler);

// Or reactive
import { BehaviorSubject } from 'rxjs';
const state$ = new BehaviorSubject(initialState);
```

#### Builder
```typescript
// Classical (mutable builder)
class Builder {
  withName(name: string) { this.name = name; return this; }
}

// Idiomatic (immutable builder or just options)
interface Options {
  name?: string;
  age?: number;
}
const user = createUser({ name: 'Alice', age: 30 });

// Or fluent immutable
const user = UserBuilder.create()
  .withName('Alice')
  .withAge(30)
  .build();
```

### Language Features to Leverage
- **Interfaces for contracts**: Structural typing is powerful
- **Union types for state**: `type State = 'idle' | 'loading' | 'error'`
- **Optional chaining**: Simplifies null handling
- **Async/await**: Makes async patterns cleaner
- **Destructuring**: Simplifies parameter objects

---

## Python

### Key Characteristics
- Duck typing ("easier to ask forgiveness")
- Multiple paradigms (OOP, functional, procedural)
- Protocols over explicit interfaces
- Context managers for resources
- Decorators for cross-cutting

### Pattern Adaptations

#### Singleton
```python
# Anti-pattern (metaclass complexity)
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        # ...

# Idiomatic (module-level)
# config.py
_config = None

def get_config():
    global _config
    if _config is None:
        _config = Config()
    return _config

# Or decorator
from functools import lru_cache

@lru_cache(maxsize=1)
def get_config():
    return Config()
```

#### Strategy
```python
# Classical (ABC)
from abc import ABC, abstractmethod

class Strategy(ABC):
    @abstractmethod
    def execute(self):
        pass

# Idiomatic (Protocol or just duck typing)
from typing import Protocol

class Strategy(Protocol):
    def execute(self) -> None: ...

# Or just functions
def strategy_a(data): ...
def strategy_b(data): ...
strategies = {'a': strategy_a, 'b': strategy_b}
```

#### Factory Method
```python
# Classical (abstract method)
class Creator(ABC):
    @abstractmethod
    def create_product(self):
        pass

# Idiomatic (class method or function)
class Product:
    @classmethod
    def from_config(cls, config):
        # Factory logic
        return cls(...)

# Or registry pattern
_registry = {}

def register(name):
    def decorator(cls):
        _registry[name] = cls
        return cls
    return decorator

@register('basic')
class BasicProduct:
    pass
```

#### Builder
```python
# Classical (method chaining)
class Builder:
    def with_name(self, name):
        self.name = name
        return self

# Idiomatic (dataclass with defaults or factory)
from dataclasses import dataclass, field

@dataclass
class User:
    name: str
    age: int = 0
    email: str = ''

# Or builder with **kwargs
def build_user(**kwargs):
    defaults = {'age': 0, 'email': ''}
    return User(**(defaults | kwargs))
```

#### Template Method
```python
# Classical (abstract methods)
class Base(ABC):
    def template_method(self):
        self.step1()
        self.step2()

    @abstractmethod
    def step1(self): pass

# Idiomatic (hooks with default)
class Base:
    def template_method(self):
        self.step1()
        self.step2()

    def step1(self):
        pass  # Default implementation
```

#### Proxy/Decorator Pattern
```python
# Classical (manual delegation)
class Proxy:
    def __init__(self, subject):
        self._subject = subject

    def request(self):
        # Before logic
        result = self._subject.request()
        # After logic
        return result

# Idiomatic (function decorator or __getattr__)
def logging_proxy(func):
    def wrapper(*args, **kwargs):
        print(f'Calling {func.__name__}')
        return func(*args, **kwargs)
    return wrapper

# Or transparent proxy
class Proxy:
    def __init__(self, subject):
        self._subject = subject

    def __getattr__(self, name):
        # Intercept all attribute access
        attr = getattr(self._subject, name)
        if callable(attr):
            return lambda *args, **kwargs: self._intercept(attr, *args, **kwargs)
        return attr
```

#### Context Manager (Resource Acquisition)
```python
# Idiomatic Python pattern
class Resource:
    def __enter__(self):
        # Acquire resource
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # Release resource
        pass

# Or with decorator
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire()
    try:
        yield resource
    finally:
        release(resource)

# Usage
with managed_resource() as r:
    r.use()
```

### Language Features to Leverage
- **Protocols (PEP 544)**: Structural subtyping
- **Dataclasses**: Reduce boilerplate for value objects
- **Decorators**: For cross-cutting concerns (logging, timing, caching)
- **Context managers**: For resource management patterns
- **Duck typing**: Avoid unnecessary abstract base classes
- **`__getattr__`/`__getattribute__`**: For transparent proxies

---

## Java

### Key Characteristics
- Nominal typing (explicit interfaces)
- Strong static typing
- Rich standard library
- Verbose but explicit
- Modern features: records, sealed classes, pattern matching

### Pattern Adaptations

#### Singleton
```java
// Classical (eager initialization)
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() { return INSTANCE; }
}

// Thread-safe lazy (double-check locking)
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// Idiomatic (enum)
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}

// Modern (Spring/CDI)
@Singleton  // Let DI container manage lifecycle
public class Service {
    // ...
}
```

#### Strategy
```java
// Classical (interface + implementations)
interface Strategy {
    void execute();
}

class ConcreteStrategyA implements Strategy {
    public void execute() { /* ... */ }
}

// Modern (functional interface + lambdas)
@FunctionalInterface
interface Strategy {
    void execute();
}

// Usage
Strategy strategy = () -> { /* ... */ };

// Or method reference
Strategy strategy = this::methodName;
```

#### Builder
```java
// Classical (mutable builder)
public class Builder {
    private String name;
    public Builder withName(String name) {
        this.name = name;
        return this;
    }
}

// Idiomatic (nested static builder)
public class User {
    private final String name;
    private final int age;

    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
    }

    public static class Builder {
        private String name;
        private int age;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// Modern (record with builder)
public record User(String name, int age) {
    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        // Same as above
    }
}
```

#### State Pattern
```java
// Classical (interface + implementations)
interface State {
    void handle(Context context);
}

// Modern (sealed classes + pattern matching)
public sealed interface State
    permits IdleState, ProcessingState, ErrorState {

    void handle(Context context);
}

// Usage with pattern matching (Java 17+)
String description = switch (state) {
    case IdleState s -> "Waiting";
    case ProcessingState s -> "Working";
    case ErrorState s -> "Failed: " + s.message();
};
```

#### Factory Method
```java
// Classical (abstract method)
abstract class Creator {
    abstract Product createProduct();
}

// Modern (static factory methods)
public class Product {
    private Product() {}

    public static Product basic() {
        return new Product();
    }

    public static Product premium() {
        return new Product(/* ... */);
    }
}
```

#### Value Object
```java
// Classical (immutable class with equals/hashCode)
public final class Point {
    private final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // getters, equals, hashCode, toString
}

// Modern (record - Java 14+)
public record Point(int x, int y) {
    // Automatically gets: constructor, getters, equals, hashCode, toString
}
```

### Language Features to Leverage
- **Records**: Immutable value objects with minimal boilerplate
- **Sealed classes**: Closed hierarchies for state machines
- **Pattern matching**: Cleaner visitor/state implementations
- **Functional interfaces**: Strategy, Command as lambdas
- **Optional**: Null handling without Null Object pattern
- **Streams**: Internal iteration over external iteration

---

## Go

### Key Characteristics
- Interface satisfaction is implicit
- Composition over inheritance
- Explicit error handling
- Concurrency primitives (goroutines, channels)
- No classes, no inheritance

### Pattern Adaptations

#### Singleton
```go
// Anti-pattern (global mutable state)
var instance *Config

func GetConfig() *Config {
    if instance == nil {
        instance = &Config{}
    }
    return instance
}

// Idiomatic (sync.Once)
var (
    instance *Config
    once     sync.Once
)

func GetConfig() *Config {
    once.Do(func() {
        instance = &Config{}
    })
    return instance
}

// Or just package-level variable with init()
var config *Config

func init() {
    config = loadConfig()
}
```

#### Strategy
```go
// Classical (interface)
type Strategy interface {
    Execute()
}

type Context struct {
    strategy Strategy
}

// Idiomatic (function type)
type Strategy func()

type Context struct {
    strategy Strategy
}

// Usage
ctx := Context{
    strategy: func() {
        // Implementation
    },
}
```

#### Factory
```go
// Classical (interface + factory function)
type Product interface {
    Use()
}

func NewProduct(kind string) Product {
    switch kind {
    case "a":
        return &ConcreteProductA{}
    case "b":
        return &ConcreteProductB{}
    }
    return nil
}

// Idiomatic (functional options)
type Option func(*Config)

func WithName(name string) Option {
    return func(c *Config) {
        c.Name = name
    }
}

func NewServer(opts ...Option) *Server {
    cfg := &Config{} // defaults
    for _, opt := range opts {
        opt(cfg)
    }
    return &Server{config: cfg}
}

// Usage
server := NewServer(
    WithName("api"),
    WithPort(8080),
)
```

#### Builder
```go
// Go doesn't use classical builder
// Instead: functional options (see above) or struct literals

type User struct {
    Name  string
    Age   int
    Email string
}

// Usage with struct literal
user := User{
    Name:  "Alice",
    Age:   30,
    Email: "alice@example.com",
}

// Or constructor with options
func NewUser(name string, opts ...UserOption) *User {
    u := &User{Name: name}
    for _, opt := range opts {
        opt(u)
    }
    return u
}
```

#### Observer
```go
// Classical (manual observers)
type Observer interface {
    Update()
}

type Subject struct {
    observers []Observer
}

// Idiomatic (channels)
type EventBus struct {
    subscribers []chan Event
}

func (eb *EventBus) Subscribe() <-chan Event {
    ch := make(chan Event, 10)
    eb.subscribers = append(eb.subscribers, ch)
    return ch
}

func (eb *EventBus) Publish(e Event) {
    for _, ch := range eb.subscribers {
        ch <- e  // Or select with default for non-blocking
    }
}

// Usage
events := bus.Subscribe()
go func() {
    for event := range events {
        // Handle event
    }
}()
```

#### Decorator/Middleware
```go
// Idiomatic (function wrapping)
type Handler func(http.ResponseWriter, *http.Request)

func LoggingMiddleware(next Handler) Handler {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println("Request:", r.URL)
        next(w, r)
        log.Println("Response sent")
    }
}

// Chaining
handler = LoggingMiddleware(AuthMiddleware(handler))
```

#### Template Method
```go
// Go doesn't have inheritance
// Use composition with interface

type Step interface {
    Execute()
}

type Algorithm struct {
    step1 Step
    step2 Step
}

func (a *Algorithm) Run() {
    a.step1.Execute()
    a.step2.Execute()
}
```

#### Adapter
```go
// Idiomatic (implicit interface satisfaction)
type LegacySystem struct{}

func (ls *LegacySystem) OldMethod() string {
    return "data"
}

type ModernInterface interface {
    NewMethod() string
}

// Adapter (just embeds and adds methods)
type Adapter struct {
    *LegacySystem
}

func (a *Adapter) NewMethod() string {
    return a.OldMethod() // Adapt
}

// Satisfies ModernInterface automatically
var _ ModernInterface = (*Adapter)(nil)
```

### Language Features to Leverage
- **Implicit interfaces**: No explicit "implements", just satisfy methods
- **Embedding**: Composition without inheritance
- **Channels**: For observer/pub-sub patterns
- **Functional options**: For builders/configuration
- **Error values**: Explicit error handling
- **Goroutines**: Concurrent observers, workers

---

## Rust

### Key Characteristics
- Ownership and borrowing
- Trait-based polymorphism
- Enum-based state machines
- Zero-cost abstractions
- Memory safety without GC

### Pattern Adaptations

#### Singleton
```rust
// Anti-pattern (mutable global with lazy_static)
use lazy_static::lazy_static;
use std::sync::Mutex;

lazy_static! {
    static ref INSTANCE: Mutex<Config> = Mutex::new(Config::new());
}

// Idiomatic (thread-local or OnceCell)
use std::sync::OnceLock;

static CONFIG: OnceLock<Config> = OnceLock::new();

fn get_config() -> &'static Config {
    CONFIG.get_or_init(|| Config::new())
}

// Or dependency injection (pass references)
```

#### Strategy
```rust
// Classical (trait object)
trait Strategy {
    fn execute(&self);
}

struct Context {
    strategy: Box<dyn Strategy>,
}

// Idiomatic (generic with trait bound)
struct Context<S: Strategy> {
    strategy: S,
}

// Or function pointer
type Strategy = fn();

struct Context {
    strategy: Strategy,
}

// Or closure
struct Context<F: Fn()> {
    strategy: F,
}
```

#### Builder
```rust
// Classical (mutable builder)
pub struct Builder {
    name: Option<String>,
    age: Option<u32>,
}

impl Builder {
    pub fn name(mut self, name: String) -> Self {
        self.name = Some(name);
        self
    }

    pub fn build(self) -> Result<User, Error> {
        Ok(User {
            name: self.name.ok_or(Error::MissingName)?,
            age: self.age.unwrap_or(0),
        })
    }
}

// Idiomatic (consuming self for linear type state)
pub struct UserBuilder<State> {
    name: String,
    _state: PhantomData<State>,
}

pub struct NoAge;
pub struct HasAge { age: u32 }

impl UserBuilder<NoAge> {
    pub fn new(name: String) -> Self {
        UserBuilder { name, _state: PhantomData }
    }

    pub fn age(self, age: u32) -> UserBuilder<HasAge> {
        UserBuilder {
            name: self.name,
            _state: PhantomData,
        }
    }
}

impl UserBuilder<HasAge> {
    pub fn build(self) -> User {
        User { name: self.name, age: self._state.age }
    }
}

// Or derive with builder crate
use derive_builder::Builder;

#[derive(Builder)]
pub struct User {
    name: String,
    #[builder(default = "0")]
    age: u32,
}
```

#### State Machine
```rust
// Classical (trait + implementations)
trait State {
    fn handle(self: Box<Self>) -> Box<dyn State>;
}

// Idiomatic (enum + match)
enum State {
    Idle,
    Processing { progress: u32 },
    Done { result: String },
    Error { message: String },
}

impl State {
    fn handle(self) -> State {
        match self {
            State::Idle => State::Processing { progress: 0 },
            State::Processing { progress } if progress < 100 => {
                State::Processing { progress: progress + 10 }
            }
            State::Processing { .. } => State::Done { result: "Complete".into() },
            State::Done { .. } => self,
            State::Error { .. } => self,
        }
    }
}

// Or typestate pattern (compile-time states)
struct Connection<State> {
    addr: String,
    _state: PhantomData<State>,
}

struct Disconnected;
struct Connected;

impl Connection<Disconnected> {
    fn connect(self) -> Connection<Connected> {
        // Connection logic
        Connection {
            addr: self.addr,
            _state: PhantomData,
        }
    }
}

impl Connection<Connected> {
    fn send(&self, data: &[u8]) {
        // Can only call on connected state
    }
}
```

#### Observer
```rust
// Classical (callbacks)
struct Subject {
    observers: Vec<Box<dyn Fn()>>,
}

// Idiomatic (channels or async streams)
use tokio::sync::broadcast;

struct EventBus {
    tx: broadcast::Sender<Event>,
}

impl EventBus {
    fn subscribe(&self) -> broadcast::Receiver<Event> {
        self.tx.subscribe()
    }

    fn publish(&self, event: Event) {
        let _ = self.tx.send(event);
    }
}
```

#### Factory
```rust
// Classical (trait method)
trait Factory {
    fn create(&self) -> Box<dyn Product>;
}

// Idiomatic (associated function or From trait)
impl User {
    pub fn from_config(config: &Config) -> Self {
        User { /* ... */ }
    }
}

// Or use From trait
impl From<Config> for User {
    fn from(config: Config) -> Self {
        User { /* ... */ }
    }
}

// Usage
let user = User::from(config);
// Or
let user: User = config.into();
```

#### Template Method
```rust
// Rust doesn't have inheritance
// Use trait with default methods

trait Algorithm {
    fn step1(&self);
    fn step2(&self) {
        // Default implementation
    }

    fn run(&self) {
        self.step1();
        self.step2();
    }
}
```

#### Decorator
```rust
// Ownership makes classical decorator tricky
// Use newtype wrapper

struct Logger<T> {
    inner: T,
}

impl<T: Service> Service for Logger<T> {
    fn call(&self, req: Request) -> Response {
        println!("Request: {:?}", req);
        let resp = self.inner.call(req);
        println!("Response: {:?}", resp);
        resp
    }
}

// Usage
let service = Logger { inner: BasicService };
```

### Language Features to Leverage
- **Enums with data**: Perfect for state machines
- **Traits**: Polymorphism without inheritance
- **Typestate pattern**: Compile-time state enforcement
- **Ownership**: Ensures cleanup, no need for explicit dispose
- **Pattern matching**: Exhaustive state handling
- **Zero-cost abstractions**: Generic + monomorphization
- **Derive macros**: Reduce boilerplate for common patterns

---

## General Principles Across Languages

### When to Use Classes vs Functions

| Pattern | Classes When | Functions When |
|---------|--------------|----------------|
| Strategy | Multiple methods per strategy | Single method, simple logic |
| Command | Need undo/state | Stateless operations |
| Observer | Need unsubscribe, complex state | Fire-and-forget notifications |
| Factory | Complex construction logic | Simple instantiation |
| Template Method | Shared state across steps | Independent steps |

### Adapting to Type Systems

| Type System | Adaptation |
|-------------|------------|
| Structural (TypeScript, Go) | Interfaces lighter, focus on shape |
| Nominal (Java, C#) | Explicit interfaces, more ceremony |
| Duck (Python, Ruby) | Protocols or conventions over interfaces |
| Trait (Rust) | Generic constraints, zero-cost |

### Memory Model Considerations

| Memory Model | Implications |
|--------------|--------------|
| GC (Java, C#, Go) | Shared references easy, observer cleanup needed |
| Reference counting (Swift, Python) | Weak references for observer pattern |
| Ownership (Rust) | Decorator needs newtype, observer needs channels |

### Concurrency Patterns

| Language | Idiomatic Concurrency |
|----------|----------------------|
| Go | Channels + goroutines for observer, worker pools |
| Rust | async/await + channels, or thread pools |
| Java | ExecutorService, CompletableFuture |
| Python | asyncio, threading (with GIL limitations) |
| JavaScript | Promises, async/await, Web Workers |

### Testing Adaptations

| Language | Test Double Strategy |
|----------|---------------------|
| TypeScript | Test implementations of interfaces |
| Python | Mock with unittest.mock or pytest monkeypatch |
| Java | Mockito for interface mocks |
| Go | Interface mocks with testify or manual |
| Rust | Trait implementations for testing |
