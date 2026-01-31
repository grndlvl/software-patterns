# Observer

## Intent

Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

## Also Known As

- Publish-Subscribe (Pub/Sub)
- Dependents
- Listener
- Event-Subscriber

## Motivation

Consider a spreadsheet application where multiple charts and tables display data from the same underlying dataset. When a user changes a cell value, all dependent visualizations must update immediately. The data model shouldn't need to know about every possible visualization type—new chart types might be added later. And visualizations shouldn't poll the data constantly checking for changes.

The Observer pattern elegantly solves this problem. The data model (Subject) maintains a list of observers—objects interested in its changes. When the data changes, the model notifies all registered observers. Each observer (the charts, tables) responds by refreshing its display. The model doesn't know or care what kind of observers exist; it just notifies them.

This decoupling is powerful. You can add new visualization types without modifying the data model. Observers can dynamically subscribe and unsubscribe at runtime. The same observer can watch multiple subjects. And the notification mechanism is entirely automatic—developers don't need to remember to update displays manually.

The pattern embodies the Hollywood Principle: "Don't call us, we'll call you." Instead of observers constantly asking "did anything change?", the subject calls observers when something does change.

## Applicability

Use the Observer pattern when:

- A change to one object requires changing others, and you don't know how many objects need to change.
- An object should notify other objects without making assumptions about what those objects are (loose coupling).
- An abstraction has two aspects, one dependent on the other. Encapsulating these aspects in separate objects lets you vary and reuse them independently.
- You need a publish-subscribe communication model.
- You want to decouple event producers from event consumers.

Common applications include:
- GUI event handling (button clicks, form changes)
- Data binding in MV* frameworks
- Distributed event systems
- Real-time data feeds
- Model-View synchronization
- Notification systems
- Reactive programming foundations

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                   <<interface>> Subject                           │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + attach(observer: Observer): void                                │   │
│    │ + detach(observer: Observer): void                                │   │
│    │ + notify(): void                                                  │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│    ┌───────────────────────────────┴───────────────────────────────────┐   │
│    │                     ConcreteSubject                               │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ - observers: list<Observer>                                       │   │
│    │ - state: any                                                      │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + attach(observer): void                                          │   │
│    │ + detach(observer): void                                          │   │
│    │ + notify(): void                                                  │   │
│    │ + getState(): any                                                 │   │
│    │ + setState(state): void                                           │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                │                                                            │
│                │ notifies                                                   │
│                ▼                                                            │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                   <<interface>> Observer                          │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + update(subject: Subject): void                                  │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│              ┌─────────────────────┼─────────────────────┐                 │
│              │                     │                     │                 │
│    ┌─────────┴─────────┐ ┌────────┴────────┐ ┌─────────┴─────────┐        │
│    │ ConcreteObserverA │ │ConcreteObserverB│ │ ConcreteObserverC │        │
│    ├───────────────────┤ ├─────────────────┤ ├───────────────────┤        │
│    │ - subject: ref    │ │ - subject: ref  │ │ - subjects: list  │        │
│    ├───────────────────┤ ├─────────────────┤ ├───────────────────┤        │
│    │ + update(subject) │ │ + update(subj)  │ │ + update(subject) │        │
│    └───────────────────┘ └─────────────────┘ └───────────────────┘        │
│                                                                             │
│   Notification Flow:                                                        │
│                                                                             │
│   ┌─────────┐     ┌─────────┐     ┌──────────┐     ┌──────────┐           │
│   │ Client  │     │ Subject │     │ObserverA │     │ObserverB │           │
│   └────┬────┘     └────┬────┘     └────┬─────┘     └────┬─────┘           │
│        │               │               │                │                  │
│        │ setState()    │               │                │                  │
│        │──────────────>│               │                │                  │
│        │               │               │                │                  │
│        │               │ update()      │                │                  │
│        │               │──────────────>│                │                  │
│        │               │               │                │                  │
│        │               │ update()      │                │                  │
│        │               │───────────────────────────────>│                  │
│        │               │               │                │                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Subject**: Knows its observers. Any number of observers may observe a subject. Provides an interface for attaching and detaching observer objects.

