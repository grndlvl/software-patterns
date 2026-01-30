# Strategy

## Intent

Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

## Also Known As

- Policy

## Motivation

Consider a navigation application that calculates routes between two points. Users might want the fastest route, the shortest route, the most scenic route, or a route that avoids highways. Each routing algorithm is complex, involves different considerations, and produces different results. How do you support multiple algorithms without cluttering the navigation code with conditionals?

The naive approach embeds all algorithms in the navigation class: `if (mode == FASTEST) ... else if (mode == SHORTEST) ...`. This creates a monolithic class that's hard to understand, test, and extend. Adding a new routing strategy means modifying the navigation class, risking bugs in existing algorithms.

The Strategy pattern solves this by defining each routing algorithm in its own class, all implementing a common interface. The navigation application holds a reference to a routing strategy and delegates route calculation to it. Switching algorithms is as simple as swapping the strategy object—the navigation code doesn't change.

This design offers remarkable flexibility. New algorithms can be added without modifying existing code. Algorithms can be selected at runtime based on user preferences, conditions, or configuration. Each algorithm is isolated, making it easy to test, optimize, or replace. And the pattern clearly separates the concerns of "what route to calculate" from "how to calculate it."

## Applicability

Use the Strategy pattern when:

- Many related classes differ only in their behavior. Strategies provide a way to configure a class with one of many behaviors.
- You need different variants of an algorithm. Strategies can be used when these variants are implemented as a class hierarchy of algorithms.
- An algorithm uses data that clients shouldn't know about. Use Strategy to avoid exposing complex, algorithm-specific data structures.
- A class defines many behaviors, and these appear as multiple conditional statements in its operations. Move related conditional branches into their own Strategy class.
- You want to be able to swap algorithms at runtime.

Common applications include:
- Sorting algorithms (quicksort, mergesort, heapsort)
- Compression algorithms (gzip, bzip2, lzma)
- Payment processing (credit card, PayPal, crypto)
- Authentication methods (password, OAuth, biometric)
- Pricing strategies (regular, promotional, loyalty)
- Validation strategies
- Rendering engines
- AI behaviors in games

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                         Context                                   │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ - strategy: Strategy                                              │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + setStrategy(strategy: Strategy): void                           │   │
│    │ + executeStrategy(): Result                                       │   │
│    │   // delegates to strategy.execute()                              │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                │                                                            │
│                │ uses                                                       │
│                ▼                                                            │
│    ┌───────────────────────────────────────────────────────────────────┐   │
│    │                   <<interface>> Strategy                          │   │
│    ├───────────────────────────────────────────────────────────────────┤   │
│    │ + execute(data: Input): Result                                    │   │
│    └───────────────────────────────────────────────────────────────────┘   │
│                                    △                                        │
│                                    │                                        │
│              ┌─────────────────────┼─────────────────────┐                 │
│              │                     │                     │                 │
│    ┌─────────┴─────────┐ ┌────────┴────────┐ ┌─────────┴─────────┐        │
│    │ ConcreteStrategyA │ │ConcreteStrategyB│ │ ConcreteStrategyC │        │
│    ├───────────────────┤ ├─────────────────┤ ├───────────────────┤        │
│    │                   │ │                 │ │                   │        │
│    ├───────────────────┤ ├─────────────────┤ ├───────────────────┤        │
│    │ + execute(data)   │ │ + execute(data) │ │ + execute(data)   │        │
│    │   // Algorithm A  │ │   // Algorithm B│ │   // Algorithm C  │        │
│    └───────────────────┘ └─────────────────┘ └───────────────────┘        │
│                                                                             │
│   Runtime Strategy Selection:                                              │
│                                                                             │
│   ┌────────┐     setStrategy(A)     ┌─────────┐     execute()              │
│   │ Client │───────────────────────>│ Context │───────────────>  Result   │
│   └────────┘                        │         │                            │
│       │                             │strategy │                            │
│       │         setStrategy(B)      │   = A   │                            │
│       │───────────────────────────> │         │                            │
│       │                             │strategy │     execute()              │
│       │                             │   = B   │───────────────>  Result'  │
│       │                             └─────────┘                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Strategy**: Declares an interface common to all supported algorithms. Context uses this interface to call the algorithm defined by a ConcreteStrategy.

- **ConcreteStrategy**: Implements the algorithm using the Strategy interface. Each concrete strategy encapsulates a specific algorithm.

