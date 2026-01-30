# Singleton

## Intent

Ensure a class has only one instance, and provide a global point of access to it. The Singleton pattern controls object creation, limiting the number of instances to exactly one while providing a way to access that instance from anywhere in the application.

## Also Known As

- Single Instance
- Global Instance

## Motivation

Consider a logging system for a large application. Multiple components need to write log messages, and those messages should go to the same log file in a coordinated way. If each component created its own logger instance, you'd have multiple file handles competing for the same file, messages appearing out of order, and resource waste from duplicate initialization.

The application needs exactly one logger instance that coordinates all logging activity. The logger must initialize itself (opening the file, setting up formatters), and all parts of the application must use that same instance. Creating multiple instances would be wasteful and could cause conflicts.

The Singleton pattern solves this by making the class itself responsible for keeping track of its sole instance. The class can ensure that no other instance can be created by making the constructor private, and it provides a global access point through a static method. The first call to this method creates the instance; subsequent calls return the same instance.

This pattern is also useful for configuration managers (one set of settings for the entire application), connection pools (one pool shared by all database operations), and caches (one shared cache to avoid duplicating data).

## Applicability

Use the Singleton pattern when:

- There must be exactly one instance of a class, and it must be accessible to clients from a well-known access point
- The sole instance should be extensible by subclassing, and clients should be able to use an extended instance without modifying their code
- You want to control access to a shared resource (file, database connection, etc.)
- A class should manage state that is global to the application
- Creating multiple instances would cause conflicts or be wasteful
- You need lazy initialization of a resource-intensive object

## Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              SINGLETON                                     │
├────────────────────────────────────────────────────────────────────────────┤
│ - instance: Singleton = null        «static»                               │
│ - data: SomeType                                                           │
├────────────────────────────────────────────────────────────────────────────┤
│ - Singleton()                       «private constructor»                  │
│ + getInstance(): Singleton          «static»                               │
│ + operation(): void                                                        │
│ + getData(): SomeType                                                      │
└────────────────────────────────────────────────────────────────────────────┘
        │
        │ returns same instance
        ▼
┌─────────────────┐
│ «sole instance» │
│    Singleton    │
└─────────────────┘


THREAD-SAFE VARIATIONS:

┌───────────────────────────────────────────────────────────────────────────────┐
│ EAGER INITIALIZATION                                                          │
├───────────────────────────────────────────────────────────────────────────────┤
│ - instance: Singleton = new Singleton()  «static, initialized at load time»  │
│ + getInstance(): Singleton { return instance }                                │
└───────────────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────────────┐
│ LAZY WITH DOUBLE-CHECKED LOCKING                                              │
├───────────────────────────────────────────────────────────────────────────────┤
│ - instance: volatile Singleton = null                                         │
│ + getInstance(): Singleton {                                                  │
│     if (instance == null) {              // First check (no locking)         │
│       synchronized(lock) {                                                    │
│         if (instance == null) {          // Second check (with lock)         │
│           instance = new Singleton()                                          │
│         }                                                                     │
│       }                                                                       │
│     }                                                                         │
│     return instance                                                           │
│   }                                                                           │
└───────────────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────────────┐
│ INITIALIZATION-ON-DEMAND HOLDER (Bill Pugh)                                   │
├───────────────────────────────────────────────────────────────────────────────┤
│ + getInstance(): Singleton { return Holder.INSTANCE }                         │
│                                                                               │
│ private static class Holder {                                                 │
│   static final INSTANCE = new Singleton()                                     │
│ }                                                                             │
└───────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Singleton** (Logger, Configuration, ConnectionPool)
  - Defines a getInstance() operation that lets clients access its unique instance. getInstance() is a class operation (static method)
  - May be responsible for creating its own unique instance
  - Contains the business logic for the single instance

- **Client**
  - Accesses a Singleton instance solely through Singleton's getInstance() operation

## Collaborations

- Clients access a Singleton instance solely through Singleton's getInstance() operation.

- The Singleton class controls when and how its instance is created and accessed.

- Clients have no way to create additional instances—the constructor is private.

## Consequences