- **Observer**: Defines an updating interface for objects that should be notified of changes in a subject.

- **ConcreteSubject**: Stores state of interest to ConcreteObservers. Sends a notification to its observers when its state changes.

- **ConcreteObserver**: Maintains a reference to a ConcreteSubject object (optional). Stores state that should stay consistent with the subject's. Implements the Observer updating interface to keep its state consistent.

## Collaborations

1. ConcreteSubject notifies its observers whenever a change occurs that could make its observers' state inconsistent with its own.

2. After being notified of a change, a ConcreteObserver may query the subject for information. The observer uses this information to reconcile its state with that of the subject.

3. The client (or subject itself) is responsible for triggering notifications at appropriate times.

4. Observers can be added or removed at any time, even during notification.

## Consequences

### Benefits

1. **Abstract coupling between Subject and Observer**: All a subject knows is that it has a list of observers conforming to the Observer interface. The subject doesn't know the concrete class of any observer.

2. **Support for broadcast communication**: Unlike an ordinary request, the notification sent by a subject doesn't need to specify a receiver. All interested objects receive the notification automatically.

3. **Dynamic relationships**: Observers can be attached and detached at any time. Subjects and observers can belong to different layers of a system.

4. **Open/Closed Principle**: You can introduce new observer classes without changing the subject (and vice versa).

5. **Establishes runtime relationships**: The coupling between subjects and observers is established at runtime, providing great flexibility.

### Liabilities

1. **Unexpected updates**: Observers don't know about each other's presence. A seemingly innocuous operation on a subject can cause a cascade of updates. The resulting dependency graph isn't obvious from the code.

2. **Memory leaks**: If observers aren't properly detached when no longer needed, the subject holds references to them, preventing garbage collection.

3. **Update overhead**: If there are many observers or updates are frequent, notification can become expensive.

4. **Update ordering**: The order in which observers are notified is typically undefined. Code that depends on a particular order is fragile.

5. **Lapsed listener problem**: Observers that forget to unsubscribe create memory leaks and may cause bugs when notified after their intended lifetime.

## Implementation

### Implementation Considerations

1. **Mapping subjects to observers**: The simplest approach stores observers in the subject. For many subjects with few observers, a separate lookup table may be more efficient.

2. **Observing more than one subject**: An observer might depend on multiple subjects. The update method should include the subject as a parameter so the observer knows which one changed.

3. **Who triggers the update?**:
   - Subject triggers after each state change (simple but may cause excessive updates)
   - Client triggers after a batch of changes (efficient but client must remember)

4. **Push vs. Pull models**:
   - Push: Subject sends detailed change information in the update call
   - Pull: Subject sends minimal notification; observers query for details
   - Push is more efficient but couples observers to specific change data

5. **Specifying modifications of interest**: Observers can register interest in specific events/aspects, receiving only relevant notifications.

6. **Avoiding dangling references**: Subjects should notify observers before being destroyed, or use weak references.

### Pseudocode: Basic Observer Infrastructure

```
// Observer interface
interface Observer {
    method update(subject: Subject, event: Event): void
}

// Subject interface
interface Subject {
    method attach(observer: Observer): void
    method detach(observer: Observer): void
    method notify(event: Event): void
}

// Base subject implementation
class BaseSubject implements Subject {
    private observers: list<Observer> = []

    method attach(observer: Observer): void {
        if observer not in this.observers {
            this.observers.add(observer)
        }
    }

    method detach(observer: Observer): void {
        this.observers.remove(observer)
    }

    method notify(event: Event): void {
        // Create copy to allow modification during iteration
        observersCopy = this.observers.copy()
        for observer in observersCopy {
            observer.update(this, event)
        }
    }
}

// Event types for typed notifications
class Event {
    public type: string
    public data: any
    public timestamp: datetime

    constructor(type: string, data: any = null) {
        this.type = type
        this.data = data
        this.timestamp = now()
    }
}
```