- **Context**: Is configured with a ConcreteStrategy object. Maintains a reference to a Strategy object. May define an interface that lets Strategy access its data. Delegates algorithm execution to the Strategy.

## Collaborations

1. Strategy and Context interact to implement the chosen algorithm. A context may pass all data required by the algorithm to the strategy when the algorithm is called. Alternatively, the context can pass itself as an argument to Strategy operations, letting the strategy call back to the context as required.

2. A context forwards requests from its clients to its strategy. Clients usually create and pass a ConcreteStrategy object to the context; thereafter, clients interact with the context exclusively. There is often a family of ConcreteStrategy classes for a client to choose from.

## Consequences

### Benefits

1. **Families of related algorithms**: Hierarchies of Strategy classes define a family of algorithms or behaviors for contexts to reuse. Inheritance can factor out common functionality.

2. **Alternative to subclassing**: Instead of subclassing Context to vary the algorithm, you compose it with different strategies. This avoids the need for a complex class hierarchy.

3. **Eliminates conditional statements**: Strategy removes conditional statements for selecting desired behavior. When different behaviors are lumped into one class, it's hard to avoid using conditional statements. Strategy moves this selection outside the context.

4. **Choice of implementations**: Strategies can provide different implementations of the same behavior. The client can choose among strategies with different time and space trade-offs.

5. **Open/Closed Principle**: You can introduce new strategies without changing the context.

6. **Runtime flexibility**: Strategies can be swapped at runtime, allowing dynamic behavior changes.

### Liabilities

1. **Clients must be aware of strategies**: Clients must understand how strategies differ to select the appropriate one. This can expose implementation details.

2. **Communication overhead**: The Strategy interface is shared by all ConcreteStrategy classes whether the algorithms use all the information passed to them or not. Some strategies may not use all parameters.

3. **Increased number of objects**: Strategies increase the number of objects in an application. This can be mitigated by implementing strategies as stateless singletons.

## Implementation

### Implementation Considerations

1. **Defining the Strategy interface**: The interface must be general enough to support different algorithms but specific enough to be useful. Two approaches:
   - Pass data as parameters: Context passes required data to strategy methods.
   - Pass context reference: Strategy extracts what it needs from the context.

2. **Strategies as function objects**: In languages with first-class functions, strategies can be functions/lambdas rather than full classes. This reduces boilerplate for simple strategies.

3. **Making strategies optional**: The context can have a default strategy or check for null, allowing optional strategy configuration.

4. **Strategy selection**: Who selects the strategy? Options include:
   - Client explicitly sets strategy
   - Context chooses based on configuration
   - Factory creates appropriate strategy
   - Dependency injection provides strategy

### Pseudocode: Strategy Infrastructure

```
// Strategy interface
interface PaymentStrategy {
    method pay(amount: float): PaymentResult
    method validate(): ValidationResult
    method getName(): string
    method getSupportedCurrencies(): list<string>
}

// Payment result
class PaymentResult {
    public success: boolean
    public transactionId: string
    public message: string
    public processingFee: float
    public timestamp: datetime
}

// Validation result
class ValidationResult {
    public valid: boolean
    public errors: list<string>
}
```

### Pseudocode: Concrete Strategies

