# Memento

## Intent

Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

## Also Known As

- Token
- Snapshot
- Checkpoint

## Motivation

Consider a graphics editor where users can draw shapes, move them around, change colors, and resize them. Users expect to undo their actions—sometimes many levels deep. When they click "undo," the editor should restore the canvas to exactly how it looked before the last action. But how do you capture and restore the state of complex objects without exposing their private internals?

A naive approach might make all internal state public or provide accessors for every private field. This breaks encapsulation and couples the undo mechanism to the internal representation. If you change how shapes store their data internally, you'd have to update the undo code too.

The Memento pattern solves this elegantly. The shape object itself creates a "memento"—an opaque snapshot of its state. The shape knows its own internals and can both create a memento and restore itself from one. But to everyone else, the memento is just a black box. The undo system (the "caretaker") can store mementos without knowing or caring what's inside them.

This separation is powerful. The shape maintains full control of its encapsulation. The undo system is decoupled from shape internals. You can add new shape types with different internal representations, and as long as they create their own mementos, the undo system works unchanged.

## Applicability

Use the Memento pattern when:

- A snapshot of an object's state must be saved so that it can be restored later.
- A direct interface to obtaining the state would expose implementation details and break encapsulation.
- You need to implement undo/redo functionality.
- You need to implement checkpoints or savepoints (for transactions, games, etc.).
- You want to store historical states for audit trails or time-travel debugging.
- You need to implement rollback functionality in transactions.

## Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌──────────────────────────────────────────────────────────────────┐    │
│    │                         Originator                               │    │
│    ├──────────────────────────────────────────────────────────────────┤    │
│    │ - state: any                                                     │    │
│    ├──────────────────────────────────────────────────────────────────┤    │
│    │ + createMemento(): Memento                                       │    │
│    │ + restore(memento: Memento): void                                │    │
│    │ + setState(state): void                                          │    │
│    │ + getState(): any                                                │    │
│    └──────────────────────────────────────────────────────────────────┘    │
│              │                                     │                        │
│              │ creates                             │ restores from          │
│              ▼                                     ▼                        │
│    ┌──────────────────────────────────────────────────────────────────┐    │
│    │                          Memento                                 │    │
│    ├──────────────────────────────────────────────────────────────────┤    │
│    │ - state: any           (private, only Originator can access)     │    │
│    │ - timestamp: datetime                                            │    │
│    │ - name: string                                                   │    │
│    ├──────────────────────────────────────────────────────────────────┤    │
│    │ + getName(): string    (public metadata)                         │    │
│    │ + getTimestamp(): datetime                                       │    │
│    └──────────────────────────────────────────────────────────────────┘    │
│              △                                                              │
│              │                                                              │
│              │ stores                                                       │
│    ┌─────────┴────────────────────────────────────────────────────────┐    │
│    │                        Caretaker                                  │    │
│    ├──────────────────────────────────────────────────────────────────┤    │
│    │ - mementos: list<Memento>                                        │    │
│    │ - originator: Originator                                         │    │
│    ├──────────────────────────────────────────────────────────────────┤    │
│    │ + save(): void                                                   │    │
│    │ + undo(): void                                                   │    │
│    │ + redo(): void                                                   │    │
│    │ + showHistory(): list<string>                                    │    │
│    └──────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│   Encapsulation Boundaries:                                                │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │  Originator can see:    Memento's full state (private access)      │  │
│   ├─────────────────────────────────────────────────────────────────────┤  │
│   │  Caretaker can see:     Memento's metadata only (name, timestamp)  │  │
│   ├─────────────────────────────────────────────────────────────────────┤  │
│   │  Caretaker CANNOT see:  Memento's internal state                   │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Participants

- **Memento**: Stores the internal state of the Originator object. Protects against access by objects other than the originator. Has two interfaces:
  - A narrow interface for the Caretaker (only metadata like name, timestamp)
  - A wide interface for the Originator (full access to stored state)

- **Originator**: Creates a memento containing a snapshot of its current internal state. Uses the memento to restore its internal state. The originator is the only object that can read and write the memento's state.

- **Caretaker**: Responsible for the memento's safekeeping. Never operates on or examines the contents of a memento. Requests mementos from the originator, holds them, and passes them back when restoration is needed.

## Collaborations

1. A caretaker requests a memento from an originator, holds it for a time, and passes it back to the originator when needed.