### Pseudocode: Event-Filtered Observer

```
// Observer that filters by event type
interface FilteredObserver {
    method onEvent(subject: Subject, event: Event): void
    method getEventTypes(): list<string>
}

// Subject that routes events to interested observers
class EventSubject implements Subject {
    private observersByEvent: map<string, list<FilteredObserver>> = {}
    private globalObservers: list<Observer> = []

    method attach(observer: Observer): void {
        this.globalObservers.add(observer)
    }

    method attachForEvents(observer: FilteredObserver, eventTypes: list<string>): void {
        for eventType in eventTypes {
            if not this.observersByEvent.containsKey(eventType) {
                this.observersByEvent.set(eventType, [])
            }
            this.observersByEvent.get(eventType).add(observer)
        }
    }

    method detach(observer: Observer): void {
        this.globalObservers.remove(observer)
    }

    method detachFiltered(observer: FilteredObserver): void {
        for eventType, observers in this.observersByEvent {
            observers.remove(observer)
        }
    }

    method notify(event: Event): void {
        // Notify global observers
        for observer in this.globalObservers.copy() {
            observer.update(this, event)
        }

        // Notify filtered observers
        if this.observersByEvent.containsKey(event.type) {
            for observer in this.observersByEvent.get(event.type).copy() {
                observer.onEvent(this, event)
            }
        }
    }
}
```

### Pseudocode: Async Observer

```
// Async observer for non-blocking notifications
interface AsyncObserver {
    method updateAsync(subject: Subject, event: Event): Promise<void>
}

class AsyncSubject {
    private observers: list<AsyncObserver> = []
    private notificationQueue: Queue<Notification> = new Queue()
    private processing: boolean = false

    method attach(observer: AsyncObserver): void {
        this.observers.add(observer)
    }

    method notifyAsync(event: Event): Promise<void> {
        promises = []
        for observer in this.observers {
            promise = observer.updateAsync(this, event)
            promises.add(promise)
        }
        return Promise.all(promises)
    }

    method notifySequential(event: Event): Promise<void> {
        for observer in this.observers {
            await observer.updateAsync(this, event)
        }
    }

    method notifyQueued(event: Event): void {
        this.notificationQueue.enqueue(new Notification(event, this.observers.copy()))
        this.processQueue()
    }

    private async method processQueue(): void {
        if this.processing {
            return
        }

        this.processing = true
        while not this.notificationQueue.isEmpty() {
            notification = this.notificationQueue.dequeue()
            for observer in notification.observers {
                try {
                    await observer.updateAsync(this, notification.event)
                } catch error {
                    log("Observer error: " + error)
                }
            }
        }
        this.processing = false
    }
}
```

## Example

A complete example implementing a stock price monitoring system:

```
// Stock price data
class StockPrice {
    public symbol: string
    public price: float
    public change: float
    public changePercent: float
    public volume: int
    public timestamp: datetime

    constructor(symbol: string, price: float, previousPrice: float, volume: int) {
        this.symbol = symbol
        this.price = price
        this.change = price - previousPrice
        this.changePercent = (this.change / previousPrice) * 100
        this.volume = volume
        this.timestamp = now()
    }
}

// Events
class StockEvent extends Event {
    public stockPrice: StockPrice

    constructor(type: string, stockPrice: StockPrice) {
        super(type, stockPrice)
        this.stockPrice = stockPrice
    }

    static method priceUpdate(stock: StockPrice): StockEvent {
        return new StockEvent("PRICE_UPDATE", stock)
    }

    static method significantChange(stock: StockPrice): StockEvent {
        return new StockEvent("SIGNIFICANT_CHANGE", stock)
    }

    static method volumeSpike(stock: StockPrice): StockEvent {
        return new StockEvent("VOLUME_SPIKE", stock)
    }
}

// Subject: Stock Exchange Feed
class StockExchange implements Subject {
    private observers: list<StockObserver> = []
    private stocks: map<string, StockPrice> = {}
    private significantChangeThreshold: float = 5.0  // 5%
    private volumeSpikeMultiplier: float = 3.0

    method attach(observer: StockObserver): void {
        this.observers.add(observer)
    }

    method detach(observer: StockObserver): void {
        this.observers.remove(observer)
    }

    method notify(event: StockEvent): void {
        for observer in this.observers.copy() {
            try {
                observer.onStockUpdate(this, event)
            } catch error {
                log("Observer error: " + error)
            }
        }
    }

    method updatePrice(symbol: string, price: float, volume: int): void {
        previousPrice = this.stocks.containsKey(symbol) ?
                        this.stocks.get(symbol).price : price

        stockPrice = new StockPrice(symbol, price, previousPrice, volume)
        this.stocks.set(symbol, stockPrice)

        // Always notify price update
        this.notify(StockEvent.priceUpdate(stockPrice))

        // Check for significant change
        if abs(stockPrice.changePercent) >= this.significantChangeThreshold {
            this.notify(StockEvent.significantChange(stockPrice))
        }

        // Check for volume spike
        if this.isVolumeSpike(symbol, volume) {
            this.notify(StockEvent.volumeSpike(stockPrice))
        }
    }

    private method isVolumeSpike(symbol: string, volume: int): boolean {
        // Compare to average volume (simplified)
        averageVolume = this.getAverageVolume(symbol)
        return volume > averageVolume * this.volumeSpikeMultiplier
    }

    method getStock(symbol: string): StockPrice {
        return this.stocks.get(symbol)
    }

    method getAllStocks(): list<StockPrice> {
        return this.stocks.values().toList()
    }
}

// Observer interface for stocks
interface StockObserver extends Observer {
    method onStockUpdate(exchange: StockExchange, event: StockEvent): void
}

// Concrete Observer: Real-time price display
class PriceTickerDisplay implements StockObserver {
    private watchlist: set<string>
    private displayLimit: int

    constructor(watchlist: list<string>, displayLimit: int = 10) {
        this.watchlist = new set(watchlist)
        this.displayLimit = displayLimit
    }

    method onStockUpdate(exchange: StockExchange, event: StockEvent): void {
        if event.type != "PRICE_UPDATE" {
            return
        }

        stock = event.stockPrice
        if stock.symbol not in this.watchlist {
            return
        }

        this.displayPrice(stock)
    }

    method update(subject: Subject, event: Event): void {
        this.onStockUpdate(subject as StockExchange, event as StockEvent)
    }

    private method displayPrice(stock: StockPrice): void {
        changeIndicator = stock.change >= 0 ? "+" : ""
        color = stock.change >= 0 ? "green" : "red"

        print(format(
            "{symbol}: ${price:.2f} ({changeIndicator}{change:.2f}, {changePercent:.2f}%)",
            symbol: stock.symbol,
            price: stock.price,
            changeIndicator: changeIndicator,
            change: stock.change,
            changePercent: stock.changePercent
        ), color: color)
    }

    method addToWatchlist(symbol: string): void {
        this.watchlist.add(symbol)
    }

    method removeFromWatchlist(symbol: string): void {
        this.watchlist.remove(symbol)
    }
}

// Concrete Observer: Alert system
class PriceAlertSystem implements StockObserver {
    private alerts: list<PriceAlert> = []
    private triggeredAlerts: set<string> = {}
    private notificationService: NotificationService

    constructor(notificationService: NotificationService) {
        this.notificationService = notificationService
    }

    method addAlert(alert: PriceAlert): void {
        this.alerts.add(alert)
    }

    method removeAlert(alertId: string): void {
        this.alerts = this.alerts.filter(a => a.id != alertId)
        this.triggeredAlerts.remove(alertId)
    }

    method onStockUpdate(exchange: StockExchange, event: StockEvent): void {
        stock = event.stockPrice

        for alert in this.alerts {
            if alert.symbol != stock.symbol {
                continue
            }

            if this.triggeredAlerts.contains(alert.id) and not alert.repeating {
                continue
            }

            if this.checkAlert(alert, stock) {
                this.triggerAlert(alert, stock)
            }
        }
    }

    method update(subject: Subject, event: Event): void {
        this.onStockUpdate(subject as StockExchange, event as StockEvent)
    }

    private method checkAlert(alert: PriceAlert, stock: StockPrice): boolean {
        switch alert.condition {
            case "ABOVE":
                return stock.price > alert.targetPrice
            case "BELOW":
                return stock.price < alert.targetPrice
            case "CHANGE_PERCENT_ABOVE":
                return abs(stock.changePercent) > alert.targetPercent
            case "VOLUME_ABOVE":
                return stock.volume > alert.targetVolume
            default:
                return false
        }
    }

    private method triggerAlert(alert: PriceAlert, stock: StockPrice): void {
        this.triggeredAlerts.add(alert.id)

        message = this.formatAlertMessage(alert, stock)
        this.notificationService.send(alert.userId, message, alert.channels)

        log("Alert triggered: " + alert.id + " for " + stock.symbol)
    }

    private method formatAlertMessage(alert: PriceAlert, stock: StockPrice): string {
        return format(
            "ALERT: {symbol} is now ${price:.2f} ({condition} {target})",
            symbol: stock.symbol,
            price: stock.price,
            condition: alert.condition,
            target: alert.targetPrice ?? alert.targetPercent + "%"
        )
    }
}

class PriceAlert {
    public id: string
    public userId: string
    public symbol: string
    public condition: string
    public targetPrice: float = null
    public targetPercent: float = null
    public targetVolume: int = null
    public repeating: boolean = false
    public channels: list<string>  // "email", "sms", "push"

    constructor(userId: string, symbol: string, condition: string) {
        this.id = generateUUID()
        this.userId = userId
        this.symbol = symbol
        this.condition = condition
        this.channels = ["push"]
    }
}

// Concrete Observer: Portfolio tracker
class PortfolioTracker implements StockObserver {
    private holdings: map<string, Holding> = {}
    private totalValue: float = 0
    private totalCost: float = 0

    method addHolding(symbol: string, shares: int, costBasis: float): void {
        this.holdings.set(symbol, new Holding(symbol, shares, costBasis))
        this.totalCost = this.totalCost + (shares * costBasis)
    }

    method onStockUpdate(exchange: StockExchange, event: StockEvent): void {
        if event.type != "PRICE_UPDATE" {
            return
        }

        stock = event.stockPrice
        if not this.holdings.containsKey(stock.symbol) {
            return
        }

        holding = this.holdings.get(stock.symbol)
        holding.currentPrice = stock.price
        holding.marketValue = holding.shares * stock.price
        holding.gain = holding.marketValue - (holding.shares * holding.costBasis)
        holding.gainPercent = (holding.gain / (holding.shares * holding.costBasis)) * 100

        this.recalculateTotal()
    }

    method update(subject: Subject, event: Event): void {
        this.onStockUpdate(subject as StockExchange, event as StockEvent)
    }

    private method recalculateTotal(): void {
        this.totalValue = 0
        for symbol, holding in this.holdings {
            this.totalValue = this.totalValue + holding.marketValue
        }
    }

    method getPortfolioSummary(): PortfolioSummary {
        return new PortfolioSummary(
            holdings: this.holdings.values().toList(),
            totalValue: this.totalValue,
            totalCost: this.totalCost,
            totalGain: this.totalValue - this.totalCost,
            totalGainPercent: ((this.totalValue - this.totalCost) / this.totalCost) * 100
        )
    }
}

class Holding {
    public symbol: string
    public shares: int
    public costBasis: float
    public currentPrice: float = 0
    public marketValue: float = 0
    public gain: float = 0
    public gainPercent: float = 0

    constructor(symbol: string, shares: int, costBasis: float) {
        this.symbol = symbol
        this.shares = shares
        this.costBasis = costBasis
    }
}

// Concrete Observer: Trading bot
class TradingBot implements StockObserver {
    private strategy: TradingStrategy
    private broker: BrokerAPI
    private positions: map<string, Position> = {}
    private enabled: boolean = true

    constructor(strategy: TradingStrategy, broker: BrokerAPI) {
        this.strategy = strategy
        this.broker = broker
    }

    method onStockUpdate(exchange: StockExchange, event: StockEvent): void {
        if not this.enabled {
            return
        }

        stock = event.stockPrice

        // Analyze and potentially trade
        signal = this.strategy.analyze(stock, event.type)

        if signal.action == "BUY" {
            this.executeBuy(stock, signal.quantity)
        } else if signal.action == "SELL" {
            this.executeSell(stock, signal.quantity)
        }
    }

    method update(subject: Subject, event: Event): void {
        this.onStockUpdate(subject as StockExchange, event as StockEvent)
    }

    private method executeBuy(stock: StockPrice, quantity: int): void {
        try {
            order = this.broker.buy(stock.symbol, quantity, stock.price)
            log("BUY executed: " + quantity + " shares of " + stock.symbol)
            this.updatePosition(stock.symbol, quantity)
        } catch error {
            log("BUY failed: " + error)
        }
    }

    private method executeSell(stock: StockPrice, quantity: int): void {
        try {
            order = this.broker.sell(stock.symbol, quantity, stock.price)
            log("SELL executed: " + quantity + " shares of " + stock.symbol)
            this.updatePosition(stock.symbol, -quantity)
        } catch error {
            log("SELL failed: " + error)
        }
    }

    method enable(): void {
        this.enabled = true
    }

    method disable(): void {
        this.enabled = false
    }
}

// Concrete Observer: Data logger for analysis
class MarketDataLogger implements StockObserver {
    private database: TimeSeriesDB
    private batchSize: int = 100
    private buffer: list<StockPrice> = []

    constructor(database: TimeSeriesDB) {
        this.database = database
    }

    method onStockUpdate(exchange: StockExchange, event: StockEvent): void {
        this.buffer.add(event.stockPrice)

        if this.buffer.size() >= this.batchSize {
            this.flush()
        }
    }

    method update(subject: Subject, event: Event): void {
        this.onStockUpdate(subject as StockExchange, event as StockEvent)
    }

    method flush(): void {
        if this.buffer.isEmpty() {
            return
        }

        try {
            this.database.batchInsert("stock_prices", this.buffer)
            this.buffer.clear()
        } catch error {
            log("Failed to write to database: " + error)
        }
    }

    method close(): void {
        this.flush()
    }
}

// Usage example
function main() {
    // Create the exchange (subject)
    exchange = new StockExchange()

    // Create observers
    ticker = new PriceTickerDisplay(["AAPL", "GOOGL", "MSFT", "AMZN"])

    alertSystem = new PriceAlertSystem(new EmailNotificationService())
    alertSystem.addAlert(new PriceAlert("user1", "AAPL", "ABOVE").setTargetPrice(200))
    alertSystem.addAlert(new PriceAlert("user1", "GOOGL", "CHANGE_PERCENT_ABOVE").setTargetPercent(3))

    portfolio = new PortfolioTracker()
    portfolio.addHolding("AAPL", 100, 150.00)
    portfolio.addHolding("GOOGL", 50, 2500.00)

    logger = new MarketDataLogger(new InfluxDB("localhost"))

    // Register observers
    exchange.attach(ticker)
    exchange.attach(alertSystem)
    exchange.attach(portfolio)
    exchange.attach(logger)

    // Simulate market updates
    exchange.updatePrice("AAPL", 198.50, 1000000)
    exchange.updatePrice("GOOGL", 2780.00, 500000)
    exchange.updatePrice("MSFT", 380.25, 750000)

    // Price spike triggers alert
    exchange.updatePrice("AAPL", 205.00, 2000000)
    // Alert system sends notification: "AAPL is now $205.00 (ABOVE $200)"

    // Check portfolio
    summary = portfolio.getPortfolioSummary()
    print("Portfolio value: $" + summary.totalValue)
    print("Total gain: $" + summary.totalGain + " (" + summary.totalGainPercent + "%)")

    // Unsubscribe ticker (no longer interested)
    exchange.detach(ticker)

    // More updates (ticker won't receive these)
    exchange.updatePrice("AAPL", 203.00, 1500000)

    // Cleanup
    logger.close()
}
```