```
// Credit Card Strategy
class CreditCardStrategy implements PaymentStrategy {
    private cardNumber: string
    private expiryDate: string
    private cvv: string
    private cardholderName: string
    private gateway: PaymentGateway

    constructor(cardNumber: string, expiryDate: string, cvv: string, name: string) {
        this.cardNumber = cardNumber
        this.expiryDate = expiryDate
        this.cvv = cvv
        this.cardholderName = name
        this.gateway = new StripeGateway()
    }

    method pay(amount: float): PaymentResult {
        // Tokenize card for security
        token = this.gateway.tokenize(this.cardNumber, this.expiryDate, this.cvv)

        // Process payment
        response = this.gateway.charge(token, amount, "USD")

        return new PaymentResult(
            success: response.status == "succeeded",
            transactionId: response.id,
            message: response.message,
            processingFee: amount * 0.029 + 0.30,  // 2.9% + $0.30
            timestamp: now()
        )
    }

    method validate(): ValidationResult {
        errors = []

        // Validate card number (Luhn algorithm)
        if not this.isValidLuhn(this.cardNumber) {
            errors.add("Invalid card number")
        }

        // Validate expiry
        if this.isExpired(this.expiryDate) {
            errors.add("Card has expired")
        }

        // Validate CVV
        if not this.cvv.matches("^[0-9]{3,4}$") {
            errors.add("Invalid CVV")
        }

        return new ValidationResult(
            valid: errors.isEmpty(),
            errors: errors
        )
    }

    method getName(): string {
        return "Credit Card"
    }

    method getSupportedCurrencies(): list<string> {
        return ["USD", "EUR", "GBP", "CAD", "AUD"]
    }

    private method isValidLuhn(number: string): boolean {
        // Luhn algorithm implementation
        // ...
    }

    private method isExpired(expiry: string): boolean {
        // Parse MM/YY and check against current date
        // ...
    }
}

// PayPal Strategy
class PayPalStrategy implements PaymentStrategy {
    private email: string
    private accessToken: string
    private client: PayPalClient

    constructor(email: string) {
        this.email = email
        this.client = new PayPalClient()
    }

    method authenticate(): boolean {
        // OAuth flow to get access token
        this.accessToken = this.client.getAccessToken(this.email)
        return this.accessToken is not null
    }

    method pay(amount: float): PaymentResult {
        if this.accessToken is null {
            return new PaymentResult(
                success: false,
                message: "Not authenticated with PayPal"
            )
        }

        response = this.client.createPayment(this.accessToken, amount, "USD")

        return new PaymentResult(
            success: response.state == "approved",
            transactionId: response.id,
            message: response.state == "approved" ? "Payment successful" : response.failureReason,
            processingFee: amount * 0.034 + 0.49,  // 3.4% + $0.49
            timestamp: now()
        )
    }

    method validate(): ValidationResult {
        errors = []

        if not this.isValidEmail(this.email) {
            errors.add("Invalid email address")
        }

        return new ValidationResult(
            valid: errors.isEmpty(),
            errors: errors
        )
    }

    method getName(): string {
        return "PayPal"
    }

    method getSupportedCurrencies(): list<string> {
        return ["USD", "EUR", "GBP", "CAD", "AUD", "JPY", "BRL"]
    }
}

// Cryptocurrency Strategy
class CryptoStrategy implements PaymentStrategy {
    private walletAddress: string
    private currency: string  // "BTC", "ETH", etc.
    private exchangeService: CryptoExchangeService

    constructor(walletAddress: string, currency: string) {
        this.walletAddress = walletAddress
        this.currency = currency
        this.exchangeService = new CoinbaseExchangeService()
    }

    method pay(amount: float): PaymentResult {
        // Convert USD amount to crypto
        cryptoAmount = this.exchangeService.convertToCrypto(amount, "USD", this.currency)

        // Generate payment request
        paymentRequest = this.exchangeService.createPaymentRequest(
            this.walletAddress,
            cryptoAmount,
            this.currency
        )

        // Wait for blockchain confirmation (simplified)
        confirmation = this.exchangeService.waitForConfirmation(paymentRequest.id, timeout: 30)

        return new PaymentResult(
            success: confirmation.confirmed,
            transactionId: confirmation.txHash,
            message: confirmation.confirmed ? "Payment confirmed" : "Payment timeout",
            processingFee: amount * 0.015,  // 1.5%
            timestamp: now()
        )
    }

    method validate(): ValidationResult {
        errors = []

        // Validate wallet address format
        if not this.isValidWalletAddress(this.walletAddress, this.currency) {
            errors.add("Invalid " + this.currency + " wallet address")
        }

        return new ValidationResult(
            valid: errors.isEmpty(),
            errors: errors
        )
    }

    method getName(): string {
        return "Cryptocurrency (" + this.currency + ")"
    }

    method getSupportedCurrencies(): list<string> {
        return [this.currency]  // Only the selected crypto
    }
}

// Bank Transfer Strategy
class BankTransferStrategy implements PaymentStrategy {
    private accountNumber: string
    private routingNumber: string
    private accountName: string
    private achProcessor: ACHProcessor

    constructor(accountNumber: string, routingNumber: string, accountName: string) {
        this.accountNumber = accountNumber
        this.routingNumber = routingNumber
        this.accountName = accountName
        this.achProcessor = new ACHProcessor()
    }

    method pay(amount: float): PaymentResult {
        // ACH transfers are not instant
        transfer = this.achProcessor.initiateTransfer(
            this.routingNumber,
            this.accountNumber,
            amount
        )

        return new PaymentResult(
            success: transfer.status == "pending",
            transactionId: transfer.id,
            message: "Transfer initiated. Funds will be available in 2-3 business days.",
            processingFee: 0.25,  // Flat fee
            timestamp: now()
        )
    }

    method validate(): ValidationResult {
        errors = []

        // Validate routing number (9 digits, checksum)
        if not this.isValidRoutingNumber(this.routingNumber) {
            errors.add("Invalid routing number")
        }

        // Validate account number
        if this.accountNumber.length < 4 or this.accountNumber.length > 17 {
            errors.add("Invalid account number")
        }

        return new ValidationResult(
            valid: errors.isEmpty(),
            errors: errors
        )
    }

    method getName(): string {
        return "Bank Transfer (ACH)"
    }

    method getSupportedCurrencies(): list<string> {
        return ["USD"]  // ACH is US only
    }
}
```