### Benefits

- **Controlled access to sole instance**: Because the Singleton class encapsulates its sole instance, it can have strict control over how and when clients access it.

- **Reduced namespace pollution**: The Singleton pattern is an improvement over global variables. It avoids polluting the namespace with global variables that store sole instances.

- **Permits refinement of operations and representation**: The Singleton class may be subclassed, and it's easy to configure an application with an instance of this extended class. You can configure the application with an instance of the class you need at runtime.

- **Permits a variable number of instances**: The pattern makes it easy to change your mind and allow more than one instance of the Singleton class. Only the getInstance() operation needs to change.

- **Lazy initialization**: The instance is created only when first requested, saving resources if the singleton is never used.

- **Global access point**: Provides a well-known access point for an instance that would otherwise need to be passed through many layers.

### Liabilities

- **Violates Single Responsibility Principle**: The class manages both its business logic and its own instantiation.

- **Difficult to test**: Singletons introduce global state, making unit tests dependent on each other and harder to isolate. Mocking singletons is challenging.

- **Hidden dependencies**: Classes that use singletons have hidden dependencies that aren't visible in their interfaces.

- **Concurrency issues**: In multi-threaded environments, singleton creation must be synchronized to prevent creating multiple instances.

- **Difficult to subclass**: Making a singleton subclassable requires careful design.

- **Can mask poor design**: Singletons are sometimes used to avoid proper dependency injection, hiding architectural problems.

- **Lifetime management**: It's difficult to control when the singleton is destroyed, which can cause issues with resource cleanup.

## Implementation

Consider the following implementation issues:

### 1. Basic Lazy Initialization (Not Thread-Safe)

```pseudocode
class Singleton {
    private static instance: Singleton = null

    // Private constructor prevents external instantiation
    private constructor() {
        // Initialization logic
    }

    static function getInstance(): Singleton {
        if (instance == null) {
            instance = new Singleton()
        }
        return instance
    }

    function doSomething() {
        // Business logic
    }
}

// Usage
singleton = Singleton.getInstance()
singleton.doSomething()
```

**Warning**: This implementation is not thread-safe. In a multi-threaded environment, two threads could both see `instance` as null and create two instances.

### 2. Eager Initialization (Thread-Safe)

```pseudocode
class Singleton {
    // Instance created at class loading time
    private static instance: Singleton = new Singleton()

    private constructor() {
        // Initialization logic
    }

    static function getInstance(): Singleton {
        return instance
    }
}
```

**Pros**: Simple, thread-safe without synchronization
**Cons**: Instance created even if never used; no exception handling during creation

### 3. Thread-Safe with Synchronized Method

```pseudocode
class Singleton {
    private static instance: Singleton = null

    private constructor() { }

    // Entire method is synchronized
    static synchronized function getInstance(): Singleton {
        if (instance == null) {
            instance = new Singleton()
        }
        return instance
    }
}
```

**Pros**: Thread-safe
**Cons**: Synchronization overhead on every call, even after instance exists

### 4. Double-Checked Locking

```pseudocode
class Singleton {
    // Must be volatile to prevent instruction reordering
    private static volatile instance: Singleton = null
    private static lock: Object = new Object()

    private constructor() { }

    static function getInstance(): Singleton {
        if (instance == null) {                    // First check (no lock)
            synchronized(lock) {
                if (instance == null) {            // Second check (with lock)
                    instance = new Singleton()
                }
            }
        }
        return instance
    }
}
```

**Pros**: Thread-safe with minimal synchronization overhead
**Cons**: Complex, requires volatile keyword, easy to implement incorrectly

### 5. Initialization-on-Demand Holder (Bill Pugh Singleton)

```pseudocode
class Singleton {
    private constructor() { }

    // Inner class not loaded until getInstance() called
    private static class Holder {
        static final INSTANCE: Singleton = new Singleton()
    }

    static function getInstance(): Singleton {
        return Holder.INSTANCE
    }
}
```

**Pros**: Lazy, thread-safe without synchronization, handles exceptions
**Cons**: Language-specific (relies on class loading guarantees)

