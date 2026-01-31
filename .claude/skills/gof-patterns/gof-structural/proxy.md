# Proxy

## Intent

Provide a surrogate or placeholder for another object to control access to it. A proxy controls access to the original object, allowing you to perform something either before or after the request gets through to the original object.

## Also Known As

- Surrogate
- Placeholder

## Motivation

One reason for controlling access to an object is to defer the full cost of its creation and initialization until we actually need to use it. Consider a document editor that can embed graphical objects in a document. Some graphical objects, like large raster images, can be expensive to create. But opening a document should be fast, so we should avoid creating all the expensive objects at once when the document is opened.

This is not necessary anyway, because not all of these objects will be visible in the document at the same time. So it makes sense to create expensive objects on demand. But what do we put in the document in place of the image? And how can we hide the fact that the image is created on demand so that we do not complicate the editor's implementation?

The solution is to use another object, an image proxy, that acts as a stand-in for the real image. The proxy acts just like the image and takes care of instantiating it when it is required. The proxy creates the real image only when the document editor asks it to display itself by invoking its `draw()` operation. The proxy forwards subsequent requests directly to the image. It must therefore keep a reference to the image after creating it.

The image proxy is a **virtual proxy** - it creates expensive objects on demand. There are other kinds of proxies: **protection proxies** control access to the original object based on access rights, **remote proxies** represent objects in different address spaces, and **smart references** perform additional actions when an object is accessed.

## Applicability

Proxy is applicable whenever there is a need for a more versatile or sophisticated reference to an object than a simple pointer. Here are several common situations in which the Proxy pattern is applicable:

- **Remote Proxy**: Provides a local representative for an object in a different address space. Also known as Ambassador.

- **Virtual Proxy**: Creates expensive objects on demand. The image proxy described in the motivation is an example.

- **Protection Proxy**: Controls access to the original object. Protection proxies are useful when objects should have different access rights.

- **Smart Reference**: A replacement for a bare pointer that performs additional actions when an object is accessed. Typical uses include:
  - Counting the number of references to the real object so that it can be freed automatically when there are no more references (smart pointers).
  - Loading a persistent object into memory when it is first referenced.
  - Checking that the real object is locked before it is accessed to ensure no other object can change it.

- **Caching Proxy**: Stores results of expensive operations and returns cached results when the same inputs occur again.

- **Logging Proxy**: Keeps a log of requests to the service object.

- **Synchronization Proxy**: Controls access to an object in a multithreaded environment.

## Structure

```
┌─────────────────────────┐
│        Client           │
└─────────────────────────┘
            │
            │ uses
            ▼
┌─────────────────────────┐
│    <<interface>>        │
│       Subject           │
├─────────────────────────┤
│ + request()             │
└─────────────────────────┘
            △
            │ implements
    ┌───────┴───────┐
    │               │
┌───────────────┐  ┌───────────────────────┐
│  RealSubject  │  │        Proxy          │
├───────────────┤  ├───────────────────────┤
│ + request()   │  │ - realSubject: Subject│
│   // actual   │  ├───────────────────────┤
│   // work     │  │ + request()           │
└───────────────┘  │   // control access   │
        △          │   // then delegate    │
        │          │   realSubject.request()│
        │          └───────────────────────┘
        │                    │
        └────────────────────┘
              references
```

### Proxy Types Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Proxy Types                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐     │
│  │  Virtual Proxy  │    │ Protection Proxy│    │  Remote Proxy   │     │
│  ├─────────────────┤    ├─────────────────┤    ├─────────────────┤     │
│  │ Lazy loading    │    │ Access control  │    │ Network calls   │     │
│  │ On-demand       │    │ Permissions     │    │ Serialization   │     │
│  │ creation        │    │ Authentication  │    │ Marshalling     │     │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘     │
│                                                                          │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐     │
│  │  Caching Proxy  │    │  Logging Proxy  │    │   Smart Proxy   │     │
│  ├─────────────────┤    ├─────────────────┤    ├─────────────────┤     │
│  │ Result caching  │    │ Request logging │    │ Reference count │     │
│  │ TTL management  │    │ Audit trails    │    │ Lock management │     │
│  │ Invalidation    │    │ Metrics         │    │ Copy-on-write   │     │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Subject**: Defines the common interface for RealSubject and Proxy so that a Proxy can be used anywhere a RealSubject is expected.