### Pseudocode: Context

```
class PaymentProcessor {
    private strategy: PaymentStrategy

    method setPaymentStrategy(strategy: PaymentStrategy): void {
        this.strategy = strategy
    }

    method getPaymentStrategy(): PaymentStrategy {
        return this.strategy
    }

    method processPayment(amount: float): PaymentResult {
        if this.strategy is null {
            throw new ConfigurationException("Payment strategy not set")
        }

        // Validate first
        validation = this.strategy.validate()
        if not validation.valid {
            return new PaymentResult(
                success: false,
                message: "Validation failed: " + join(validation.errors, ", ")
            )
        }

        // Process payment
        return this.strategy.pay(amount)
    }

    method getAvailableStrategies(): list<PaymentStrategyInfo> {
        return [
            new PaymentStrategyInfo("credit_card", "Credit Card", ["USD", "EUR", "GBP"]),
            new PaymentStrategyInfo("paypal", "PayPal", ["USD", "EUR", "GBP"]),
            new PaymentStrategyInfo("crypto_btc", "Bitcoin", ["BTC"]),
            new PaymentStrategyInfo("crypto_eth", "Ethereum", ["ETH"]),
            new PaymentStrategyInfo("bank_transfer", "Bank Transfer", ["USD"])
        ]
    }
}
```

## Example

A complete example implementing a text compression system:

```
// Compression strategy interface
interface CompressionStrategy {
    method compress(data: bytes): CompressedData
    method decompress(compressed: CompressedData): bytes
    method getName(): string
    method getFileExtension(): string
    method estimateCompressionRatio(data: bytes): float
    method supportsStreaming(): boolean
}

class CompressedData {
    public algorithm: string
    public originalSize: int
    public compressedSize: int
    public data: bytes
    public checksum: string

    method getCompressionRatio(): float {
        return this.compressedSize / this.originalSize
    }
}

// GZIP Strategy
class GzipStrategy implements CompressionStrategy {
    private level: int  // 1-9, higher = better compression, slower

    constructor(level: int = 6) {
        this.level = clamp(level, 1, 9)
    }

    method compress(data: bytes): CompressedData {
        compressed = gzipCompress(data, level: this.level)
        checksum = crc32(data)

        return new CompressedData(
            algorithm: this.getName(),
            originalSize: data.length,
            compressedSize: compressed.length,
            data: compressed,
            checksum: checksum
        )
    }

    method decompress(compressed: CompressedData): bytes {
        data = gzipDecompress(compressed.data)

        // Verify checksum
        if crc32(data) != compressed.checksum {
            throw new CorruptedDataException("Checksum mismatch")
        }

        return data
    }

    method getName(): string {
        return "gzip"
    }

    method getFileExtension(): string {
        return ".gz"
    }

    method estimateCompressionRatio(data: bytes): float {
        // GZIP typically achieves 60-90% reduction for text
        if isTextData(data) {
            return 0.3  // 70% reduction
        }
        return 0.7  // 30% reduction for binary
    }

    method supportsStreaming(): boolean {
        return true
    }
}

// LZ4 Strategy (fast compression)
class Lz4Strategy implements CompressionStrategy {
    private highCompression: boolean

    constructor(highCompression: boolean = false) {
        this.highCompression = highCompression
    }

    method compress(data: bytes): CompressedData {
        compressed = this.highCompression ?
                     lz4CompressHC(data) :
                     lz4Compress(data)

        return new CompressedData(
            algorithm: this.getName(),
            originalSize: data.length,
            compressedSize: compressed.length,
            data: compressed,
            checksum: xxhash64(data)
        )
    }

    method decompress(compressed: CompressedData): bytes {
        data = lz4Decompress(compressed.data, compressed.originalSize)

        if xxhash64(data) != compressed.checksum {
            throw new CorruptedDataException("Checksum mismatch")
        }

        return data
    }

    method getName(): string {
        return this.highCompression ? "lz4-hc" : "lz4"
    }

    method getFileExtension(): string {
        return ".lz4"
    }

    method estimateCompressionRatio(data: bytes): float {
        // LZ4 trades compression ratio for speed
        return 0.5  // ~50% reduction typical
    }

    method supportsStreaming(): boolean {
        return true
    }
}

// LZMA Strategy (high compression)
class LzmaStrategy implements CompressionStrategy {
    private preset: int  // 0-9

    constructor(preset: int = 6) {
        this.preset = clamp(preset, 0, 9)
    }

    method compress(data: bytes): CompressedData {
        compressed = lzmaCompress(data, preset: this.preset)

        return new CompressedData(
            algorithm: this.getName(),
            originalSize: data.length,
            compressedSize: compressed.length,
            data: compressed,
            checksum: sha256(data)
        )
    }

    method decompress(compressed: CompressedData): bytes {
        data = lzmaDecompress(compressed.data)

        if sha256(data) != compressed.checksum {
            throw new CorruptedDataException("Checksum mismatch")
        }

        return data
    }

    method getName(): string {
        return "lzma"
    }

    method getFileExtension(): string {
        return ".xz"
    }

    method estimateCompressionRatio(data: bytes): float {
        // LZMA achieves best compression
        if isTextData(data) {
            return 0.15  // 85% reduction for text
        }
        return 0.5
    }

    method supportsStreaming(): boolean {
        return false  // LZMA needs full data for best compression
    }
}

// Zstandard Strategy (balanced)
class ZstdStrategy implements CompressionStrategy {
    private level: int  // 1-22

    constructor(level: int = 3) {
        this.level = clamp(level, 1, 22)
    }

    method compress(data: bytes): CompressedData {
        compressed = zstdCompress(data, level: this.level)

        return new CompressedData(
            algorithm: this.getName(),
            originalSize: data.length,
            compressedSize: compressed.length,
            data: compressed,
            checksum: xxhash64(data)
        )
    }

    method decompress(compressed: CompressedData): bytes {
        data = zstdDecompress(compressed.data)

        if xxhash64(data) != compressed.checksum {
            throw new CorruptedDataException("Checksum mismatch")
        }

        return data
    }

    method getName(): string {
        return "zstd"
    }

    method getFileExtension(): string {
        return ".zst"
    }

    method estimateCompressionRatio(data: bytes): float {
        // Zstd is between gzip and lzma
        if isTextData(data) {
            return 0.25
        }
        return 0.6
    }

    method supportsStreaming(): boolean {
        return true
    }
}

// No compression strategy (passthrough)
class NoCompressionStrategy implements CompressionStrategy {
    method compress(data: bytes): CompressedData {
        return new CompressedData(
            algorithm: "none",
            originalSize: data.length,
            compressedSize: data.length,
            data: data.copy(),
            checksum: crc32(data)
        )
    }

    method decompress(compressed: CompressedData): bytes {
        return compressed.data.copy()
    }

    method getName(): string {
        return "none"
    }

    method getFileExtension(): string {
        return ""
    }

    method estimateCompressionRatio(data: bytes): float {
        return 1.0  // No compression
    }

    method supportsStreaming(): boolean {
        return true
    }
}

// Context: File archiver
class FileArchiver {
    private strategy: CompressionStrategy
    private strategyRegistry: map<string, CompressionStrategy>

    constructor() {
        this.strategy = new GzipStrategy()  // Default
        this.registerStrategies()
    }

    private method registerStrategies(): void {
        this.strategyRegistry = {
            "gzip": new GzipStrategy(),
            "gzip-fast": new GzipStrategy(1),
            "gzip-best": new GzipStrategy(9),
            "lz4": new Lz4Strategy(),
            "lz4-hc": new Lz4Strategy(true),
            "lzma": new LzmaStrategy(),
            "zstd": new ZstdStrategy(),
            "zstd-fast": new ZstdStrategy(1),
            "zstd-best": new ZstdStrategy(19),
            "none": new NoCompressionStrategy()
        }
    }

    method setStrategy(strategy: CompressionStrategy): void {
        this.strategy = strategy
    }

    method setStrategyByName(name: string): void {
        if not this.strategyRegistry.containsKey(name) {
            throw new UnknownStrategyException("Unknown compression: " + name)
        }
        this.strategy = this.strategyRegistry.get(name)
    }

    method getAvailableStrategies(): list<string> {
        return this.strategyRegistry.keys().toList()
    }

    method compressFile(inputPath: string, outputPath: string = null): CompressionResult {
        // Read file
        data = readFile(inputPath)

        // Compress
        startTime = now()
        compressed = this.strategy.compress(data)
        elapsed = now() - startTime

        // Determine output path
        if outputPath is null {
            outputPath = inputPath + this.strategy.getFileExtension()
        }

        // Write compressed data with header
        header = this.createHeader(compressed)
        writeFile(outputPath, header + compressed.data)

        return new CompressionResult(
            inputPath: inputPath,
            outputPath: outputPath,
            originalSize: compressed.originalSize,
            compressedSize: compressed.compressedSize,
            ratio: compressed.getCompressionRatio(),
            algorithm: compressed.algorithm,
            duration: elapsed
        )
    }

    method decompressFile(inputPath: string, outputPath: string = null): DecompressionResult {
        // Read file
        fileData = readFile(inputPath)

        // Parse header
        header, compressedBytes = this.parseHeader(fileData)

        // Select strategy based on header
        if not this.strategyRegistry.containsKey(header.algorithm) {
            throw new UnknownStrategyException("Unknown compression in file: " + header.algorithm)
        }
        strategy = this.strategyRegistry.get(header.algorithm)

        // Create CompressedData from file
        compressed = new CompressedData(
            algorithm: header.algorithm,
            originalSize: header.originalSize,
            compressedSize: compressedBytes.length,
            data: compressedBytes,
            checksum: header.checksum
        )

        // Decompress
        startTime = now()
        data = strategy.decompress(compressed)
        elapsed = now() - startTime

        // Determine output path
        if outputPath is null {
            outputPath = inputPath.removeSuffix(strategy.getFileExtension())
        }

        // Write decompressed data
        writeFile(outputPath, data)

        return new DecompressionResult(
            inputPath: inputPath,
            outputPath: outputPath,
            compressedSize: compressed.compressedSize,
            originalSize: data.length,
            algorithm: header.algorithm,
            duration: elapsed
        )
    }

    // Auto-select best strategy based on data characteristics
    method compressFileAuto(inputPath: string): CompressionResult {
        data = readFile(inputPath)
        bestStrategy = this.selectBestStrategy(data)
        this.setStrategy(bestStrategy)
        return this.compressFile(inputPath)
    }

    private method selectBestStrategy(data: bytes): CompressionStrategy {
        // For small files, use fast compression
        if data.length < 10 * 1024 {  // < 10KB
            return this.strategyRegistry.get("lz4")
        }

        // For large files, use zstd for good balance
        if data.length > 100 * 1024 * 1024 {  // > 100MB
            return this.strategyRegistry.get("zstd")
        }

        // For text files, use high compression
        if isTextData(data) {
            return this.strategyRegistry.get("zstd-best")
        }

        // Default to gzip for compatibility
        return this.strategyRegistry.get("gzip")
    }

    method benchmark(data: bytes): list<BenchmarkResult> {
        results = []

        for name, strategy in this.strategyRegistry {
            if name == "none" {
                continue
            }

            // Time compression
            startCompress = now()
            compressed = strategy.compress(data)
            compressTime = now() - startCompress

            // Time decompression
            startDecompress = now()
            decompressed = strategy.decompress(compressed)
            decompressTime = now() - startDecompress

            // Verify
            if decompressed != data {
                throw new VerificationException("Data mismatch for " + name)
            }

            results.add(new BenchmarkResult(
                algorithm: name,
                originalSize: data.length,
                compressedSize: compressed.compressedSize,
                ratio: compressed.getCompressionRatio(),
                compressTime: compressTime,
                decompressTime: decompressTime,
                compressSpeed: data.length / compressTime.seconds,
                decompressSpeed: data.length / decompressTime.seconds
            ))
        }

        return results.sortedBy(r => r.ratio)
    }
}

// Usage example
function main() {
    archiver = new FileArchiver()

    // List available strategies
    print("Available compression algorithms:")
    for name in archiver.getAvailableStrategies() {
        print("  - " + name)
    }

    // Compress with default (gzip)
    result = archiver.compressFile("/data/document.txt")
    print("Compressed: " + result.originalSize + " -> " + result.compressedSize +
          " (" + (result.ratio * 100).toFixed(1) + "%)")

    // Compress with specific strategy
    archiver.setStrategyByName("zstd-best")
    result = archiver.compressFile("/data/large-file.bin")
    print("Zstd: " + (result.ratio * 100).toFixed(1) + "% in " + result.duration + "ms")

    // Auto-select best strategy
    result = archiver.compressFileAuto("/data/mixed-content.dat")
    print("Auto selected: " + result.algorithm)

    // Decompress (auto-detects algorithm from file header)
    decompressResult = archiver.decompressFile("/data/document.txt.gz")

    // Benchmark all strategies
    testData = readFile("/data/sample.txt")
    benchmarks = archiver.benchmark(testData)

    print("\nBenchmark results:")
    print("Algorithm      | Ratio  | Compress  | Decompress")
    print("---------------|--------|-----------|------------")
    for b in benchmarks {
        print(format("{algo:14} | {ratio:5.1f}% | {cspeed:6.1f} MB/s | {dspeed:6.1f} MB/s",
            algo: b.algorithm,
            ratio: b.ratio * 100,
            cspeed: b.compressSpeed / 1024 / 1024,
            dspeed: b.decompressSpeed / 1024 / 1024
        ))
    }
}
```