## Known Uses

- **Java AWT/Swing Event Model**: `ActionListener`, `MouseListener`, etc. are observers of UI components.

- **JavaScript DOM Events**: `addEventListener` implements the Observer pattern for DOM events.

- **React/Vue Reactivity**: State management systems use observers to trigger re-renders when state changes.

- **RxJS/Reactive Extensions**: The entire reactive programming paradigm is built on Observer.

- **Node.js EventEmitter**: The foundation of Node's event-driven architecture.

- **Android LiveData**: Lifecycle-aware observable data holder.

- **.NET Events**: C# events and delegates implement publish-subscribe.

- **MVC Frameworks**: Models notify views of changes in virtually all MVC implementations.

- **Message Queues**: Pub/sub systems like Redis, RabbitMQ implement Observer at a distributed level.

- **WebSockets**: Real-time web applications use observer-like patterns for server push.

## Related Patterns

- **Mediator**: Can be used to encapsulate complex update semantics. Instead of observers updating themselves, a mediator coordinates updates.

- **Singleton**: Subjects are often singletons (event buses, application state managers).

- **Command**: Notifications can be encapsulated as Command objects, especially useful for queuing or logging.

- **Memento**: Observers can store subject state snapshots when notified, enabling undo.

- **Strategy**: Observers can use different strategies to handle updates.