### 6. Enum Singleton (Java-Specific Best Practice)

```pseudocode
enum Singleton {
    INSTANCE;

    private connection: DatabaseConnection

    // Constructor called once when enum is loaded
    Singleton() {
        connection = new DatabaseConnection()
    }

    function getConnection(): DatabaseConnection {
        return connection
    }

    function doSomething() {
        // Business logic
    }
}

// Usage
Singleton.INSTANCE.doSomething()
```

**Pros**: Thread-safe, serialization-safe, prevents reflection attacks, simple
**Cons**: Language-specific, can't extend other classes

### 7. Singleton with Configuration/Parameters

```pseudocode
class ConfigurableSingleton {
    private static instance: ConfigurableSingleton = null
    private config: Configuration

    private constructor(config: Configuration) {
        this.config = config
    }

    // Must be initialized before first use
    static function initialize(config: Configuration) {
        if (instance != null) {
            throw new IllegalStateException("Already initialized")
        }
        instance = new ConfigurableSingleton(config)
    }

    static function getInstance(): ConfigurableSingleton {
        if (instance == null) {
            throw new IllegalStateException("Not initialized. Call initialize() first")
        }
        return instance
    }

    // Alternative: Initialize with defaults if not configured
    static function getInstanceWithDefault(): ConfigurableSingleton {
        if (instance == null) {
            instance = new ConfigurableSingleton(Configuration.defaults())
        }
        return instance
    }
}
```

### 8. Resettable Singleton (For Testing)

```pseudocode
class ResettableSingleton {
    private static instance: ResettableSingleton = null

    private constructor() { }

    static function getInstance(): ResettableSingleton {
        if (instance == null) {
            instance = new ResettableSingleton()
        }
        return instance
    }

    // Only for testing - not part of public API
    @VisibleForTesting
    static function resetInstance() {
        instance = null
    }

    // Alternative: Allow injection for testing
    @VisibleForTesting
    static function setInstance(testInstance: ResettableSingleton) {
        instance = testInstance
    }
}
```

### 9. Singleton Registry (Multiple Named Singletons)

```pseudocode
class SingletonRegistry {
    private static instances: Map<String, Object> = {}
    private static lock: Object = new Object()

    static function getInstance<T>(key: String, creator: Supplier<T>): T {
        if (!instances.containsKey(key)) {
            synchronized(lock) {
                if (!instances.containsKey(key)) {
                    instances.put(key, creator())
                }
            }
        }
        return instances.get(key) as T
    }

    static function hasInstance(key: String): Boolean {
        return instances.containsKey(key)
    }
}

// Usage
logger = SingletonRegistry.getInstance("logger", () => new Logger())
cache = SingletonRegistry.getInstance("cache", () => new Cache())
```

### 10. Singleton with Dependency Injection Container

Modern applications often avoid manual singletons by using DI containers:

```pseudocode
// Framework configuration
@Configuration
class AppConfig {
    @Bean
    @Scope("singleton")  // Default scope - one instance per container
    function logger(): Logger {
        return new Logger(getLogConfig())
    }

    @Bean
    @Scope("singleton")
    function cache(): Cache {
        return new Cache(getCacheConfig())
    }
}

// Usage - framework injects the singleton
class MyService {
    private logger: Logger

    @Inject
    constructor(logger: Logger) {
        this.logger = logger
    }
}
```

## Example

Here's a complete example of the Singleton pattern applied to an application configuration manager:

```pseudocode
// ============================================================
// CONFIGURATION VALUE CLASSES
// ============================================================

class DatabaseConfig {
    host: String
    port: Integer
    database: String
    username: String
    password: String
    maxConnections: Integer
    connectionTimeout: Integer

    function getConnectionString(): String {
        return "jdbc:postgresql://" + host + ":" + port + "/" + database
    }
}

class CacheConfig {
    enabled: Boolean
    provider: String  // "redis", "memcached", "memory"
    host: String
    port: Integer
    ttlSeconds: Integer
    maxSize: Integer
}

class LoggingConfig {
    level: String  // "DEBUG", "INFO", "WARN", "ERROR"
    format: String
    outputPath: String
    maxFileSizeMb: Integer
    maxFiles: Integer
}

class FeatureFlags {
    private flags: Map<String, Boolean> = {}

    function isEnabled(feature: String): Boolean {
        return flags.getOrDefault(feature, false)
    }

    function setFlag(feature: String, enabled: Boolean) {
        flags.put(feature, enabled)
    }

    function getAllFlags(): Map<String, Boolean> {
        return Map.copyOf(flags)
    }
}

// ============================================================
// SINGLETON CONFIGURATION MANAGER
// ============================================================

class ConfigurationManager {
    // Volatile for thread safety with double-checked locking
    private static volatile instance: ConfigurationManager = null
    private static lock: Object = new Object()

    // Configuration data
    private databaseConfig: DatabaseConfig
    private cacheConfig: CacheConfig
    private loggingConfig: LoggingConfig
    private featureFlags: FeatureFlags
    private customProperties: Map<String, String>

    // Metadata
    private configSource: String
    private loadedAt: DateTime
    private environment: String

    // Listeners for configuration changes
    private changeListeners: List<ConfigChangeListener> = []

    // Private constructor
    private constructor() {
        this.customProperties = {}
        this.featureFlags = new FeatureFlags()
    }

    // Double-checked locking getInstance
    static function getInstance(): ConfigurationManager {
        if (instance == null) {
            synchronized(lock) {
                if (instance == null) {
                    instance = new ConfigurationManager()
                }
            }
        }
        return instance
    }

    // ============================================================
    // INITIALIZATION METHODS
    // ============================================================

    function loadFromFile(filePath: String) {
        validateNotLoaded()

        content = File.read(filePath)
        data = parseConfigFile(content)

        this.databaseConfig = parseDatabaseConfig(data)
        this.cacheConfig = parseCacheConfig(data)
        this.loggingConfig = parseLoggingConfig(data)
        this.featureFlags = parseFeatureFlags(data)
        this.customProperties = parseCustomProperties(data)

        this.configSource = filePath
        this.loadedAt = DateTime.now()
        this.environment = data.get("environment", "development")

        notifyListeners("loaded", null)
    }

    function loadFromEnvironment() {
        validateNotLoaded()

        this.databaseConfig = new DatabaseConfig()
        this.databaseConfig.host = Env.get("DB_HOST", "localhost")
        this.databaseConfig.port = Env.getInt("DB_PORT", 5432)
        this.databaseConfig.database = Env.get("DB_NAME", "app")
        this.databaseConfig.username = Env.get("DB_USER", "postgres")
        this.databaseConfig.password = Env.get("DB_PASSWORD", "")
        this.databaseConfig.maxConnections = Env.getInt("DB_MAX_CONN", 10)
        this.databaseConfig.connectionTimeout = Env.getInt("DB_TIMEOUT", 30000)

        this.cacheConfig = new CacheConfig()
        this.cacheConfig.enabled = Env.getBool("CACHE_ENABLED", true)
        this.cacheConfig.provider = Env.get("CACHE_PROVIDER", "memory")
        this.cacheConfig.host = Env.get("CACHE_HOST", "localhost")
        this.cacheConfig.port = Env.getInt("CACHE_PORT", 6379)
        this.cacheConfig.ttlSeconds = Env.getInt("CACHE_TTL", 3600)

        this.loggingConfig = new LoggingConfig()
        this.loggingConfig.level = Env.get("LOG_LEVEL", "INFO")
        this.loggingConfig.format = Env.get("LOG_FORMAT", "json")
        this.loggingConfig.outputPath = Env.get("LOG_PATH", "/var/log/app")

        this.configSource = "environment"
        this.loadedAt = DateTime.now()
        this.environment = Env.get("APP_ENV", "development")

        notifyListeners("loaded", null)
    }

    function loadFromRemote(configServerUrl: String, applicationName: String) {
        validateNotLoaded()

        response = Http.get(configServerUrl + "/" + applicationName)
        if (!response.isSuccess()) {
            throw new ConfigurationException("Failed to load from config server: " + response.status)
        }

        data = Json.parse(response.body)
        // ... parse similar to loadFromFile

        this.configSource = configServerUrl
        this.loadedAt = DateTime.now()

        notifyListeners("loaded", null)
    }

    private function validateNotLoaded() {
        if (this.loadedAt != null) {
            throw new ConfigurationException(
                "Configuration already loaded from: " + configSource +
                ". Call reload() to refresh."
            )
        }
    }

    // ============================================================
    // ACCESSOR METHODS
    // ============================================================

    function getDatabaseConfig(): DatabaseConfig {
        ensureLoaded()
        return databaseConfig
    }

    function getCacheConfig(): CacheConfig {
        ensureLoaded()
        return cacheConfig
    }

    function getLoggingConfig(): LoggingConfig {
        ensureLoaded()
        return loggingConfig
    }

    function getFeatureFlags(): FeatureFlags {
        ensureLoaded()
        return featureFlags
    }

    function isFeatureEnabled(feature: String): Boolean {
        ensureLoaded()
        return featureFlags.isEnabled(feature)
    }

    function getProperty(key: String): String {
        ensureLoaded()
        return customProperties.get(key)
    }

    function getProperty(key: String, defaultValue: String): String {
        ensureLoaded()
        return customProperties.getOrDefault(key, defaultValue)
    }

    function getEnvironment(): String {
        ensureLoaded()
        return environment
    }

    function isProduction(): Boolean {
        return environment == "production"
    }

    function isDevelopment(): Boolean {
        return environment == "development"
    }

    function isLoaded(): Boolean {
        return loadedAt != null
    }

    function getConfigSource(): String {
        return configSource
    }

    function getLoadedAt(): DateTime {
        return loadedAt
    }

    private function ensureLoaded() {
        if (loadedAt == null) {
            throw new ConfigurationException("Configuration not loaded. Call load*() first.")
        }
    }

    // ============================================================
    // DYNAMIC UPDATES
    // ============================================================

    function setFeatureFlag(feature: String, enabled: Boolean) {
        ensureLoaded()
        oldValue = featureFlags.isEnabled(feature)
        featureFlags.setFlag(feature, enabled)
        notifyListeners("featureFlag:" + feature, {
            "old": oldValue,
            "new": enabled
        })
    }

    function setProperty(key: String, value: String) {
        ensureLoaded()
        oldValue = customProperties.get(key)
        customProperties.put(key, value)
        notifyListeners("property:" + key, {
            "old": oldValue,
            "new": value
        })
    }

    function reload() {
        if (configSource == null) {
            throw new ConfigurationException("No config source to reload from")
        }

        // Reset state
        this.loadedAt = null

        // Reload from original source
        if (configSource == "environment") {
            loadFromEnvironment()
        } else if (configSource.startsWith("http")) {
            // Extract app name from URL for remote reload
            loadFromRemote(configSource, extractAppName())
        } else {
            loadFromFile(configSource)
        }

        notifyListeners("reloaded", null)
    }

    // ============================================================
    // CHANGE LISTENERS
    // ============================================================

    interface ConfigChangeListener {
        function onConfigChange(changeType: String, details: Map<String, Object>)
    }

    function addChangeListener(listener: ConfigChangeListener) {
        changeListeners.add(listener)
    }

    function removeChangeListener(listener: ConfigChangeListener) {
        changeListeners.remove(listener)
    }

    private function notifyListeners(changeType: String, details: Map<String, Object>) {
        for (listener in changeListeners) {
            try {
                listener.onConfigChange(changeType, details)
            } catch (Exception e) {
                // Log but don't fail - listeners shouldn't break config
                System.err.println("Config listener error: " + e.message)
            }
        }
    }

    // ============================================================
    // TESTING SUPPORT
    // ============================================================

    @VisibleForTesting
    static function resetForTesting() {
        synchronized(lock) {
            instance = null
        }
    }

    @VisibleForTesting
    static function setTestInstance(testInstance: ConfigurationManager) {
        synchronized(lock) {
            instance = testInstance
        }
    }

    // Create a test configuration with defaults
    @VisibleForTesting
    static function createTestInstance(): ConfigurationManager {
        testInstance = new ConfigurationManager()

        testInstance.databaseConfig = new DatabaseConfig()
        testInstance.databaseConfig.host = "localhost"
        testInstance.databaseConfig.port = 5432
        testInstance.databaseConfig.database = "test_db"
        testInstance.databaseConfig.username = "test"
        testInstance.databaseConfig.password = "test"
        testInstance.databaseConfig.maxConnections = 5

        testInstance.cacheConfig = new CacheConfig()
        testInstance.cacheConfig.enabled = false
        testInstance.cacheConfig.provider = "memory"

        testInstance.loggingConfig = new LoggingConfig()
        testInstance.loggingConfig.level = "DEBUG"
        testInstance.loggingConfig.format = "text"

        testInstance.environment = "test"
        testInstance.configSource = "test"
        testInstance.loadedAt = DateTime.now()

        return testInstance
    }

    // ============================================================
    // HELPER METHODS
    // ============================================================

    private function parseConfigFile(content: String): Map<String, Object> {
        // Detect format and parse
        if (content.trim().startsWith("{")) {
            return Json.parse(content)
        } else if (content.contains(":") && !content.contains("=")) {
            return Yaml.parse(content)
        } else {
            return Properties.parse(content)
        }
    }

    private function parseDatabaseConfig(data: Map<String, Object>): DatabaseConfig {
        dbData = data.get("database", {})
        config = new DatabaseConfig()
        config.host = dbData.get("host", "localhost")
        config.port = dbData.get("port", 5432)
        config.database = dbData.get("name", "app")
        config.username = dbData.get("username", "postgres")
        config.password = dbData.get("password", "")
        config.maxConnections = dbData.get("maxConnections", 10)
        config.connectionTimeout = dbData.get("connectionTimeout", 30000)
        return config
    }

    // ... similar parse methods for other config sections
}

// ============================================================
// USAGE EXAMPLES
// ============================================================

function main() {
    // Initialize configuration at application startup
    config = ConfigurationManager.getInstance()

    // Load from appropriate source based on environment
    if (Env.has("CONFIG_SERVER_URL")) {
        config.loadFromRemote(
            Env.get("CONFIG_SERVER_URL"),
            Env.get("APP_NAME", "myapp")
        )
    } else if (File.exists("config.yaml")) {
        config.loadFromFile("config.yaml")
    } else {
        config.loadFromEnvironment()
    }

    // Add listener for config changes
    config.addChangeListener(new ConfigChangeListener() {
        function onConfigChange(changeType: String, details: Map<String, Object>) {
            print("Configuration changed: " + changeType)
        }
    })

    // Use configuration anywhere in the application
    startApplication()
}

function startApplication() {
    config = ConfigurationManager.getInstance()

    // Database connection
    dbConfig = config.getDatabaseConfig()
    connectionString = dbConfig.getConnectionString()
    pool = DatabasePool.create(connectionString, dbConfig.maxConnections)

    // Cache setup
    cacheConfig = config.getCacheConfig()
    if (cacheConfig.enabled) {
        cache = CacheFactory.create(cacheConfig)
    }

    // Feature flag check
    if (config.isFeatureEnabled("new-dashboard")) {
        enableNewDashboard()
    }

    // Environment-specific behavior
    if (config.isDevelopment()) {
        enableDebugMode()
    }
}

class UserService {
    function createUser(userData: UserData): User {
        config = ConfigurationManager.getInstance()

        // Check feature flag
        if (config.isFeatureEnabled("email-verification")) {
            sendVerificationEmail(userData.email)
        }

        // Use custom property
        defaultRole = config.getProperty("user.defaultRole", "member")
        userData.role = defaultRole

        return userRepository.save(userData)
    }
}

// ============================================================
// TESTING WITH SINGLETON
// ============================================================

class UserServiceTest {
    @BeforeEach
    function setup() {
        // Reset singleton and install test configuration
        ConfigurationManager.resetForTesting()
        testConfig = ConfigurationManager.createTestInstance()
        ConfigurationManager.setTestInstance(testConfig)
    }

    @AfterEach
    function teardown() {
        ConfigurationManager.resetForTesting()
    }

    @Test
    function testCreateUserWithEmailVerification() {
        // Arrange
        config = ConfigurationManager.getInstance()
        config.setFeatureFlag("email-verification", true)

        service = new UserService()
        userData = new UserData("test@example.com", "Test User")

        // Act
        user = service.createUser(userData)

        // Assert
        assertNotNull(user)
        assertTrue(emailWasSent("test@example.com"))
    }

    @Test
    function testCreateUserWithoutEmailVerification() {
        // Arrange
        config = ConfigurationManager.getInstance()
        config.setFeatureFlag("email-verification", false)

        service = new UserService()
        userData = new UserData("test@example.com", "Test User")

        // Act
        user = service.createUser(userData)

        // Assert
        assertNotNull(user)
        assertFalse(emailWasSent("test@example.com"))
    }
}
```