- **RealSubject**: Defines the real object that the proxy represents. Contains the actual business logic.

- **Proxy**: Maintains a reference that lets the proxy access the real subject. Provides an interface identical to Subject's so that a proxy can be substituted for the real subject. Controls access to the real subject and may be responsible for creating and deleting it. Depending on the proxy type, it may:
  - **Remote Proxy**: Encode requests and send them to the real subject in a different address space.
  - **Virtual Proxy**: Cache information about the real subject to postpone accessing it.
  - **Protection Proxy**: Check that the caller has the access permissions required to perform a request.

## Collaborations

1. The client interacts with the Proxy through the Subject interface.

2. The Proxy receives requests from the client and performs its specific duties (access control, lazy loading, caching, logging, etc.).

3. If appropriate, the Proxy forwards the request to the RealSubject.

4. The Proxy may perform additional operations before or after forwarding the request.

5. The client is unaware of whether it is working with a proxy or the real subject.

## Consequences

### Benefits

- **Controlled access**: The proxy can control access to the real subject, including who can access it and when.

- **Lazy initialization**: Virtual proxies can delay expensive object creation until it is actually needed.

- **Remote transparency**: Remote proxies hide the complexity of network communication from clients.

- **Logging and auditing**: Logging proxies can track all access to the real subject without modifying it.

- **Caching**: Caching proxies can store results and avoid repeated expensive operations.

- **Security**: Protection proxies can implement access control without modifying the real subject.

- **Smart resource management**: Smart proxies can manage object lifecycle, reference counting, and resource cleanup.

- **Open/Closed Principle**: You can introduce new proxies without changing the service or clients.

### Liabilities

- **Increased complexity**: The pattern introduces additional classes and indirection.

- **Response delay**: Some proxies add processing overhead. Remote proxies add network latency.

- **Code duplication**: Proxies must implement the same interface as the real subject, which can lead to boilerplate code.

- **Maintenance burden**: Changes to the Subject interface require changes to all proxies.

- **Hidden behavior**: Proxies can make the system harder to understand because they hide behavior from clients.

- **Memory overhead**: Proxies consume memory even before the real subject is created.

## Implementation

### Implementation Considerations

1. **Overloading member access operators**: In languages like C++, you can overload the `->` and `*` operators to make proxy usage transparent. In other languages, you may need to use reflection or code generation.

2. **Proxy does not always need to know the real subject's concrete type**: Sometimes the proxy can work with the Subject interface alone, especially for protection and logging proxies.

3. **Copy-on-write**: A virtual proxy can implement copy-on-write optimization. Multiple clients can share the same real subject until one of them modifies it.

4. **Proxy chains**: Multiple proxies can be chained, each adding its own behavior (similar to Decorator).

### Virtual Proxy Implementation

```
// Subject interface
interface Image
    method display()
    method getWidth(): Integer
    method getHeight(): Integer
    method getFilename(): String
end interface

// Real Subject - expensive to create
class HighResolutionImage implements Image
    private filename: String
    private width: Integer
    private height: Integer
    private imageData: Bytes

    constructor(filename: String)
        this.filename = filename
        loadFromDisk()  // Expensive operation
    end constructor

    private method loadFromDisk()
        print "Loading high-resolution image: " + filename
        // Simulate expensive loading
        sleep(2000)  // 2 seconds to load

        // Read image file
        fileData = readFile(filename)
        imageData = decodeImage(fileData)
        width = imageData.width
        height = imageData.height

        print "Image loaded: " + width + "x" + height
    end method

    method display()
        print "Displaying high-resolution image: " + filename
        // Render imageData to screen
    end method

    method getWidth(): Integer
        return width
    end method

    method getHeight(): Integer
        return height
    end method

    method getFilename(): String
        return filename
    end method
end class

// Virtual Proxy - lazy loading
class ImageProxy implements Image
    private filename: String
    private realImage: HighResolutionImage
    private cachedWidth: Integer
    private cachedHeight: Integer

    constructor(filename: String)
        this.filename = filename
        this.realImage = null

        // Load only metadata (cheap operation)
        loadMetadata()
    end constructor

    private method loadMetadata()
        // Read just the image header, not the full data
        header = readImageHeader(filename)
        cachedWidth = header.width
        cachedHeight = header.height
    end method

    private method ensureLoaded()
        if realImage == null
            realImage = new HighResolutionImage(filename)
        end if
    end method

    method display()
        // Only load when actually needed
        ensureLoaded()
        realImage.display()
    end method

    method getWidth(): Integer
        // Can return cached value without loading
        return cachedWidth
    end method

    method getHeight(): Integer
        // Can return cached value without loading
        return cachedHeight
    end method

    method getFilename(): String
        return filename
    end method
end class

// Usage
// Opening a document with 100 images is fast because we only create proxies
document = new Document()

for i = 0 to 100
    // Creating proxy is cheap - no image loading yet
    image = new ImageProxy("image_" + i + ".jpg")
    document.addImage(image)
    print "Added image: " + image.getFilename() + " (" + image.getWidth() + "x" + image.getHeight() + ")"
end for

// Only when user scrolls to see an image is it actually loaded
visibleImage = document.getImageAt(50)
visibleImage.display()  // NOW it loads from disk
```

