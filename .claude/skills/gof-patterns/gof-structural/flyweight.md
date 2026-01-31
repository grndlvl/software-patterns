# Flyweight

## Intent

Use sharing to support large numbers of fine-grained objects efficiently. Flyweight minimizes memory usage by sharing as much data as possible with similar objects. It is a way to use objects in large numbers when a simple repeated representation would use an unacceptable amount of memory.

## Also Known As

- Cache
- Shared Object Pool

## Motivation

Some applications could benefit from using objects throughout their design, but a naive implementation would be prohibitively expensive. For example, most document editor implementations have text formatting and editing facilities that are modularized to some extent. Object-oriented document editors typically use objects to represent embedded elements like tables and figures. However, they usually stop short of using an object for each character in the document, even though doing so would promote flexibility at the finest levels.

Characters and embedded elements could then be treated uniformly with respect to how they are drawn and formatted. The application could be extended to support new character sets without disturbing other functionality. The document's physical structure could mirror its logical structure.

The problem is that a document may contain hundreds of thousands of character objects, which would consume massive amounts of memory and incur unacceptable runtime overhead. The Flyweight pattern describes how to share objects to allow their use at fine granularities without prohibitive cost.

A flyweight is a shared object that can be used in multiple contexts simultaneously. The flyweight acts as an independent object in each context - it is indistinguishable from an instance of the object that is not shared. Flyweights cannot make assumptions about the context in which they operate. The key concept here is the distinction between intrinsic and extrinsic state.

**Intrinsic state** is stored in the flyweight; it consists of information that is independent of the flyweight's context, thereby making it sharable. **Extrinsic state** depends on and varies with the flyweight's context and therefore cannot be shared. Client objects are responsible for passing extrinsic state to the flyweight when it needs it.

## Applicability

Apply the Flyweight pattern when ALL of the following are true:

- An application uses a large number of objects.

- Storage costs are high because of the sheer quantity of objects.

- Most object state can be made extrinsic (moved outside the object).

- Many groups of objects may be replaced by relatively few shared objects once extrinsic state is removed.

- The application does not depend on object identity. Since flyweight objects may be shared, identity tests will return true for conceptually distinct objects.

## Structure

```
                                    ┌─────────────────────────┐
                                    │    FlyweightFactory     │
                                    ├─────────────────────────┤
                                    │ - flyweights: Map       │
                                    ├─────────────────────────┤
                                    │ + getFlyweight(key)     │
                                    │   if not in map:        │
                                    │     create & store      │
                                    │   return flyweights[key]│
                                    └─────────────────────────┘
                                              │
                                              │ creates/returns
                                              ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                              <<interface>>                                 │
│                                Flyweight                                   │
├───────────────────────────────────────────────────────────────────────────┤
│  + operation(extrinsicState)                                               │
└───────────────────────────────────────────────────────────────────────────┘
                                        △
                                        │
                    ┌───────────────────┴───────────────────┐
                    │                                       │
        ┌───────────────────────┐           ┌───────────────────────────┐
        │  ConcreteFlyweight    │           │  UnsharedConcreteFlyweight│
        ├───────────────────────┤           ├───────────────────────────┤
        │ - intrinsicState      │           │ - allState                │
        ├───────────────────────┤           ├───────────────────────────┤
        │ + operation(          │           │ + operation(              │
        │     extrinsicState)   │           │     extrinsicState)       │
        │   // uses both states │           │   // doesn't share        │
        └───────────────────────┘           └───────────────────────────┘


                    ┌─────────────────────────────┐
                    │           Client            │
                    ├─────────────────────────────┤
                    │ - extrinsicState            │
                    ├─────────────────────────────┤
                    │  // stores/computes         │
                    │  // extrinsic state         │
                    │  // calls flyweight.        │
                    │  // operation(extrinsic)    │
                    └─────────────────────────────┘
```

### State Separation Diagram