2. The originator creates a memento that captures its current state and returns it.

3. When undo/restore is needed, the caretaker gives the memento back to the originator.

4. The originator extracts the state from the memento and restores itself.

5. Mementos are passive—they only store state; they don't operate on it.

## Consequences

### Benefits

1. **Preserves encapsulation boundaries**: The pattern avoids exposing information that only an originator should manage but that must be stored outside the originator.

2. **Simplifies the originator**: Having clients manage the state they ask for simplifies the originator and keeps clients from having to notify the originator when they're done.

3. **Easy undo/redo implementation**: The caretaker maintains a history of mementos, making multi-level undo straightforward.

4. **Supports different snapshot strategies**: Incremental mementos, compressed mementos, or full snapshots can all work with the same interface.

5. **Audit trail capability**: Mementos with timestamps create a natural audit trail of state changes.

### Liabilities

1. **Can be expensive**: If the originator has substantial state, creating and storing mementos can be costly in terms of memory and processing time.

2. **Hidden costs**: In languages without built-in support for varying protection levels, it can be difficult to ensure only the originator accesses the memento's state.

3. **Caretaker doesn't know memento size**: A caretaker might incur large storage costs without knowing how much state is in a memento.

4. **Deep copy requirements**: Mementos must typically store deep copies of the originator's state to prevent aliasing issues.

## Implementation

### Implementation Considerations

1. **Language support for two interfaces**: Ideal implementation has the originator as a friend/nested class of memento so it can access private state. In other languages, you might use package-private access or accept less strict encapsulation.

2. **Storing incremental changes**: For large objects, storing deltas rather than full snapshots can save memory. This trades complexity for efficiency.

3. **Deep vs. shallow copy**: Ensure mementos contain independent copies of mutable state. References to shared mutable objects will cause problems when state is restored.

4. **Memento validation**: Consider adding validation to ensure mementos come from the correct originator and haven't been corrupted.

### Pseudocode: Basic Memento Infrastructure

```
// Memento class with narrow and wide interfaces
class Memento {
    // Private state - only originator should access
    private state: any
    private stateHash: string

    // Public metadata - anyone can access
    private timestamp: datetime
    private name: string

    // Constructor accessible only to Originator (implementation varies by language)
    constructor(state: any, name: string) {
        this.state = deepCopy(state)
        this.stateHash = hash(state)
        this.timestamp = now()
        this.name = name
    }

    // Narrow interface for Caretaker
    method getName(): string {
        return this.name
    }

    method getTimestamp(): datetime {
        return this.timestamp
    }

    // Wide interface for Originator (package-private or friend access)
    internal method getState(): any {
        return deepCopy(this.state)
    }

    internal method getStateHash(): string {
        return this.stateHash
    }
}

// Originator - the object whose state we want to save
class TextDocument {
    private content: string = ""
    private cursorPosition: int = 0
    private selectionStart: int = -1
    private selectionEnd: int = -1
    private formatting: map<Range, Style> = {}

    // Create a memento capturing current state
    method createMemento(name: string = "Snapshot"): Memento {
        state = {
            content: this.content,
            cursorPosition: this.cursorPosition,
            selectionStart: this.selectionStart,
            selectionEnd: this.selectionEnd,
            formatting: deepCopy(this.formatting)
        }
        return new Memento(state, name)
    }

    // Restore state from a memento
    method restore(memento: Memento): void {
        state = memento.getState()  // Uses wide interface
        this.content = state.content
        this.cursorPosition = state.cursorPosition
        this.selectionStart = state.selectionStart
        this.selectionEnd = state.selectionEnd
        this.formatting = state.formatting
    }

    // Document operations
    method insert(text: string): void {
        before = this.content.substring(0, this.cursorPosition)
        after = this.content.substring(this.cursorPosition)
        this.content = before + text + after
        this.cursorPosition = this.cursorPosition + text.length
        this.clearSelection()
    }

    method delete(count: int): void {
        if this.hasSelection() {
            this.deleteSelection()
        } else {
            before = this.content.substring(0, this.cursorPosition)
            after = this.content.substring(this.cursorPosition + count)
            this.content = before + after
        }
    }

    method getContent(): string {
        return this.content
    }

    // ... other document methods
}

// Caretaker - manages memento history
class UndoManager {
    private originator: TextDocument
    private undoStack: list<Memento> = []
    private redoStack: list<Memento> = []
    private maxHistorySize: int = 100

    constructor(originator: TextDocument) {
        this.originator = originator
    }

    method save(actionName: string = "Edit"): void {
        memento = this.originator.createMemento(actionName)

        // Clear redo stack on new action
        this.redoStack.clear()

        // Add to undo stack
        this.undoStack.push(memento)

        // Trim history if too large
        while this.undoStack.size() > this.maxHistorySize {
            this.undoStack.removeFirst()
        }
    }

    method undo(): boolean {
        if this.undoStack.isEmpty() {
            return false
        }

        // Save current state to redo stack
        currentMemento = this.originator.createMemento("Before Undo")
        this.redoStack.push(currentMemento)

        // Restore previous state
        previousMemento = this.undoStack.pop()
        this.originator.restore(previousMemento)

        return true
    }

    method redo(): boolean {
        if this.redoStack.isEmpty() {
            return false
        }

        // Save current state to undo stack
        currentMemento = this.originator.createMemento("Before Redo")
        this.undoStack.push(currentMemento)

        // Restore next state
        nextMemento = this.redoStack.pop()
        this.originator.restore(nextMemento)

        return true
    }

    method canUndo(): boolean {
        return not this.undoStack.isEmpty()
    }

    method canRedo(): boolean {
        return not this.redoStack.isEmpty()
    }

    method getUndoDescription(): string {
        if this.undoStack.isEmpty() {
            return null
        }
        return this.undoStack.peek().getName()
    }

    method getRedoDescription(): string {
        if this.redoStack.isEmpty() {
            return null
        }
        return this.redoStack.peek().getName()
    }

    method getHistory(): list<string> {
        result = []
        for memento in this.undoStack {
            result.add(memento.getTimestamp() + ": " + memento.getName())
        }
        return result
    }

    method clear(): void {
        this.undoStack.clear()
        this.redoStack.clear()
    }
}
```