### Protection Proxy Implementation

```
// Subject interface
interface BankAccount
    method getBalance(): Decimal
    method deposit(amount: Decimal)
    method withdraw(amount: Decimal): Boolean
    method getTransactionHistory(): List<Transaction>
    method setInterestRate(rate: Decimal)  // Admin only
    method closeAccount()  // Admin only
end interface

// Real Subject
class RealBankAccount implements BankAccount
    private accountNumber: String
    private balance: Decimal
    private interestRate: Decimal
    private transactions: List<Transaction>
    private closed: Boolean

    constructor(accountNumber: String, initialBalance: Decimal)
        this.accountNumber = accountNumber
        this.balance = initialBalance
        this.interestRate = 0.02
        this.transactions = new ArrayList()
        this.closed = false
    end constructor

    method getBalance(): Decimal
        return balance
    end method

    method deposit(amount: Decimal)
        if closed
            throw new AccountClosedException()
        end if

        balance = balance + amount
        transactions.add(new Transaction(TransactionType.DEPOSIT, amount, DateTime.now()))
    end method

    method withdraw(amount: Decimal): Boolean
        if closed
            throw new AccountClosedException()
        end if

        if amount > balance
            return false
        end if

        balance = balance - amount
        transactions.add(new Transaction(TransactionType.WITHDRAWAL, amount, DateTime.now()))
        return true
    end method

    method getTransactionHistory(): List<Transaction>
        return Collections.unmodifiableList(transactions)
    end method

    method setInterestRate(rate: Decimal)
        this.interestRate = rate
    end method

    method closeAccount()
        this.closed = true
    end method
end class

// Protection Proxy
class SecureBankAccountProxy implements BankAccount
    private realAccount: RealBankAccount
    private currentUser: User
    private auditLog: AuditLog

    constructor(realAccount: RealBankAccount, currentUser: User, auditLog: AuditLog)
        this.realAccount = realAccount
        this.currentUser = currentUser
        this.auditLog = auditLog
    end constructor

    private method checkPermission(operation: String)
        auditLog.log(currentUser, operation, realAccount.accountNumber)

        if not hasPermission(operation)
            throw new UnauthorizedAccessException(
                "User " + currentUser.name + " does not have permission for: " + operation
            )
        end if
    end method

    private method hasPermission(operation: String): Boolean
        // Check if user owns the account or has admin rights
        switch operation
            case "VIEW_BALANCE":
            case "DEPOSIT":
            case "WITHDRAW":
            case "VIEW_HISTORY":
                return currentUser.ownsAccount(realAccount.accountNumber) or
                       currentUser.hasRole(Role.TELLER) or
                       currentUser.hasRole(Role.ADMIN)

            case "SET_INTEREST_RATE":
            case "CLOSE_ACCOUNT":
                return currentUser.hasRole(Role.ADMIN)

            default:
                return false
        end switch
    end method

    method getBalance(): Decimal
        checkPermission("VIEW_BALANCE")
        return realAccount.getBalance()
    end method

    method deposit(amount: Decimal)
        checkPermission("DEPOSIT")

        // Additional validation
        if amount <= 0
            throw new InvalidAmountException("Deposit amount must be positive")
        end if

        if amount > 10000 and not currentUser.hasRole(Role.ADMIN)
            throw new LargeTransactionException("Deposits over $10,000 require admin approval")
        end if

        realAccount.deposit(amount)
    end method

    method withdraw(amount: Decimal): Boolean
        checkPermission("WITHDRAW")

        // Additional validation
        if amount <= 0
            throw new InvalidAmountException("Withdrawal amount must be positive")
        end if

        // Daily limit for non-admins
        if not currentUser.hasRole(Role.ADMIN)
            todayWithdrawals = calculateTodayWithdrawals()
            if todayWithdrawals + amount > 5000
                throw new DailyLimitExceededException("Daily withdrawal limit is $5,000")
            end if
        end if

        return realAccount.withdraw(amount)
    end method

    method getTransactionHistory(): List<Transaction>
        checkPermission("VIEW_HISTORY")

        // Non-owners can only see last 30 days
        if not currentUser.ownsAccount(realAccount.accountNumber)
            fullHistory = realAccount.getTransactionHistory()
            thirtyDaysAgo = DateTime.now().minusDays(30)
            return fullHistory.filter(t -> t.date.isAfter(thirtyDaysAgo))
        end if

        return realAccount.getTransactionHistory()
    end method

    method setInterestRate(rate: Decimal)
        checkPermission("SET_INTEREST_RATE")

        // Validate rate
        if rate < 0 or rate > 0.25
            throw new InvalidRateException("Interest rate must be between 0% and 25%")
        end if

        realAccount.setInterestRate(rate)
    end method

    method closeAccount()
        checkPermission("CLOSE_ACCOUNT")

        // Ensure balance is zero
        if realAccount.getBalance() != 0
            throw new NonZeroBalanceException("Cannot close account with non-zero balance")
        end if

        realAccount.closeAccount()
    end method

    private method calculateTodayWithdrawals(): Decimal
        history = realAccount.getTransactionHistory()
        today = DateTime.now().toDate()

        total = 0
        for each transaction in history
            if transaction.date.toDate() == today and transaction.type == TransactionType.WITHDRAWAL
                total = total + transaction.amount
            end if
        end for

        return total
    end method
end class

// Usage
realAccount = new RealBankAccount("12345", 1000.00)

// Customer accessing their own account
customer = new User("john_doe", [Role.CUSTOMER], ["12345"])
customerProxy = new SecureBankAccountProxy(realAccount, customer, auditLog)

customerProxy.getBalance()  // OK
customerProxy.withdraw(100)  // OK
customerProxy.setInterestRate(0.05)  // Throws UnauthorizedAccessException

// Admin accessing the account
admin = new User("admin", [Role.ADMIN], [])
adminProxy = new SecureBankAccountProxy(realAccount, admin, auditLog)

adminProxy.getBalance()  // OK
adminProxy.setInterestRate(0.05)  // OK - admin has permission
adminProxy.closeAccount()  // Throws NonZeroBalanceException (business rule)
```