```
Before Flyweight:
┌─────────────────────────────────────────────────────────────┐
│                    Character Object                          │
├─────────────────────────────────────────────────────────────┤
│  char: 'A'              ─┐                                   │
│  font: "Times"           │ Intrinsic (same for all 'A's)    │
│  size: 12                │                                   │
│  bold: false            ─┘                                   │
│  x: 100                 ─┐                                   │
│  y: 50                   │ Extrinsic (varies per instance)  │
│  color: red             ─┘                                   │
└─────────────────────────────────────────────────────────────┘

After Flyweight:
┌─────────────────────────┐     ┌─────────────────────────┐
│   Flyweight ('A')       │     │   Context (position)    │
├─────────────────────────┤     ├─────────────────────────┤
│  char: 'A'              │     │  x: 100                 │
│  font: "Times"          │◀────│  y: 50                  │
│  size: 12               │     │  color: red             │
│  bold: false            │     │  flyweight: ref to 'A'  │
└─────────────────────────┘     └─────────────────────────┘
        │                               ┌─────────────────────────┐
        │                               │   Context (position)    │
        │                               ├─────────────────────────┤
        └──────────────────────────────▶│  x: 200                 │
                                        │  y: 50                  │
                                        │  color: blue            │
                                        │  flyweight: ref to 'A'  │
                                        └─────────────────────────┘

1000 'A' characters → 1 Flyweight + 1000 small Context objects
```

## Participants

- **Flyweight**: Declares an interface through which flyweights can receive and act on extrinsic state.

- **ConcreteFlyweight**: Implements the Flyweight interface and stores intrinsic state. A ConcreteFlyweight object must be sharable. Any state it stores must be intrinsic; that is, it must be independent of the ConcreteFlyweight's context.

- **UnsharedConcreteFlyweight**: Not all Flyweight subclasses need to be shared. The Flyweight interface enables sharing but does not enforce it. It is common for UnsharedConcreteFlyweight objects to have ConcreteFlyweight objects as children.

- **FlyweightFactory**: Creates and manages flyweight objects. Ensures that flyweights are shared properly. When a client requests a flyweight, the factory supplies an existing instance or creates one if none exists.

- **Client**: Maintains a reference to flyweight(s). Computes or stores the extrinsic state of flyweight(s).

## Collaborations

1. The client requests a flyweight from the FlyweightFactory.

2. The factory looks up the flyweight in a pool. If found, it returns the existing flyweight; otherwise, it creates a new one, adds it to the pool, and returns it.

3. The client stores or computes extrinsic state externally.

4. When operations are performed, the client passes the extrinsic state to the flyweight.

5. The flyweight uses both its intrinsic state and the passed extrinsic state to perform the operation.

## Consequences

### Benefits

- **Reduced memory consumption**: Flyweights can produce significant storage savings when objects share intrinsic state. The more flyweights are shared, the greater the savings.

- **Reduced object count**: Fewer objects mean less memory overhead for object headers and references.

- **Centralized state management**: Intrinsic state is stored once and managed centrally, making updates easier.

- **Improved cache performance**: Fewer objects can mean better cache utilization as the shared flyweights remain in cache.

### Liabilities

- **Runtime cost for extrinsic state**: Computing or looking up extrinsic state takes time. This cost must be weighed against the memory savings.

- **Increased complexity**: The pattern separates intrinsic and extrinsic state, which can make the code harder to understand and maintain.

- **Thread safety considerations**: Shared flyweights must be thread-safe. Intrinsic state must be immutable, and operations must be careful with extrinsic state.

- **Difficult debugging**: Since many logical objects share the same physical object, debugging becomes more complex.

- **Identity concerns**: Clients cannot rely on object identity for equality checks. Two conceptually different objects may be the same flyweight.

## Implementation

### Implementation Considerations

1. **Removing extrinsic state**: The key to applying Flyweight is deciding what state is intrinsic (sharable) and what is extrinsic (context-dependent). Move as much state as possible to extrinsic.

2. **Managing shared objects**: The FlyweightFactory manages the pool. Consider using weak references if flyweights should be garbage collected when no longer used.

3. **Immutability**: Flyweights must be immutable to be safely shared. Make intrinsic state final/readonly and set only in the constructor.

4. **Factory access**: Make flyweight constructors private and provide access only through the factory to ensure sharing.

5. **Hierarchical flyweights**: Flyweights can form hierarchies. A character flyweight might reference a shared font flyweight.

### Basic Flyweight Implementation