### Pseudocode: Incremental Memento

```
// For large objects, store deltas instead of full state
interface DeltaMemento {
    method apply(state: any): any       // Apply delta to get new state
    method reverse(state: any): any     // Reverse delta to get previous state
}

class InsertTextDelta implements DeltaMemento {
    private position: int
    private text: string

    constructor(position: int, text: string) {
        this.position = position
        this.text = text
    }

    method apply(content: string): string {
        before = content.substring(0, this.position)
        after = content.substring(this.position)
        return before + this.text + after
    }

    method reverse(content: string): string {
        before = content.substring(0, this.position)
        after = content.substring(this.position + this.text.length)
        return before + after
    }
}

class DeleteTextDelta implements DeltaMemento {
    private position: int
    private deletedText: string

    constructor(position: int, deletedText: string) {
        this.position = position
        this.deletedText = deletedText
    }

    method apply(content: string): string {
        before = content.substring(0, this.position)
        after = content.substring(this.position + this.deletedText.length)
        return before + after
    }

    method reverse(content: string): string {
        before = content.substring(0, this.position)
        after = content.substring(this.position)
        return before + this.deletedText + after
    }
}

// Caretaker for delta-based history
class DeltaUndoManager {
    private initialState: Memento
    private deltas: list<DeltaMemento> = []
    private currentPosition: int = -1  // Points to last applied delta
    private originator: TextDocument

    constructor(originator: TextDocument) {
        this.originator = originator
        this.initialState = originator.createMemento("Initial")
    }

    method recordDelta(delta: DeltaMemento): void {
        // Clear any deltas after current position (invalidated by new change)
        while this.deltas.size() > this.currentPosition + 1 {
            this.deltas.removeLast()
        }

        this.deltas.add(delta)
        this.currentPosition = this.deltas.size() - 1
    }

    method undo(): boolean {
        if this.currentPosition < 0 {
            return false
        }

        delta = this.deltas.get(this.currentPosition)
        currentContent = this.originator.getContent()
        newContent = delta.reverse(currentContent)
        this.originator.setContent(newContent)

        this.currentPosition = this.currentPosition - 1
        return true
    }

    method redo(): boolean {
        if this.currentPosition >= this.deltas.size() - 1 {
            return false
        }

        this.currentPosition = this.currentPosition + 1
        delta = this.deltas.get(this.currentPosition)
        currentContent = this.originator.getContent()
        newContent = delta.apply(currentContent)
        this.originator.setContent(newContent)

        return true
    }
}
```