### Caching Proxy Implementation

```
// Subject interface
interface WeatherService
    method getCurrentWeather(city: String): WeatherData
    method getForecast(city: String, days: Integer): List<WeatherData>
    method getHistoricalData(city: String, date: Date): WeatherData
end interface

// Real Subject - makes actual API calls
class RealWeatherService implements WeatherService
    private apiKey: String
    private apiEndpoint: String

    constructor(apiKey: String)
        this.apiKey = apiKey
        this.apiEndpoint = "https://api.weather.com/v1"
    end constructor

    method getCurrentWeather(city: String): WeatherData
        print "Fetching current weather for " + city + " from API..."
        response = httpGet(apiEndpoint + "/current?city=" + city + "&key=" + apiKey)
        return parseWeatherData(response)
    end method

    method getForecast(city: String, days: Integer): List<WeatherData>
        print "Fetching " + days + "-day forecast for " + city + " from API..."
        response = httpGet(apiEndpoint + "/forecast?city=" + city + "&days=" + days + "&key=" + apiKey)
        return parseForecastData(response)
    end method

    method getHistoricalData(city: String, date: Date): WeatherData
        print "Fetching historical weather for " + city + " on " + date + " from API..."
        response = httpGet(apiEndpoint + "/history?city=" + city + "&date=" + date + "&key=" + apiKey)
        return parseWeatherData(response)
    end method
end class

// Cache entry with TTL
class CacheEntry<T>
    public value: T
    public expiresAt: DateTime
    public hits: Integer

    constructor(value: T, ttl: Duration)
        this.value = value
        this.expiresAt = DateTime.now().plus(ttl)
        this.hits = 0
    end constructor

    method isExpired(): Boolean
        return DateTime.now().isAfter(expiresAt)
    end method

    method hit(): T
        hits = hits + 1
        return value
    end method
end class

// Caching Proxy
class CachingWeatherProxy implements WeatherService
    private realService: WeatherService
    private cache: Map<String, CacheEntry>
    private currentWeatherTTL: Duration
    private forecastTTL: Duration
    private historicalTTL: Duration
    private maxCacheSize: Integer
    private stats: CacheStats

    constructor(realService: WeatherService, config: CacheConfig)
        this.realService = realService
        this.cache = new LinkedHashMap()  // Maintains insertion order for LRU
        this.currentWeatherTTL = config.currentWeatherTTL or Duration.minutes(10)
        this.forecastTTL = config.forecastTTL or Duration.hours(1)
        this.historicalTTL = config.historicalTTL or Duration.days(30)  // Historical doesn't change
        this.maxCacheSize = config.maxCacheSize or 1000
        this.stats = new CacheStats()
    end constructor

    method getCurrentWeather(city: String): WeatherData
        cacheKey = "current:" + city.toLowerCase()

        // Try cache first
        cached = getFromCache(cacheKey)
        if cached != null
            stats.recordHit()
            return cached
        end if

        // Cache miss - fetch from real service
        stats.recordMiss()
        data = realService.getCurrentWeather(city)

        // Store in cache
        putInCache(cacheKey, data, currentWeatherTTL)

        return data
    end method

    method getForecast(city: String, days: Integer): List<WeatherData>
        cacheKey = "forecast:" + city.toLowerCase() + ":" + days

        cached = getFromCache(cacheKey)
        if cached != null
            stats.recordHit()
            return cached
        end if

        stats.recordMiss()
        data = realService.getForecast(city, days)
        putInCache(cacheKey, data, forecastTTL)

        return data
    end method

    method getHistoricalData(city: String, date: Date): WeatherData
        cacheKey = "historical:" + city.toLowerCase() + ":" + date.toString()

        cached = getFromCache(cacheKey)
        if cached != null
            stats.recordHit()
            return cached
        end if

        stats.recordMiss()
        data = realService.getHistoricalData(city, date)

        // Historical data can be cached for a long time
        putInCache(cacheKey, data, historicalTTL)

        return data
    end method

    private method getFromCache(key: String): Object
        entry = cache.get(key)

        if entry == null
            return null
        end if

        if entry.isExpired()
            cache.remove(key)
            stats.recordExpiration()
            return null
        end if

        return entry.hit()
    end method

    private method putInCache(key: String, value: Object, ttl: Duration)
        // Evict if cache is full (LRU - remove oldest entries)
        while cache.size() >= maxCacheSize
            oldestKey = cache.keySet().iterator().next()
            cache.remove(oldestKey)
            stats.recordEviction()
        end while

        cache.put(key, new CacheEntry(value, ttl))
    end method

    // Cache management methods
    method invalidate(pattern: String)
        keysToRemove = new ArrayList()
        for each key in cache.keySet()
            if key.matches(pattern)
                keysToRemove.add(key)
            end if
        end for

        for each key in keysToRemove
            cache.remove(key)
        end for
    end method

    method invalidateCity(city: String)
        invalidate(".*:" + city.toLowerCase() + ".*")
    end method

    method clearCache()
        cache.clear()
    end method

    method getCacheStats(): CacheStats
        return stats.copy()
    end method

    method getCacheSize(): Integer
        return cache.size()
    end method

    method warmCache(cities: List<String>)
        // Pre-populate cache with common queries
        for each city in cities
            getCurrentWeather(city)
            getForecast(city, 7)
        end for
    end method
end class

// Cache statistics
class CacheStats
    private hits: Integer
    private misses: Integer
    private evictions: Integer
    private expirations: Integer

    constructor()
        this.hits = 0
        this.misses = 0
        this.evictions = 0
        this.expirations = 0
    end constructor

    method recordHit()
        hits = hits + 1
    end method

    method recordMiss()
        misses = misses + 1
    end method

    method recordEviction()
        evictions = evictions + 1
    end method

    method recordExpiration()
        expirations = expirations + 1
    end method

    method getHitRate(): Float
        total = hits + misses
        if total == 0
            return 0
        end if
        return hits / total
    end method

    method toString(): String
        return "CacheStats{hits=" + hits + ", misses=" + misses +
               ", hitRate=" + (getHitRate() * 100) + "%, evictions=" + evictions +
               ", expirations=" + expirations + "}"
    end method
end class

// Usage
realService = new RealWeatherService("api-key-123")
config = new CacheConfig(
    currentWeatherTTL: Duration.minutes(5),
    forecastTTL: Duration.minutes(30),
    maxCacheSize: 500
)
weatherService = new CachingWeatherProxy(realService, config)

// Warm cache with popular cities
weatherService.warmCache(["New York", "London", "Tokyo", "Paris"])

// First call - cache miss, fetches from API
weather1 = weatherService.getCurrentWeather("New York")
// "Fetching current weather for New York from API..."

// Second call - cache hit, no API call
weather2 = weatherService.getCurrentWeather("New York")
// No output - returned from cache

// Check stats
print weatherService.getCacheStats()
// "CacheStats{hits=1, misses=5, hitRate=16.67%, evictions=0, expirations=0}"
```

