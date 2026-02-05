# Composite Solution Example: Caching System

## Problem

Build an efficient caching layer with:
- Fast key-value lookup (O(1) access time)
- LRU (Least Recently Used) eviction policy
- Access control and lazy loading
- Memory efficiency for similar objects

## Solution Stack

### 1. Data Structure Layer (clrs-algorithms)

**Hash Table** for O(1) lookup
**Doubly Linked List** for O(1) LRU operations

```
Hash Table: key → node reference
Doubly Linked List: maintains access order

  [MRU] ⇄ [node] ⇄ [node] ⇄ [node] ⇄ [LRU]
           ↑       ↑       ↑
           |       |       |
    Hash: {k1, k2, k3} → node references
```

**How they combine for LRU cache:**
1. Hash table gives O(1) key lookup
2. Doubly linked list gives O(1) reordering (move to front on access)
3. Doubly linked list gives O(1) eviction (remove from tail)

**Key operations:**
- `get(key)`: Hash lookup + move node to front → O(1)
- `put(key, value)`: Hash insert + add to front + possibly evict tail → O(1)
- `evict()`: Remove tail node + remove from hash → O(1)

### 2. Access Pattern (gof-patterns)

**Proxy Pattern** for access control and lazy loading

```typescript
interface Cache {
  get(key: string): any;
  put(key: string, value: any): void;
}

class CacheProxy implements Cache {
  private realCache: LRUCache;
  private accessControl: AccessControl;

  get(key: string): any {
    // Check permissions BEFORE hitting cache
    if (!this.accessControl.canRead(key)) {
      throw new UnauthorizedError();
    }

    // Lazy load from backing store if cache miss
    let value = this.realCache.get(key);
    if (value === undefined) {
      value = this.loadFromStore(key);
      this.realCache.put(key, value);
    }

    return value;
  }

  put(key: string, value: any): void {
    if (!this.accessControl.canWrite(key)) {
      throw new UnauthorizedError();
    }
    this.realCache.put(key, value);
  }
}
```

**Flyweight Pattern** for sharing common data

```typescript
class CachedObject {
  // Intrinsic state (shared)
  private sharedData: SharedDataFlyweight;

  // Extrinsic state (unique)
  private uniqueId: string;
  private timestamp: number;

  constructor(sharedData: SharedDataFlyweight, id: string) {
    this.sharedData = sharedData; // Reference, not copy
    this.uniqueId = id;
    this.timestamp = Date.now();
  }
}

// Factory manages flyweight sharing
class SharedDataFactory {
  private pool: Map<string, SharedDataFlyweight> = new Map();

  getSharedData(key: string): SharedDataFlyweight {
    if (!this.pool.has(key)) {
      this.pool.set(key, new SharedDataFlyweight(key));
    }
    return this.pool.get(key)!;
  }
}
```

**Example:** 1000 user session objects sharing 10 role configurations
- Without flyweight: 1000 × role_config_size
- With flyweight: 1000 × reference_size + 10 × role_config_size

### 3. Consistency (ddia)

**Invalidation strategies**

**Time-based TTL (Time To Live)**
```typescript
class TTLCache {
  private cache: Map<string, { value: any, expiry: number }>;

  put(key: string, value: any, ttlSeconds: number): void {
    const expiry = Date.now() + (ttlSeconds * 1000);
    this.cache.set(key, { value, expiry });
  }

  get(key: string): any | undefined {
    const entry = this.cache.get(key);
    if (!entry) return undefined;

    if (Date.now() > entry.expiry) {
      this.cache.delete(key); // Lazy eviction
      return undefined;
    }

    return entry.value;
  }
}
```

**Event-driven invalidation**
```typescript
class EventDrivenCache {
  constructor(eventBus: EventBus) {
    eventBus.subscribe('user.updated', (userId) => {
      this.invalidate(`user:${userId}`);
      this.invalidate(`user:${userId}:*`); // Pattern-based
    });
  }
}
```

**Write-through vs Write-behind**

| Strategy | Consistency | Performance | Use Case |
|----------|-------------|-------------|----------|
| Write-through | Strong (sync) | Slower writes | Financial data |
| Write-behind | Eventual (async) | Fast writes | Analytics logs |

```typescript
// Write-through: Strong consistency, slower
async putWriteThrough(key: string, value: any): Promise<void> {
  await this.backingStore.write(key, value); // Wait for DB
  this.cache.put(key, value);
}

// Write-behind: Eventual consistency, faster
async putWriteBehind(key: string, value: any): Promise<void> {
  this.cache.put(key, value);
  this.writeQueue.enqueue({ key, value }); // Async flush
}
```

## Complete Implementation Sketch