## Example

A complete example implementing a game save system:

```
// Game state components
class Position {
    public x: float
    public y: float
    public z: float

    constructor(x: float, y: float, z: float) {
        this.x = x
        this.y = y
        this.z = z
    }

    method clone(): Position {
        return new Position(this.x, this.y, this.z)
    }
}

class InventoryItem {
    public id: string
    public name: string
    public quantity: int
    public durability: int

    method clone(): InventoryItem {
        item = new InventoryItem()
        item.id = this.id
        item.name = this.name
        item.quantity = this.quantity
        item.durability = this.durability
        return item
    }
}

class QuestProgress {
    public questId: string
    public stage: int
    public objectives: map<string, boolean>

    method clone(): QuestProgress {
        progress = new QuestProgress()
        progress.questId = this.questId
        progress.stage = this.stage
        progress.objectives = new map(this.objectives)
        return progress
    }
}

// Game save memento
class GameSaveMemento {
    // Private state - only GameState can access
    private playerName: string
    private position: Position
    private health: int
    private maxHealth: int
    private mana: int
    private maxMana: int
    private level: int
    private experience: int
    private gold: int
    private inventory: list<InventoryItem>
    private equippedItems: map<string, string>  // slot -> itemId
    private skills: map<string, int>            // skillId -> level
    private quests: list<QuestProgress>
    private discoveredLocations: set<string>
    private gameTime: float
    private difficulty: string

    // Public metadata
    private saveId: string
    private saveName: string
    private timestamp: datetime
    private playTime: duration
    private thumbnailPath: string

    constructor(state: GameStateData, saveName: string) {
        // Deep copy all state
        this.playerName = state.playerName
        this.position = state.position.clone()
        this.health = state.health
        this.maxHealth = state.maxHealth
        this.mana = state.mana
        this.maxMana = state.maxMana
        this.level = state.level
        this.experience = state.experience
        this.gold = state.gold
        this.inventory = state.inventory.map(item => item.clone())
        this.equippedItems = new map(state.equippedItems)
        this.skills = new map(state.skills)
        this.quests = state.quests.map(q => q.clone())
        this.discoveredLocations = new set(state.discoveredLocations)
        this.gameTime = state.gameTime
        this.difficulty = state.difficulty

        // Set metadata
        this.saveId = generateUUID()
        this.saveName = saveName
        this.timestamp = now()
        this.playTime = state.playTime
        this.thumbnailPath = null
    }

    // Narrow interface for SaveManager
    method getSaveId(): string {
        return this.saveId
    }

    method getSaveName(): string {
        return this.saveName
    }

    method getTimestamp(): datetime {
        return this.timestamp
    }

    method getPlayTime(): duration {
        return this.playTime
    }

    method getPlayerLevel(): int {
        return this.level
    }

    method getLocationDescription(): string {
        return "Level " + this.level + " at " + formatPosition(this.position)
    }

    method getThumbnailPath(): string {
        return this.thumbnailPath
    }

    method setThumbnailPath(path: string): void {
        this.thumbnailPath = path
    }

    // Wide interface for GameState (internal/friend access)
    internal method getFullState(): GameStateData {
        return new GameStateData(
            playerName: this.playerName,
            position: this.position.clone(),
            health: this.health,
            maxHealth: this.maxHealth,
            mana: this.mana,
            maxMana: this.maxMana,
            level: this.level,
            experience: this.experience,
            gold: this.gold,
            inventory: this.inventory.map(item => item.clone()),
            equippedItems: new map(this.equippedItems),
            skills: new map(this.skills),
            quests: this.quests.map(q => q.clone()),
            discoveredLocations: new set(this.discoveredLocations),
            gameTime: this.gameTime,
            difficulty: this.difficulty,
            playTime: this.playTime
        )
    }

    // Serialization for persistent storage
    method serialize(): bytes {
        data = {
            version: 2,
            saveId: this.saveId,
            saveName: this.saveName,
            timestamp: this.timestamp.toISOString(),
            playTime: this.playTime.toSeconds(),
            state: {
                playerName: this.playerName,
                position: [this.position.x, this.position.y, this.position.z],
                health: this.health,
                maxHealth: this.maxHealth,
                mana: this.mana,
                maxMana: this.maxMana,
                level: this.level,
                experience: this.experience,
                gold: this.gold,
                inventory: this.serializeInventory(),
                equippedItems: this.equippedItems,
                skills: this.skills,
                quests: this.serializeQuests(),
                discoveredLocations: this.discoveredLocations.toList(),
                gameTime: this.gameTime,
                difficulty: this.difficulty
            }
        }
        return compress(JSON.stringify(data))
    }

    static method deserialize(data: bytes): GameSaveMemento {
        json = JSON.parse(decompress(data))

        // Version migration if needed
        if json.version < 2 {
            json = migrateFromV1(json)
        }

        // Reconstruct memento
        memento = new GameSaveMemento()
        memento.saveId = json.saveId
        memento.saveName = json.saveName
        memento.timestamp = parseDateTime(json.timestamp)
        memento.playTime = Duration.ofSeconds(json.playTime)
        // ... reconstruct all state fields
        return memento
    }
}

// Originator - the game state
class GameState {
    private playerName: string
    private position: Position
    private health: int
    private maxHealth: int
    private mana: int
    private maxMana: int
    private level: int
    private experience: int
    private gold: int
    private inventory: list<InventoryItem>
    private equippedItems: map<string, string>
    private skills: map<string, int>
    private quests: list<QuestProgress>
    private discoveredLocations: set<string>
    private gameTime: float
    private difficulty: string
    private playTime: duration
    private sessionStartTime: datetime

    constructor() {
        this.sessionStartTime = now()
        this.playTime = Duration.ZERO
    }

    method createSave(saveName: string): GameSaveMemento {
        // Update play time before saving
        this.playTime = this.playTime.plus(now() - this.sessionStartTime)
        this.sessionStartTime = now()

        stateData = new GameStateData(
            playerName: this.playerName,
            position: this.position,
            health: this.health,
            maxHealth: this.maxHealth,
            mana: this.mana,
            maxMana: this.maxMana,
            level: this.level,
            experience: this.experience,
            gold: this.gold,
            inventory: this.inventory,
            equippedItems: this.equippedItems,
            skills: this.skills,
            quests: this.quests,
            discoveredLocations: this.discoveredLocations,
            gameTime: this.gameTime,
            difficulty: this.difficulty,
            playTime: this.playTime
        )

        return new GameSaveMemento(stateData, saveName)
    }

    method loadSave(memento: GameSaveMemento): void {
        stateData = memento.getFullState()

        this.playerName = stateData.playerName
        this.position = stateData.position
        this.health = stateData.health
        this.maxHealth = stateData.maxHealth
        this.mana = stateData.mana
        this.maxMana = stateData.maxMana
        this.level = stateData.level
        this.experience = stateData.experience
        this.gold = stateData.gold
        this.inventory = stateData.inventory
        this.equippedItems = stateData.equippedItems
        this.skills = stateData.skills
        this.quests = stateData.quests
        this.discoveredLocations = stateData.discoveredLocations
        this.gameTime = stateData.gameTime
        this.difficulty = stateData.difficulty
        this.playTime = stateData.playTime
        this.sessionStartTime = now()

        // Trigger any necessary reload events
        this.onStateLoaded()
    }

    private method onStateLoaded(): void {
        // Notify subsystems that state changed
        // Reload textures, spawn enemies, etc.
    }

    // Game methods
    method takeDamage(amount: int): void {
        this.health = max(0, this.health - amount)
    }

    method heal(amount: int): void {
        this.health = min(this.maxHealth, this.health + amount)
    }

    method moveTo(position: Position): void {
        this.position = position
    }

    method addItem(item: InventoryItem): void {
        this.inventory.add(item)
    }

    method addExperience(amount: int): void {
        this.experience = this.experience + amount
        this.checkLevelUp()
    }

    // ... many more game methods
}

// Caretaker - manages save files
class SaveManager {
    private gameState: GameState
    private saveDirectory: string
    private autoSaveInterval: duration
    private maxAutoSaves: int = 3
    private autoSaveCounter: int = 0

    constructor(gameState: GameState, saveDirectory: string) {
        this.gameState = gameState
        this.saveDirectory = saveDirectory
        this.autoSaveInterval = Duration.ofMinutes(5)
    }

    method manualSave(saveName: string): SaveResult {
        try {
            memento = this.gameState.createSave(saveName)

            // Take screenshot for thumbnail
            thumbnailPath = this.captureScreenshot(memento.getSaveId())
            memento.setThumbnailPath(thumbnailPath)

            // Serialize and write to disk
            data = memento.serialize()
            filePath = this.saveDirectory + "/" + memento.getSaveId() + ".save"
            writeFile(filePath, data)

            // Update save index
            this.updateSaveIndex(memento)

            return SaveResult.success(memento.getSaveId())
        } catch error {
            return SaveResult.failure(error.message)
        }
    }

    method autoSave(): SaveResult {
        this.autoSaveCounter = (this.autoSaveCounter + 1) % this.maxAutoSaves
        saveName = "Autosave " + (this.autoSaveCounter + 1)
        return this.manualSave(saveName)
    }

    method quickSave(): SaveResult {
        return this.manualSave("Quicksave")
    }

    method loadSave(saveId: string): LoadResult {
        try {
            filePath = this.saveDirectory + "/" + saveId + ".save"
            data = readFile(filePath)
            memento = GameSaveMemento.deserialize(data)

            this.gameState.loadSave(memento)

            return LoadResult.success()
        } catch error {
            return LoadResult.failure(error.message)
        }
    }

    method quickLoad(): LoadResult {
        // Find most recent quicksave
        saves = this.listSaves()
        quicksave = saves.find(s => s.name == "Quicksave")

        if quicksave is null {
            return LoadResult.failure("No quicksave found")
        }

        return this.loadSave(quicksave.id)
    }

    method listSaves(): list<SaveInfo> {
        index = this.loadSaveIndex()
        return index.saves.sortedByDescending(s => s.timestamp)
    }

    method deleteSave(saveId: string): boolean {
        try {
            filePath = this.saveDirectory + "/" + saveId + ".save"
            deleteFile(filePath)

            // Delete thumbnail
            thumbnailPath = this.saveDirectory + "/thumbnails/" + saveId + ".png"
            if fileExists(thumbnailPath) {
                deleteFile(thumbnailPath)
            }

            // Update index
            this.removeFromSaveIndex(saveId)
            return true
        } catch {
            return false
        }
    }

    method getSaveDetails(saveId: string): SaveDetails {
        filePath = this.saveDirectory + "/" + saveId + ".save"
        data = readFile(filePath)
        memento = GameSaveMemento.deserialize(data)

        return new SaveDetails(
            id: memento.getSaveId(),
            name: memento.getSaveName(),
            timestamp: memento.getTimestamp(),
            playTime: memento.getPlayTime(),
            playerLevel: memento.getPlayerLevel(),
            location: memento.getLocationDescription(),
            thumbnailPath: memento.getThumbnailPath()
        )
    }

    private method captureScreenshot(saveId: string): string {
        thumbnailDir = this.saveDirectory + "/thumbnails"
        ensureDirectoryExists(thumbnailDir)
        path = thumbnailDir + "/" + saveId + ".png"
        captureScreen(path, width: 320, height: 180)
        return path
    }

    private method updateSaveIndex(memento: GameSaveMemento): void {
        // Maintain an index file for fast listing without loading all saves
    }

    private method loadSaveIndex(): SaveIndex {
        // Load or create save index
    }
}

// Checkpoint system for auto-restore on death
class CheckpointManager {
    private gameState: GameState
    private lastCheckpoint: GameSaveMemento = null
    private checkpointLocations: set<string>

    constructor(gameState: GameState) {
        this.gameState = gameState
        this.checkpointLocations = new set()
    }

    method registerCheckpointLocation(locationId: string): void {
        this.checkpointLocations.add(locationId)
    }

    method onPlayerEnterLocation(locationId: string): void {
        if this.checkpointLocations.contains(locationId) {
            this.createCheckpoint("Checkpoint: " + locationId)
        }
    }

    method createCheckpoint(name: string): void {
        this.lastCheckpoint = this.gameState.createSave(name)
        showMessage("Checkpoint reached")
    }

    method hasCheckpoint(): boolean {
        return this.lastCheckpoint is not null
    }

    method restoreCheckpoint(): boolean {
        if this.lastCheckpoint is null {
            return false
        }

        this.gameState.loadSave(this.lastCheckpoint)
        showMessage("Restored to last checkpoint")
        return true
    }

    method onPlayerDeath(): void {
        if this.hasCheckpoint() {
            // Wait a moment, then restore
            delay(2000, () => this.restoreCheckpoint())
        } else {
            showGameOverScreen()
        }
    }
}

// Usage example
function main() {
    gameState = new GameState()
    saveManager = new SaveManager(gameState, "/saves")
    checkpointManager = new CheckpointManager(gameState)

    // Register checkpoint locations
    checkpointManager.registerCheckpointLocation("town_entrance")
    checkpointManager.registerCheckpointLocation("dungeon_entrance")
    checkpointManager.registerCheckpointLocation("boss_room")

    // Simulate gameplay
    gameState.setPlayerName("Hero")
    gameState.moveTo(new Position(100, 0, 50))
    gameState.addExperience(500)
    gameState.addItem(new InventoryItem("sword", "Iron Sword", 1, 100))

    // Enter checkpoint location
    checkpointManager.onPlayerEnterLocation("town_entrance")

    // More gameplay
    gameState.takeDamage(30)
    gameState.addItem(new InventoryItem("potion", "Health Potion", 5, -1))

    // Manual save
    result = saveManager.manualSave("Before Boss Fight")
    print("Saved: " + result.saveId)

    // List saves
    saves = saveManager.listSaves()
    for save in saves {
        print(save.name + " - " + save.timestamp + " - Level " + save.playerLevel)
    }

    // Player dies
    gameState.takeDamage(1000)
    if gameState.health <= 0 {
        checkpointManager.onPlayerDeath()
        // Restores to "Checkpoint: town_entrance"
    }

    // Load a specific save
    saveManager.loadSave(saves.first().id)
}
```