## Example

### Remote Proxy - Database Repository

A real-world example of a remote proxy that provides local access to a remote database service.

```
// Subject interface
interface UserRepository
    method findById(id: String): User
    method findByEmail(email: String): User
    method findAll(page: Integer, size: Integer): Page<User>
    method save(user: User): User
    method delete(id: String): Boolean
    method count(): Long
end interface

// Real Subject - direct database access (would be on server)
class DatabaseUserRepository implements UserRepository
    private database: Database
    private tableName: String

    constructor(database: Database)
        this.database = database
        this.tableName = "users"
    end constructor

    method findById(id: String): User
        row = database.queryOne(
            "SELECT * FROM " + tableName + " WHERE id = ?",
            [id]
        )

        if row == null
            return null
        end if

        return mapRowToUser(row)
    end method

    method findByEmail(email: String): User
        row = database.queryOne(
            "SELECT * FROM " + tableName + " WHERE email = ?",
            [email]
        )

        if row == null
            return null
        end if

        return mapRowToUser(row)
    end method

    method findAll(page: Integer, size: Integer): Page<User>
        offset = page * size
        rows = database.query(
            "SELECT * FROM " + tableName + " ORDER BY created_at DESC LIMIT ? OFFSET ?",
            [size, offset]
        )

        users = rows.map(row -> mapRowToUser(row))
        total = count()

        return new Page(users, page, size, total)
    end method

    method save(user: User): User
        if user.id == null
            // Insert
            user.id = generateId()
            user.createdAt = DateTime.now()
            database.execute(
                "INSERT INTO " + tableName + " (id, name, email, created_at) VALUES (?, ?, ?, ?)",
                [user.id, user.name, user.email, user.createdAt]
            )
        else
            // Update
            user.updatedAt = DateTime.now()
            database.execute(
                "UPDATE " + tableName + " SET name = ?, email = ?, updated_at = ? WHERE id = ?",
                [user.name, user.email, user.updatedAt, user.id]
            )
        end if

        return user
    end method

    method delete(id: String): Boolean
        rowsAffected = database.execute(
            "DELETE FROM " + tableName + " WHERE id = ?",
            [id]
        )
        return rowsAffected > 0
    end method

    method count(): Long
        result = database.queryOne("SELECT COUNT(*) as count FROM " + tableName, [])
        return result.count
    end method

    private method mapRowToUser(row: Row): User
        return new User(
            id: row.getString("id"),
            name: row.getString("name"),
            email: row.getString("email"),
            createdAt: row.getDateTime("created_at"),
            updatedAt: row.getDateTime("updated_at")
        )
    end method
end class

// Remote Proxy - client-side, communicates over network
class RemoteUserRepositoryProxy implements UserRepository
    private endpoint: String
    private httpClient: HttpClient
    private serializer: JsonSerializer
    private retryConfig: RetryConfig
    private circuitBreaker: CircuitBreaker

    constructor(endpoint: String, config: ProxyConfig)
        this.endpoint = endpoint
        this.httpClient = new HttpClient(config.timeout)
        this.serializer = new JsonSerializer()
        this.retryConfig = config.retryConfig or RetryConfig.default()
        this.circuitBreaker = new CircuitBreaker(config.circuitBreakerConfig)
    end constructor

    method findById(id: String): User
        return executeWithResilience(() -> {
            response = httpClient.get(endpoint + "/users/" + id)

            if response.status == 404
                return null
            end if

            checkResponse(response)
            return serializer.deserialize(response.body, User.class)
        })
    end method

    method findByEmail(email: String): User
        return executeWithResilience(() -> {
            response = httpClient.get(endpoint + "/users?email=" + urlEncode(email))

            if response.status == 404
                return null
            end if

            checkResponse(response)
            return serializer.deserialize(response.body, User.class)
        })
    end method

    method findAll(page: Integer, size: Integer): Page<User>
        return executeWithResilience(() -> {
            response = httpClient.get(endpoint + "/users?page=" + page + "&size=" + size)
            checkResponse(response)
            return serializer.deserialize(response.body, Page<User>.class)
        })
    end method

    method save(user: User): User
        return executeWithResilience(() -> {
            json = serializer.serialize(user)

            if user.id == null
                response = httpClient.post(endpoint + "/users", json)
            else
                response = httpClient.put(endpoint + "/users/" + user.id, json)
            end if

            checkResponse(response)
            return serializer.deserialize(response.body, User.class)
        })
    end method

    method delete(id: String): Boolean
        return executeWithResilience(() -> {
            response = httpClient.delete(endpoint + "/users/" + id)

            if response.status == 404
                return false
            end if

            checkResponse(response)
            return true
        })
    end method

    method count(): Long
        return executeWithResilience(() -> {
            response = httpClient.get(endpoint + "/users/count")
            checkResponse(response)
            result = serializer.deserialize(response.body, CountResult.class)
            return result.count
        })
    end method

    private method executeWithResilience<T>(operation: Supplier<T>): T
        // Check circuit breaker
        if circuitBreaker.isOpen()
            throw new ServiceUnavailableException("Service temporarily unavailable")
        end if

        // Retry with exponential backoff
        lastException = null
        for attempt = 1 to retryConfig.maxAttempts
            try
                result = operation.get()
                circuitBreaker.recordSuccess()
                return result
            catch NetworkException as e
                lastException = e
                circuitBreaker.recordFailure()

                if attempt < retryConfig.maxAttempts
                    delay = retryConfig.baseDelay * Math.pow(2, attempt - 1)
                    sleep(delay)
                end if
            catch Exception as e
                // Non-retryable exception
                circuitBreaker.recordFailure()
                throw e
            end try
        end for

        throw new RemoteServiceException("Failed after " + retryConfig.maxAttempts + " attempts", lastException)
    end method

    private method checkResponse(response: HttpResponse)
        if response.status >= 400
            error = serializer.deserialize(response.body, ErrorResponse.class)

            switch response.status
                case 400:
                    throw new ValidationException(error.message)
                case 401:
                    throw new AuthenticationException(error.message)
                case 403:
                    throw new AuthorizationException(error.message)
                case 404:
                    throw new NotFoundException(error.message)
                case 409:
                    throw new ConflictException(error.message)
                case 500:
                    throw new ServerException(error.message)
                default:
                    throw new RemoteServiceException("HTTP " + response.status + ": " + error.message)
            end switch
        end if
    end method
end class

// Circuit Breaker for resilience
class CircuitBreaker
    private state: CircuitState
    private failureCount: Integer
    private lastFailureTime: DateTime
    private config: CircuitBreakerConfig

    enum CircuitState { CLOSED, OPEN, HALF_OPEN }

    constructor(config: CircuitBreakerConfig)
        this.config = config
        this.state = CircuitState.CLOSED
        this.failureCount = 0
    end constructor

    method isOpen(): Boolean
        if state == CircuitState.OPEN
            // Check if enough time has passed to try again
            if DateTime.now().isAfter(lastFailureTime.plus(config.openDuration))
                state = CircuitState.HALF_OPEN
                return false
            end if
            return true
        end if
        return false
    end method

    method recordSuccess()
        failureCount = 0
        state = CircuitState.CLOSED
    end method

    method recordFailure()
        failureCount = failureCount + 1
        lastFailureTime = DateTime.now()

        if failureCount >= config.failureThreshold
            state = CircuitState.OPEN
        end if
    end method
end class

// Combined Proxy - caching + remote
class CachingRemoteUserRepositoryProxy implements UserRepository
    private remoteProxy: RemoteUserRepositoryProxy
    private cache: Cache<String, User>
    private listCache: Cache<String, Page<User>>

    constructor(endpoint: String, cacheConfig: CacheConfig)
        this.remoteProxy = new RemoteUserRepositoryProxy(endpoint)
        this.cache = new Cache(cacheConfig.userTTL)
        this.listCache = new Cache(cacheConfig.listTTL)
    end constructor

    method findById(id: String): User
        cacheKey = "user:" + id

        // Try cache
        cached = cache.get(cacheKey)
        if cached != null
            return cached
        end if

        // Fetch and cache
        user = remoteProxy.findById(id)
        if user != null
            cache.put(cacheKey, user)
        end if

        return user
    end method

    method save(user: User): User
        savedUser = remoteProxy.save(user)

        // Update cache
        cache.put("user:" + savedUser.id, savedUser)

        // Invalidate list cache
        listCache.clear()

        return savedUser
    end method

    method delete(id: String): Boolean
        result = remoteProxy.delete(id)

        if result
            // Invalidate caches
            cache.remove("user:" + id)
            listCache.clear()
        end if

        return result
    end method

    // ... other methods with similar caching logic
end class

// Usage - client code is simple
userRepository = new CachingRemoteUserRepositoryProxy(
    "https://api.example.com",
    new CacheConfig(userTTL: Duration.minutes(5))
)

// All the complexity is hidden
user = userRepository.findById("123")
user.name = "Updated Name"
userRepository.save(user)
```