```
// Flyweight interface
interface TreeType
    method draw(canvas: Canvas, x: Integer, y: Integer)
end interface

// Concrete Flyweight - shared tree type
class ConcreteTreeType implements TreeType
    // Intrinsic state - same for all trees of this type
    private name: String
    private color: Color
    private texture: Texture  // Large texture data

    constructor(name: String, color: Color, texturePath: String)
        this.name = name
        this.color = color
        this.texture = loadTexture(texturePath)  // Expensive operation
    end constructor

    method draw(canvas: Canvas, x: Integer, y: Integer)
        // Use intrinsic state (texture, color) with extrinsic state (x, y)
        canvas.drawImage(texture, x, y)
        canvas.setColor(color)
        canvas.drawOverlay(x, y)
    end method

    method getName(): String
        return name
    end method
end class

// Flyweight Factory
class TreeFactory
    private static treeTypes: Map<String, TreeType> = new Map()

    static method getTreeType(name: String, color: Color, texturePath: String): TreeType
        key = name + "_" + color.toString() + "_" + texturePath

        if not treeTypes.containsKey(key)
            // Create new flyweight only if it doesn't exist
            treeTypes.put(key, new ConcreteTreeType(name, color, texturePath))
            print "Created new TreeType: " + name
        end if

        return treeTypes.get(key)
    end method

    static method getTypeCount(): Integer
        return treeTypes.size()
    end method
end class

// Context - stores extrinsic state
class Tree
    // Extrinsic state - unique per tree instance
    private x: Integer
    private y: Integer
    private age: Integer

    // Reference to shared flyweight
    private type: TreeType

    constructor(x: Integer, y: Integer, age: Integer, type: TreeType)
        this.x = x
        this.y = y
        this.age = age
        this.type = type
    end constructor

    method draw(canvas: Canvas)
        // Pass extrinsic state to flyweight
        type.draw(canvas, x, y)
    end method

    method getAge(): Integer
        return age
    end method

    method grow()
        age = age + 1
    end method
end class

// Client - forest that contains many trees
class Forest
    private trees: List<Tree>

    constructor()
        trees = new ArrayList()
    end constructor

    method plantTree(x: Integer, y: Integer, name: String, color: Color, texturePath: String)
        // Get shared flyweight from factory
        type = TreeFactory.getTreeType(name, color, texturePath)

        // Create tree with extrinsic state
        tree = new Tree(x, y, 0, type)
        trees.add(tree)
    end method

    method draw(canvas: Canvas)
        for each tree in trees
            tree.draw(canvas)
        end for
    end method

    method getTreeCount(): Integer
        return trees.size()
    end method

    method getTypeCount(): Integer
        return TreeFactory.getTypeCount()
    end method
end class

// Usage
forest = new Forest()

// Plant 1,000,000 trees of various types
for i = 0 to 1000000
    x = random(0, 1000)
    y = random(0, 1000)

    // Only a few tree types, but many instances
    if random(0, 10) < 3
        forest.plantTree(x, y, "Oak", Color.DARK_GREEN, "oak.png")
    else if random(0, 10) < 6
        forest.plantTree(x, y, "Pine", Color.GREEN, "pine.png")
    else
        forest.plantTree(x, y, "Birch", Color.LIGHT_GREEN, "birch.png")
    end if
end for

print "Trees planted: " + forest.getTreeCount()      // 1,000,000
print "Tree types: " + forest.getTypeCount()         // 3

// Without Flyweight: 1,000,000 full Tree objects with textures
// With Flyweight: 3 TreeType objects + 1,000,000 lightweight Tree contexts
```

### Flyweight with Composite Structure