## Known Uses

- **Java Runtime**: `Runtime.getRuntime()` returns the singleton Runtime instance.

- **Spring Framework**: Beans are singletons by default within the application context.

- **Logging Frameworks**: SLF4J's `LoggerFactory`, Log4j's `LogManager`.

- **Database Connections**: Connection pool managers are typically singletons.

- **Window Managers**: Desktop environments use singleton window managers.

- **File Systems**: Many operating systems use singleton file system objects.

- **Print Spoolers**: One spooler manages all print jobs.

- **Hardware Access**: Drivers for unique hardware resources (GPU, display) are often singletons.

- **JavaScript Modules**: ES6 modules are singleton-like by default.

- **Redux Store**: The Redux pattern uses a single store for application state.

## Related Patterns

- **Abstract Factory, Builder, Prototype**: Can all be implemented as Singletons. Factories often work well as singletons because the application usually only needs one factory.

- **Facade**: Often implemented as a Singleton because only one Facade object is required.

- **State**: State objects are often Singletons.

- **Flyweight**: Flyweight factory is often a Singleton.

- **Monostate**: An alternative to Singleton that makes all members static, achieving similar goals through different means.

## When NOT to Use

The Singleton pattern is one of the most overused and criticized patterns. Consider these reasons to avoid it:

- **Unit testing**: Singletons make unit testing difficult because they maintain state between tests and are hard to mock.

- **Hidden dependencies**: Classes using singletons have hidden dependencies not visible in their APIs.

- **Concurrency issues**: Global state complicates concurrent programming.

- **Tight coupling**: Code becomes tightly coupled to the singleton class.

- **SOLID violations**: Violates Single Responsibility (class manages its own lifecycle) and Dependency Inversion (depends on concrete implementation).

- **Difficult to maintain**: Changes to the singleton affect all consumers.

**Simpler alternatives**:

- **Dependency Injection**: Pass dependencies explicitly
  ```pseudocode
  class UserService {
      constructor(config: Configuration, logger: Logger) {
          this.config = config
          this.logger = logger
      }
  }
  ```

- **DI Container**: Let a framework manage singleton lifecycle
  ```pseudocode
  @Injectable({ providedIn: 'root' })  // Angular singleton service
  class ConfigService { }
  ```

- **Module-level instance**: Create one instance at module level
  ```pseudocode
  // config.js
  export const config = new Configuration()
  export const logger = new Logger()
  ```

- **Pass as parameter**: Thread configuration through function calls
  ```pseudocode
  function processOrder(order: Order, config: Configuration): Result {
      // Use passed config instead of singleton
  }
  ```

**Signs you've misused Singleton**:
- You have multiple singletons that depend on each other
- Tests require reset methods or special setup
- You're using Singleton for things that aren't truly global
- Different parts of the app need different configurations
- You find yourself passing the singleton instance as a parameter anyway
- You have singletons in library code (libraries shouldn't force global state)

**When Singleton IS appropriate**:
- Hardware access (truly one resource)
- Logger (truly global, stateless writes)
- Configuration loaded once at startup (read-only after init)
- Caches that are truly application-wide
- Thread pools with application-wide scope

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