## Known Uses

- **Java Collections**: `Collections.sort()` accepts a `Comparator` strategy for custom ordering.

- **Java I/O**: `LayoutManager` in Swing (FlowLayout, BorderLayout, GridLayout) are layout strategies.

- **Python sorted()**: The `key` parameter accepts a strategy function for sort key extraction.

- **Authentication Libraries**: Passport.js uses strategies for different authentication methods.

- **Validation Libraries**: Joi, Yup allow pluggable validation strategies.

- **Compression Libraries**: Most compression libraries (zlib, lz4, zstd) implement the same interface.

- **Payment Gateways**: Payment SDKs often use strategy pattern for different payment methods.

- **Caching**: Cache invalidation strategies (LRU, LFU, FIFO) are classic strategy implementations.

- **Routing/Navigation**: Mapping applications use strategies for different route optimization goals.

## Related Patterns

- **State**: State and Strategy have similar structures but different intents. State allows behavior to change as internal state changes; Strategy allows algorithm selection independent of context state.

- **Decorator**: Decorator changes an object's skin (adds responsibilities); Strategy changes its guts (swaps algorithm).

- **Template Method**: Template Method uses inheritance to vary part of an algorithm; Strategy uses composition to vary the entire algorithm.

- **Factory Method**: Often used to create the appropriate strategy based on configuration or context.