```
// Text editor with character flyweights
// Flyweight for characters
class CharacterFlyweight
    // Intrinsic state
    private char: Character
    private font: Font
    private fontSize: Integer
    private bold: Boolean
    private italic: Boolean

    constructor(char: Character, font: Font, fontSize: Integer, bold: Boolean, italic: Boolean)
        this.char = char
        this.font = font
        this.fontSize = fontSize
        this.bold = bold
        this.italic = italic
    end constructor

    method render(context: RenderContext, x: Integer, y: Integer, color: Color)
        context.setFont(font)
        context.setFontSize(fontSize)
        context.setBold(bold)
        context.setItalic(italic)
        context.setColor(color)
        context.drawChar(char, x, y)
    end method

    method getWidth(): Integer
        return font.getCharWidth(char, fontSize, bold)
    end method

    method getHeight(): Integer
        return font.getHeight(fontSize)
    end method

    method getChar(): Character
        return char
    end method

    method getKey(): String
        return char + "_" + font.name + "_" + fontSize + "_" + bold + "_" + italic
    end method
end class

// Factory for character flyweights
class CharacterFactory
    private static cache: Map<String, CharacterFlyweight> = new Map()
    private static hits: Integer = 0
    private static misses: Integer = 0

    static method getCharacter(
        char: Character,
        font: Font,
        fontSize: Integer,
        bold: Boolean,
        italic: Boolean
    ): CharacterFlyweight
        key = char + "_" + font.name + "_" + fontSize + "_" + bold + "_" + italic

        if cache.containsKey(key)
            hits = hits + 1
            return cache.get(key)
        end if

        misses = misses + 1
        flyweight = new CharacterFlyweight(char, font, fontSize, bold, italic)
        cache.put(key, flyweight)
        return flyweight
    end method

    static method getCacheStats(): CacheStats
        return new CacheStats(
            size: cache.size(),
            hits: hits,
            misses: misses,
            hitRate: hits / (hits + misses)
        )
    end method

    static method clearCache()
        cache.clear()
        hits = 0
        misses = 0
    end method
end class

// Context - character position in document
class CharacterContext
    // Extrinsic state
    private x: Integer
    private y: Integer
    private color: Color
    private highlighted: Boolean

    // Reference to flyweight
    private flyweight: CharacterFlyweight

    constructor(flyweight: CharacterFlyweight, x: Integer, y: Integer, color: Color)
        this.flyweight = flyweight
        this.x = x
        this.y = y
        this.color = color
        this.highlighted = false
    end constructor

    method render(context: RenderContext)
        displayColor = highlighted ? Color.YELLOW_HIGHLIGHT : color
        flyweight.render(context, x, y, displayColor)
    end method

    method setPosition(x: Integer, y: Integer)
        this.x = x
        this.y = y
    end method

    method setHighlighted(highlighted: Boolean)
        this.highlighted = highlighted
    end method

    method getWidth(): Integer
        return flyweight.getWidth()
    end method

    method getHeight(): Integer
        return flyweight.getHeight()
    end method

    method getChar(): Character
        return flyweight.getChar()
    end method

    method containsPoint(px: Integer, py: Integer): Boolean
        return px >= x and px < x + getWidth() and
               py >= y and py < y + getHeight()
    end method
end class

// Line of text - contains character contexts
class TextLine
    private characters: List<CharacterContext>
    private y: Integer
    private lineHeight: Integer

    constructor(y: Integer)
        this.characters = new ArrayList()
        this.y = y
        this.lineHeight = 0
    end constructor

    method addCharacter(flyweight: CharacterFlyweight, color: Color)
        // Calculate x position
        x = 0
        if not characters.isEmpty()
            lastChar = characters.last()
            x = lastChar.x + lastChar.getWidth()
        end if

        context = new CharacterContext(flyweight, x, y, color)
        characters.add(context)

        // Update line height
        if context.getHeight() > lineHeight
            lineHeight = context.getHeight()
        end if
    end method

    method render(renderContext: RenderContext)
        for each charContext in characters
            charContext.render(renderContext)
        end for
    end method

    method getHeight(): Integer
        return lineHeight
    end method

    method getWidth(): Integer
        if characters.isEmpty()
            return 0
        end if
        lastChar = characters.last()
        return lastChar.x + lastChar.getWidth()
    end method

    method getCharacterAt(x: Integer): CharacterContext
        for each charContext in characters
            if charContext.containsPoint(x, y)
                return charContext
            end if
        end for
        return null
    end method
end class

// Document - contains lines of text
class Document
    private lines: List<TextLine>
    private currentFont: Font
    private currentSize: Integer
    private currentBold: Boolean
    private currentItalic: Boolean
    private currentColor: Color

    constructor()
        lines = new ArrayList()
        lines.add(new TextLine(0))
        currentFont = Font.DEFAULT
        currentSize = 12
        currentBold = false
        currentItalic = false
        currentColor = Color.BLACK
    end constructor

    method setFont(font: Font)
        this.currentFont = font
    end method

    method setFontSize(size: Integer)
        this.currentSize = size
    end method

    method setBold(bold: Boolean)
        this.currentBold = bold
    end method

    method setItalic(italic: Boolean)
        this.currentItalic = italic
    end method

    method setColor(color: Color)
        this.currentColor = color
    end method

    method insertCharacter(char: Character)
        if char == '\n'
            // Start new line
            lastLine = lines.last()
            newY = lastLine.y + lastLine.getHeight()
            lines.add(new TextLine(newY))
        else
            // Get flyweight for this character style
            flyweight = CharacterFactory.getCharacter(
                char,
                currentFont,
                currentSize,
                currentBold,
                currentItalic
            )

            // Add to current line
            currentLine = lines.last()
            currentLine.addCharacter(flyweight, currentColor)
        end if
    end method

    method insertText(text: String)
        for each char in text
            insertCharacter(char)
        end for
    end method

    method render(context: RenderContext)
        for each line in lines
            line.render(context)
        end for
    end method

    method getStats(): DocumentStats
        totalChars = 0
        for each line in lines
            totalChars = totalChars + line.characters.size()
        end for

        cacheStats = CharacterFactory.getCacheStats()

        return new DocumentStats(
            totalCharacters: totalChars,
            flyweightsUsed: cacheStats.size,
            memorySaved: calculateMemorySaved(totalChars, cacheStats.size)
        )
    end method

    private method calculateMemorySaved(chars: Integer, flyweights: Integer): String
        // Estimate memory usage
        // Without flyweight: each char stores font, size, style = ~100 bytes
        // With flyweight: each char stores position, color, reference = ~20 bytes
        withoutFlyweight = chars * 100
        withFlyweight = chars * 20 + flyweights * 100
        saved = withoutFlyweight - withFlyweight
        return formatBytes(saved)
    end method
end class

// Usage
document = new Document()

// Type a large document
document.setFont(Font.TIMES_NEW_ROMAN)
document.setFontSize(12)
document.insertText("Lorem ipsum dolor sit amet, consectetur adipiscing elit. ")

document.setBold(true)
document.insertText("This is bold text. ")

document.setBold(false)
document.setItalic(true)
document.insertText("This is italic text.")

document.setItalic(false)
document.insertCharacter('\n')

// ... more text ...

stats = document.getStats()
print "Total characters: " + stats.totalCharacters
print "Flyweights used: " + stats.flyweightsUsed
print "Memory saved: " + stats.memorySaved
```