```typescript
// 1. Core LRU Cache (CLRS data structures)
class LRUCache<K, V> {
  private capacity: number;
  private cache: Map<K, Node<K, V>>;
  private head: Node<K, V>;
  private tail: Node<K, V>;

  constructor(capacity: number) {
    this.capacity = capacity;
    this.cache = new Map();
    // Sentinel nodes
    this.head = new Node(null, null);
    this.tail = new Node(null, null);
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }

  get(key: K): V | undefined {
    const node = this.cache.get(key);
    if (!node) return undefined;

    this.moveToFront(node);
    return node.value;
  }

  put(key: K, value: V): void {
    let node = this.cache.get(key);

    if (node) {
      node.value = value;
      this.moveToFront(node);
    } else {
      if (this.cache.size >= this.capacity) {
        this.evictLRU();
      }

      node = new Node(key, value);
      this.addToFront(node);
      this.cache.set(key, node);
    }
  }

  private moveToFront(node: Node<K, V>): void {
    this.removeNode(node);
    this.addToFront(node);
  }

  private evictLRU(): void {
    const lru = this.tail.prev!;
    this.removeNode(lru);
    this.cache.delete(lru.key!);
  }

  private removeNode(node: Node<K, V>): void {
    node.prev!.next = node.next;
    node.next!.prev = node.prev;
  }

  private addToFront(node: Node<K, V>): void {
    node.next = this.head.next;
    node.prev = this.head;
    this.head.next!.prev = node;
    this.head.next = node;
  }
}

class Node<K, V> {
  constructor(
    public key: K | null,
    public value: V | null,
    public prev?: Node<K, V>,
    public next?: Node<K, V>
  ) {}
}

// 2. Flyweight for memory efficiency (GoF pattern)
class FlyweightCache<K, V> extends LRUCache<K, V> {
  private flyweightFactory: FlyweightFactory;

  put(key: K, value: any): void {
    // Extract shared data
    const sharedData = this.flyweightFactory.getShared(value.type);
    const lightweight = { ...value, shared: sharedData };
    super.put(key, lightweight as V);
  }
}

// 3. Proxy for access control (GoF pattern)
class CacheProxyWithLazyLoad<K, V> implements Cache<K, V> {
  private realCache: LRUCache<K, V>;
  private accessControl: AccessControl;
  private backingStore: Store<K, V>;

  async get(key: K): Promise<V | undefined> {
    if (!this.accessControl.canRead(key)) {
      throw new UnauthorizedError();
    }

    let value = this.realCache.get(key);

    if (value === undefined) {
      // Lazy load from backing store
      value = await this.backingStore.load(key);
      if (value !== undefined) {
        this.realCache.put(key, value);
      }
    }

    return value;
  }

  async put(key: K, value: V): Promise<void> {
    if (!this.accessControl.canWrite(key)) {
      throw new UnauthorizedError();
    }

    this.realCache.put(key, value);

    // Write-through for consistency (DDIA)
    await this.backingStore.write(key, value);
  }
}

// 4. TTL and invalidation (DDIA consistency)
class TTLCacheWithInvalidation<K, V> extends LRUCache<K, V> {
  private ttls: Map<K, number> = new Map();

  constructor(capacity: number, private eventBus: EventBus) {
    super(capacity);

    // Event-driven invalidation
    eventBus.subscribe('invalidate', (keys: K[]) => {
      keys.forEach(k => this.invalidate(k));
    });
  }

  put(key: K, value: V, ttlSeconds?: number): void {
    super.put(key, value);

    if (ttlSeconds) {
      this.ttls.set(key, Date.now() + (ttlSeconds * 1000));
    }
  }

  get(key: K): V | undefined {
    const expiry = this.ttls.get(key);
    if (expiry && Date.now() > expiry) {
      this.invalidate(key);
      return undefined;
    }

    return super.get(key);
  }

  invalidate(key: K): void {
    this.ttls.delete(key);
    // Invalidate from LRU (would need to expose method)
  }
}
```

## Usage Example

```typescript
// Assemble the composite cache
const eventBus = new EventBus();
const lruCache = new TTLCacheWithInvalidation<string, User>(1000, eventBus);
const flyweightCache = new FlyweightCache(lruCache);
const cache = new CacheProxyWithLazyLoad(
  flyweightCache,
  new AccessControl(),
  new UserStore()
);

// Use it
const user = await cache.get('user:123'); // Access control + lazy load + LRU
cache.put('user:123', updatedUser, 300); // TTL=300s, write-through

// Invalidate on events
eventBus.publish('user.updated', ['user:123']);
```

## Trade-offs

| Decision | Choice | Trade-off |
|----------|--------|-----------|
| **Eviction policy** | LRU | Better than FIFO for temporal locality, but not optimal for all patterns (consider LFU/ARC) |
| **Consistency** | Write-through | Strong consistency but slower writes. Use write-behind for high throughput scenarios |
| **Access control** | Proxy pattern | Adds overhead per access. Consider role-based caching if performance critical |
| **Memory optimization** | Flyweight | Complexity cost. Only beneficial when sharing ratio is high (>10:1) |
| **TTL granularity** | Per-key | More flexible but more memory. Consider global TTL for simpler cases |
| **Hash function** | Built-in Map | Fast for strings. Use custom hash for complex keys with collisions |

## When to Use This Stack

✅ **Good fit:**
- High read-to-write ratio (>10:1)
- Working set fits in memory with headroom
- Access patterns show temporal locality
- Multiple objects share common data
- Need access control per key

❌ **Poor fit:**
- Working set >> available memory (use tiered caching)
- Uniform random access (LRU doesn't help)
- Strong consistency required (use database)
- Write-heavy workload (caching overhead > benefit)

## References

- **CLRS**: Chapter 11 (Hash Tables), Chapter 10.2 (Linked Lists)
- **GoF**: Proxy (207), Flyweight (195)
- **DDIA**: Chapter 3 (Storage), Chapter 5 (Replication)