## Known Uses

- **Java RMI (Remote Method Invocation)**: Stubs act as proxies for remote objects, handling serialization and network communication.

- **Hibernate Lazy Loading**: Proxy objects stand in for entities, loading the real data only when accessed.

- **Spring AOP**: Proxies are created for beans to add cross-cutting concerns like transactions, security, and logging.

- **ES6 Proxy**: JavaScript's built-in Proxy object allows intercepting operations on objects.

- **Python's `__getattr__`**: Can be used to implement proxy behavior for attribute access.

- **ORM Lazy Loading**: Django, ActiveRecord, and other ORMs use proxies for lazy-loaded relationships.

- **CDN (Content Delivery Networks)**: CDN edge servers act as caching proxies for origin servers.

- **API Gateways**: Act as proxies that add authentication, rate limiting, and routing.

- **Virtual Proxies in GUI**: Image placeholders in web browsers and document editors.

- **Copy-on-Write**: File systems and containers use proxy-like mechanisms for efficient copies.

## Related Patterns

- **Adapter**: Adapter provides a different interface to the wrapped object. Proxy provides the same interface.

- **Decorator**: Decorator adds responsibilities to an object. Proxy controls access. Decorator typically does not control object creation; Proxy often does.

- **Facade**: Facade provides a simplified interface to a complex subsystem. Proxy provides the same interface as its subject.