## Example

### Game Particle System

A real-world example of a particle system in a game where thousands of particles share common properties.

```
// Flyweight - particle template
class ParticleTemplate
    // Intrinsic state - shared by all particles of this type
    private texture: Texture
    private blendMode: BlendMode
    private baseColor: Color
    private shader: Shader
    private animation: SpriteAnimation

    constructor(config: ParticleConfig)
        this.texture = loadTexture(config.texturePath)
        this.blendMode = config.blendMode
        this.baseColor = config.baseColor
        this.shader = loadShader(config.shaderName)

        if config.animated
            this.animation = new SpriteAnimation(
                texture,
                config.frameWidth,
                config.frameHeight,
                config.frameCount,
                config.frameRate
            )
        end if
    end constructor

    method render(
        renderer: Renderer,
        x: Float,
        y: Float,
        scale: Float,
        rotation: Float,
        alpha: Float,
        colorTint: Color,
        animationTime: Float
    )
        renderer.setBlendMode(blendMode)
        renderer.setShader(shader)

        // Combine base color with tint and alpha
        finalColor = baseColor.multiply(colorTint).withAlpha(alpha)
        renderer.setColor(finalColor)

        // Get current animation frame if animated
        sourceRect = null
        if animation != null
            sourceRect = animation.getFrameRect(animationTime)
        end if

        // Draw particle
        renderer.drawSprite(
            texture,
            x, y,
            scale, scale,
            rotation,
            sourceRect
        )
    end method

    method getTexture(): Texture
        return texture
    end method
end class

// Flyweight Factory
class ParticleTemplateFactory
    private static templates: Map<String, ParticleTemplate> = new Map()

    static method getTemplate(config: ParticleConfig): ParticleTemplate
        key = config.getId()

        if not templates.containsKey(key)
            templates.put(key, new ParticleTemplate(config))
        end if

        return templates.get(key)
    end method

    static method preloadTemplates(configs: List<ParticleConfig>)
        for each config in configs
            getTemplate(config)
        end for
    end method

    static method getMemoryUsage(): Integer
        total = 0
        for each template in templates.values()
            total = total + template.getTexture().getMemorySize()
        end for
        return total
    end method
end class

// Particle - context with extrinsic state
class Particle
    // Extrinsic state - unique per particle
    private x: Float
    private y: Float
    private velocityX: Float
    private velocityY: Float
    private scale: Float
    private rotation: Float
    private rotationSpeed: Float
    private alpha: Float
    private colorTint: Color
    private lifetime: Float
    private age: Float
    private alive: Boolean

    // Reference to flyweight
    private template: ParticleTemplate

    constructor(template: ParticleTemplate, emitConfig: EmitConfig)
        this.template = template
        this.x = emitConfig.x + randomRange(-emitConfig.spread, emitConfig.spread)
        this.y = emitConfig.y + randomRange(-emitConfig.spread, emitConfig.spread)

        // Calculate initial velocity from angle and speed
        angle = emitConfig.angle + randomRange(-emitConfig.angleVariance, emitConfig.angleVariance)
        speed = emitConfig.speed + randomRange(-emitConfig.speedVariance, emitConfig.speedVariance)
        this.velocityX = cos(angle) * speed
        this.velocityY = sin(angle) * speed

        this.scale = emitConfig.startScale + randomRange(-emitConfig.scaleVariance, emitConfig.scaleVariance)
        this.rotation = randomRange(0, 360)
        this.rotationSpeed = randomRange(-emitConfig.rotationSpeed, emitConfig.rotationSpeed)
        this.alpha = emitConfig.startAlpha
        this.colorTint = emitConfig.startColor
        this.lifetime = emitConfig.lifetime + randomRange(-emitConfig.lifetimeVariance, emitConfig.lifetimeVariance)
        this.age = 0
        this.alive = true
    end constructor

    method update(deltaTime: Float, physics: ParticlePhysics)
        if not alive
            return
        end if

        age = age + deltaTime

        if age >= lifetime
            alive = false
            return
        end if

        // Calculate life ratio for interpolation
        lifeRatio = age / lifetime

        // Apply physics
        velocityX = velocityX + physics.gravityX * deltaTime
        velocityY = velocityY + physics.gravityY * deltaTime
        velocityX = velocityX * (1 - physics.drag * deltaTime)
        velocityY = velocityY * (1 - physics.drag * deltaTime)

        // Update position
        x = x + velocityX * deltaTime
        y = y + velocityY * deltaTime

        // Update rotation
        rotation = rotation + rotationSpeed * deltaTime

        // Interpolate alpha (fade out)
        alpha = lerp(physics.startAlpha, physics.endAlpha, lifeRatio)

        // Interpolate scale
        scale = lerp(physics.startScale, physics.endScale, lifeRatio)

        // Interpolate color
        colorTint = lerpColor(physics.startColor, physics.endColor, lifeRatio)
    end method

    method render(renderer: Renderer)
        if not alive
            return
        end if

        // Delegate to flyweight with extrinsic state
        template.render(
            renderer,
            x, y,
            scale,
            rotation,
            alpha,
            colorTint,
            age  // For animation timing
        )
    end method

    method isAlive(): Boolean
        return alive
    end method

    method kill()
        alive = false
    end method
end class

// Particle Emitter - manages particles
class ParticleEmitter
    private template: ParticleTemplate
    private particles: List<Particle>
    private particlePool: ObjectPool<Particle>
    private emitConfig: EmitConfig
    private physics: ParticlePhysics
    private emitAccumulator: Float
    private active: Boolean
    private maxParticles: Integer

    constructor(templateConfig: ParticleConfig, emitConfig: EmitConfig, physics: ParticlePhysics)
        this.template = ParticleTemplateFactory.getTemplate(templateConfig)
        this.particles = new ArrayList()
        this.particlePool = new ObjectPool(() -> new Particle(template, emitConfig))
        this.emitConfig = emitConfig
        this.physics = physics
        this.emitAccumulator = 0
        this.active = true
        this.maxParticles = emitConfig.maxParticles
    end constructor

    method update(deltaTime: Float)
        if active
            // Emit new particles
            emitAccumulator = emitAccumulator + deltaTime
            particlesToEmit = floor(emitAccumulator * emitConfig.emitRate)
            emitAccumulator = emitAccumulator - particlesToEmit / emitConfig.emitRate

            for i = 0 to particlesToEmit
                if particles.size() < maxParticles
                    particle = particlePool.acquire()
                    particle.reset(template, emitConfig)
                    particles.add(particle)
                end if
            end for
        end if

        // Update existing particles
        deadParticles = new ArrayList()
        for each particle in particles
            particle.update(deltaTime, physics)
            if not particle.isAlive()
                deadParticles.add(particle)
            end if
        end for

        // Recycle dead particles
        for each dead in deadParticles
            particles.remove(dead)
            particlePool.release(dead)
        end for
    end method

    method render(renderer: Renderer)
        for each particle in particles
            particle.render(renderer)
        end for
    end method

    method setPosition(x: Float, y: Float)
        emitConfig.x = x
        emitConfig.y = y
    end method

    method start()
        active = true
    end method

    method stop()
        active = false
    end method

    method burst(count: Integer)
        for i = 0 to count
            if particles.size() < maxParticles
                particle = particlePool.acquire()
                particle.reset(template, emitConfig)
                particles.add(particle)
            end if
        end for
    end method

    method getParticleCount(): Integer
        return particles.size()
    end method
end class

// Particle System - manages multiple emitters
class ParticleSystem
    private emitters: Map<String, ParticleEmitter>
    private renderer: Renderer

    constructor(renderer: Renderer)
        this.emitters = new Map()
        this.renderer = renderer
    end constructor

    method createEmitter(
        name: String,
        templateConfig: ParticleConfig,
        emitConfig: EmitConfig,
        physics: ParticlePhysics
    ): ParticleEmitter
        emitter = new ParticleEmitter(templateConfig, emitConfig, physics)
        emitters.put(name, emitter)
        return emitter
    end method

    method getEmitter(name: String): ParticleEmitter
        return emitters.get(name)
    end method

    method update(deltaTime: Float)
        for each emitter in emitters.values()
            emitter.update(deltaTime)
        end for
    end method

    method render()
        for each emitter in emitters.values()
            emitter.render(renderer)
        end for
    end method

    method getStats(): ParticleStats
        totalParticles = 0
        for each emitter in emitters.values()
            totalParticles = totalParticles + emitter.getParticleCount()
        end for

        templateMemory = ParticleTemplateFactory.getMemoryUsage()

        return new ParticleStats(
            totalParticles: totalParticles,
            activeEmitters: emitters.size(),
            templateCount: ParticleTemplateFactory.templates.size(),
            templateMemory: templateMemory,
            estimatedSavings: calculateSavings(totalParticles, templateMemory)
        )
    end method

    private method calculateSavings(particles: Integer, templateMemory: Integer): String
        // Without flyweight: each particle would store texture reference and shader
        // With flyweight: particles only store position, velocity, etc.
        withoutFlyweight = particles * 500  // ~500 bytes per particle with texture data
        withFlyweight = particles * 50 + templateMemory  // ~50 bytes per particle + shared templates
        saved = withoutFlyweight - withFlyweight
        return formatBytes(saved)
    end method
end class

// Usage in a game
game.particleSystem = new ParticleSystem(renderer)

// Preload common particle templates
ParticleTemplateFactory.preloadTemplates([
    new ParticleConfig("fire", "particles/fire.png", BlendMode.ADDITIVE),
    new ParticleConfig("smoke", "particles/smoke.png", BlendMode.ALPHA),
    new ParticleConfig("spark", "particles/spark.png", BlendMode.ADDITIVE),
    new ParticleConfig("dust", "particles/dust.png", BlendMode.ALPHA)
])

// Create fire effect - 1000 particles sharing one template
fireEmitter = game.particleSystem.createEmitter(
    "campfire",
    new ParticleConfig("fire", "particles/fire.png", BlendMode.ADDITIVE),
    new EmitConfig(x: 400, y: 300, emitRate: 50, lifetime: 1.5, maxParticles: 1000),
    new ParticlePhysics(gravityY: -100, startAlpha: 1, endAlpha: 0)
)

// Create explosion effect - burst of sparks
explosionEmitter = game.particleSystem.createEmitter(
    "explosion",
    new ParticleConfig("spark", "particles/spark.png", BlendMode.ADDITIVE),
    new EmitConfig(x: 0, y: 0, spread: 10, maxParticles: 500),
    new ParticlePhysics(gravityY: 200, drag: 2)
)

// Trigger explosion
explosionEmitter.setPosition(enemy.x, enemy.y)
explosionEmitter.burst(200)

// In game loop
method gameLoop(deltaTime: Float)
    game.particleSystem.update(deltaTime)
    game.particleSystem.render()

    // Show stats
    stats = game.particleSystem.getStats()
    print "Particles: " + stats.totalParticles
    print "Templates: " + stats.templateCount
    print "Memory saved: " + stats.estimatedSavings
end method
```