- **Flyweight**: Stateless strategies can be shared as flyweights.

## When NOT to Use

1. **Single algorithm**: If there's only one algorithm and no foreseeable need for alternatives, the pattern adds unnecessary complexity.

```
// Overkill: Strategy for single implementation
interface Hasher {
    method hash(data): string
}
class SHA256Hasher implements Hasher { ... }
// Only one hasher, no variants

// Just use the function directly
method hash(data): string {
    return sha256(data)
}
```

2. **Algorithms rarely change**: If the algorithm is stable and selected at compile time, polymorphism via inheritance may be simpler.

3. **Simple conditional logic**: If algorithm selection is a simple if/else with minimal code, introducing strategy objects is overkill.

```
// Overkill for simple logic
interface DiscountStrategy { method calculate(price): float }
class NoDiscount { ... }
class TenPercentOff { ... }
class TwentyPercentOff { ... }

// Just use simple conditionals
method applyDiscount(price, discountCode) {
    switch discountCode {
        case "10OFF": return price * 0.9
        case "20OFF": return price * 0.8
        default: return price
    }
}
```

4. **When algorithms need deep context access**: If strategies need extensive access to context internals, the separation may create awkward interfaces.

5. **Performance-critical paths with strategy overhead**: In extremely hot paths, the virtual dispatch overhead of strategy calls may be unacceptable.

6. **When clients shouldn't choose**: If the algorithm should be determined internally, not by clients, the pattern's explicit strategy selection is wrong.

The Strategy pattern excels when you have multiple interchangeable algorithms and want clean separation between algorithm selection and algorithm implementation. For simpler cases, functions, lambdas, or simple conditionals may be more appropriate.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