- **Flyweight**: Flyweight explains when and how to share objects. Proxy controls access to a single object.

- **Smart Pointer**: A form of proxy that manages object lifecycle through reference counting.

## When NOT to Use

- **Simple direct access is sufficient**: If there is no need for access control, lazy loading, caching, or remote access, a proxy adds unnecessary complexity.

- **Performance-critical paths**: Proxy adds indirection. In tight loops, this overhead may be unacceptable.

- **When the interface is unstable**: Every change to the Subject interface requires updating the Proxy.

- **For mandatory behavior**: If the additional behavior (logging, caching) must always happen, consider baking it into the real subject rather than relying on clients to use the proxy.

- **When transparency causes confusion**: Sometimes hiding the fact that an object is remote or lazy-loaded can lead to unexpected behavior and difficult debugging.

- **Single-use scenarios**: Creating proxy infrastructure for a one-off use case is overkill.

- **When identity matters**: Clients that rely on object identity will be confused by proxies.

---

## Summary

The Proxy pattern provides a surrogate to control access to another object. Its variations - virtual, protection, remote, caching, logging, and smart reference - address different access control needs. The pattern is particularly valuable for lazy initialization of expensive objects, controlling access based on permissions, hiding network communication complexity, and adding caching without modifying existing code. The key tradeoff is increased complexity versus the benefits of controlled access. Use Proxy when you need to add a layer of control between clients and the real object; avoid it when direct access is simpler and sufficient.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