## Known Uses

- **Text Editors**: Virtually all text editors (VSCode, Sublime, vim) use memento for undo/redo.

- **Game Save Systems**: Video games store game state in save files that are essentially mementos.

- **Database Transactions**: The savepoint/rollback mechanism in databases uses memento concepts.

- **Version Control**: Git commits are mementos of repository state (though more sophisticated with deltas).

- **Browser History**: Forward/back navigation stores page state mementos.

- **Photoshop History**: Adobe Photoshop's History panel maintains state snapshots.

- **IDE Refactoring**: When IDEs perform refactoring, they save state to allow rollback if something goes wrong.

- **Form Autosave**: Web forms often save draft state periodically for recovery.

- **Serialization Frameworks**: JSON/XML serializers essentially create mementos of object state.

## Related Patterns

- **Command**: Often used together. Commands execute operations; mementos store the state before execution for undo. Each command can store a memento of affected objects.

- **Iterator**: Can iterate over a history of mementos for time-travel debugging or audit trails.

- **Prototype**: Both involve copying object state. Prototype copies for creating new objects; Memento copies for restoring later.

- **State**: Memento captures state at a point in time; State pattern manages state transitions. Memento can snapshot State objects.

## When NOT to Use

1. **Immutable objects**: If your objects are immutable, you don't need mementos—the objects themselves serve as snapshots.