## Known Uses

- **Java String Pool**: Java interns string literals, sharing identical string instances.

- **Integer Cache**: Java caches Integer objects for values -128 to 127 (`Integer.valueOf()`).

- **Symbol Tables**: Compilers use symbol tables that share identifiers, reducing memory for large programs.

- **Font Glyphs**: Text rendering systems share glyph objects for each character in a font.

- **Game Engines**: Unity's particle systems, Unreal's Niagara, and other engines use flyweight for particles, tiles, and sprites.

- **Word Processors**: Character formatting in documents (Microsoft Word, Google Docs) uses flyweight for style information.

- **Web Browsers**: CSS rule objects are shared across elements with the same styles.

- **Database Connection Pools**: Connection objects are shared (though this is also Pool pattern).

- **Icon Libraries**: GUI toolkits share icon images across all buttons/menus using the same icon.

## Related Patterns

- **Composite**: Often combined with Flyweight. Composite's leaf nodes can be implemented as flyweights when there are many leaves with shared state.

- **State and Strategy**: Often implemented as flyweights when they are stateless.

- **Factory Method**: FlyweightFactory uses factory methods to create and manage flyweights.

- **Singleton**: FlyweightFactory is often a Singleton.

- **Object Pool**: Related concept but different purpose. Pool manages reusable objects; Flyweight shares immutable objects.