## When NOT to Use

1. **Simple one-to-one relationships**: If there's only ever one observer, direct method calls are simpler.

```
// Overkill: Observer for single listener
class Button {
    private observer: ClickObserver  // Only ever one
}

// Just use a callback
class Button {
    private onClick: function(): void
}
```

2. **When update order matters critically**: Observer doesn't guarantee notification order. If order is crucial, use a different mechanism.

3. **Synchronous chains causing performance issues**: Long chains of synchronous observer updates can block the main thread. Consider async notifications or batching.

4. **When observers need responses**: Observer is one-way notification. If the subject needs responses from observers, use a different pattern.

5. **Tight coupling is acceptable**: In small, cohesive modules where coupling isn't a concern, direct calls are clearer.

6. **Memory-constrained environments**: Observer requires maintaining lists of references. In extremely memory-tight situations, this overhead may be unacceptable.

```
// Memory leak waiting to happen
class LongLivedSubject {
    observers: list<ShortLivedObserver>  // These never get removed
}

// Observers are created frequently but never detached
for i = 0 to 10000 {
    subject.attach(new ShortLivedObserver())  // Memory leak!
}
```

7. **Debugging complex notification chains**: When many observers trigger further notifications, the execution flow becomes hard to trace. Consider whether simpler direct calls would be more maintainable.

The Observer pattern excels at decoupling event sources from handlers in dynamic, loosely coupled systems. Avoid it when the complexity isn't justified or when its dynamic nature makes debugging difficult.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