```
// Unnecessary: Memento for immutable object
immutableDoc = new ImmutableDocument("content")
memento = immutableDoc.createMemento()
// Just keep a reference to immutableDoc instead!

history.add(immutableDoc)  // The document IS the snapshot
```

2. **Objects with trivial state**: For objects with just a few simple fields, direct storage is simpler than the memento pattern.

```
// Overkill: Memento for simple point
class Point {
    x: int
    y: int
}

// Just store x and y directly
undoStack.push({x: point.x, y: point.y})
```

3. **When state is too large**: If mementos are prohibitively expensive, consider incremental snapshots, delta compression, or only storing what changed.

4. **When encapsulation isn't important**: If the object's state is already public or you control all code that accesses it, the encapsulation benefit of memento doesn't apply.

5. **Real-time systems with tight memory constraints**: The memory overhead of maintaining snapshot history may be unacceptable.

6. **When state includes unserializable resources**: Open file handles, network connections, and running threads can't be meaningfully captured in a memento.

```
// Problematic: Can't snapshot these
class ServerConnection {
    private socket: Socket          // Can't serialize an open socket
    private threadPool: ThreadPool  // Can't serialize running threads
    private fileHandle: File        // Can't serialize file handles
}
```

7. **Simple linear undo**: If you only need single-level undo, storing just the previous state is simpler than the full memento pattern infrastructure.

The Memento pattern excels when you need robust undo/redo, checkpointing, or state snapshots while preserving encapsulation. For simpler cases, direct state management may be more appropriate.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