## When NOT to Use

- **When objects are unique**: If most objects have unique intrinsic state, there is little to share.

- **When memory is not a concern**: If you have few objects or plenty of memory, the added complexity is not worth it.

- **When extrinsic state is expensive**: If computing or storing extrinsic state is more expensive than the memory saved, Flyweight is counterproductive.

- **When identity matters**: If code relies on object identity (`obj1 == obj2`), sharing breaks this assumption.

- **When objects are mutable**: Flyweights must be immutable. If objects need to change their intrinsic state, they cannot be shared.

- **When state separation is unclear**: If you cannot cleanly separate intrinsic from extrinsic state, forcing Flyweight leads to awkward designs.

- **For premature optimization**: Do not apply Flyweight speculatively. Profile first to confirm that memory is actually a problem.

- **When thread safety is complex**: If making the flyweight thread-safe is difficult or expensive, consider alternatives.

---

## Summary

The Flyweight pattern is a powerful optimization for scenarios with many similar objects. Its effectiveness depends on cleanly separating intrinsic (shared) from extrinsic (context-specific) state. The pattern shines when you have thousands or millions of objects where most of the object's data is identical - characters in a document, particles in a game, or tiles in a map. The key tradeoffs are increased complexity in state management versus significant memory savings. Always profile before applying Flyweight to ensure the optimization is warranted.

---

*Based on concepts from "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Helm, Johnson, and Vlissides (Gang of Four), 1994.*
