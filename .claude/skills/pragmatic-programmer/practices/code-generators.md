# Code Generators

> "Write code that writes code."
> — David Thomas & Andrew Hunt

Code generators are programs that create source code, configuration files, or other artifacts from higher-level specifications. Instead of writing repetitive code by hand, pragmatic programmers build tools that generate it consistently and correctly.

## Types of Code Generators

### Passive Generators

**Definition**: One-time code production tools that generate output once, which developers then own and modify.

**Characteristics**:
- Run once at project setup or when adding new components
- Generated code becomes part of the codebase
- Developers customize and maintain generated output
- No ongoing synchronization needed

**Use Cases**:
- Project scaffolding and initial structure
- Boilerplate class creation
- Database migration file templates
- Test file skeletons

```pseudocode
COMMAND:
  generate controller User --actions index show create

GENERATES (one time):
  FILE: controllers/user_controller.ext
    FUNCTION index():
      // TODO: implement index logic
    END

    FUNCTION show(id):
      // TODO: implement show logic
    END

    FUNCTION create(data):
      // TODO: implement create logic
    END
  END FILE

DEVELOPER then customizes this code directly
```

### Active Generators

**Definition**: Ongoing code production tools that regenerate output whenever specifications change.

**Characteristics**:
- Run repeatedly as specs evolve
- Generated code should not be manually edited
- Changes made only to source specifications
- Maintains perfect synchronization

**Use Cases**:
- Database ORM models from schema
- API clients from OpenAPI specifications
- Language bindings from IDL files
- Documentation from code annotations

```pseudocode
SCHEMA FILE: database/schema.def
  TABLE users:
    FIELD id: INTEGER PRIMARY_KEY
    FIELD username: STRING UNIQUE NOT_NULL
    FIELD email: STRING NOT_NULL
    FIELD created_at: TIMESTAMP DEFAULT_NOW
  END TABLE
END SCHEMA

GENERATOR runs on schema change:
  READ schema.def
  FOR EACH table IN schema:
    GENERATE model class with:
      - Properties for each field
      - Type validation
      - Database mapping
      - Query builders
  END FOR

GENERATES: models/user.ext (DO NOT EDIT - regenerated from schema)
```

## When to Use Code Generation

| Situation | Generate? | Why |
|-----------|-----------|-----|
| Repeating same pattern 3+ times | Yes | DRY principle - eliminate repetition |
| Data structure has canonical source | Yes | Single source of truth ensures consistency |
| Multiple representations needed | Yes | Schema → models, API, docs, tests |
| Simple templating suffices | Maybe | Balance complexity vs. benefit |
| Logic is truly unique | No | Generation adds no value |
| Pattern varies significantly | No | Templates become too complex |

## Template-Based Generation

Template systems replace placeholders with actual values.

```pseudocode
TEMPLATE: crud_service.template
  CLASS {{EntityName}}Service:
    PRIVATE repository: {{EntityName}}Repository

    FUNCTION get_all():
      RETURN repository.find_all()
    END

    FUNCTION get_by_id(id):
      result = repository.find(id)
      IF result IS NULL:
        THROW NotFoundException("{{EntityName}} not found")
      END
      RETURN result
    END

    FUNCTION create(data):
      entity = NEW {{EntityName}}(data)
      VALIDATE entity
      RETURN repository.save(entity)
    END
  END CLASS
END TEMPLATE

GENERATOR:
  INPUT: entity_name = "Product"
  LOAD template FROM "crud_service.template"

  replacements = {
    "{{EntityName}}": entity_name
  }

  output = APPLY replacements TO template
  WRITE output TO "services/product_service.ext"
END GENERATOR
```

## Schema-Driven Generation

More sophisticated approach using structured specifications.

```pseudocode
SCHEMA: api_spec.schema
  ENTITY Product:
    FIELDS:
      - id: INTEGER (primary_key, auto_increment)
      - name: STRING (required, max_length: 200)
      - price: DECIMAL (required, min: 0)
      - category_id: INTEGER (foreign_key: Category)

    OPERATIONS:
      - list (public)
      - get (public)
      - create (authenticated, role: admin)
      - update (authenticated, role: admin)
      - delete (authenticated, role: admin)
  END ENTITY
END SCHEMA

GENERATOR SUITE:
  FUNCTION generate_all(schema):
    entities = PARSE schema

    FOR EACH entity IN entities:
      generate_model(entity)
      generate_repository(entity)
      generate_service(entity)
      generate_controller(entity)
      generate_routes(entity)
      generate_validators(entity)
      generate_tests(entity)
      generate_api_docs(entity)
    END FOR
  END FUNCTION
END GENERATOR
```

## Keeping Generated Code in Sync

### Protected Regions Pattern

Allow manual customization within generated files using protected blocks.

```pseudocode
TEMPLATE with protected regions:
  CLASS {{EntityName}}Service:
    // GENERATED: DO NOT EDIT ABOVE THIS LINE

    // CUSTOM CODE BEGIN: additional_properties
    // Add custom properties here - preserved across regeneration
    // CUSTOM CODE END: additional_properties

    FUNCTION get_all():
      RETURN repository.find_all()
    END

    // CUSTOM CODE BEGIN: custom_methods
    // Add custom methods here - preserved across regeneration
    // CUSTOM CODE END: custom_methods
  END CLASS

REGENERATION LOGIC:
  FUNCTION regenerate_with_preservation(template, output_file):
    IF output_file EXISTS:
      old_content = READ output_file
      custom_blocks = EXTRACT_CUSTOM_BLOCKS(old_content)
    ELSE:
      custom_blocks = EMPTY
    END IF

    new_content = GENERATE_FROM_TEMPLATE(template)
    final_content = MERGE_CUSTOM_BLOCKS(new_content, custom_blocks)

    WRITE final_content TO output_file
  END FUNCTION
END REGENERATION
```

### Separation Strategy

Keep generated and custom code completely separate.

```pseudocode
GENERATED FILE: models/product.generated.ext
  // AUTO-GENERATED - DO NOT EDIT
  CLASS ProductGenerated:
    PROPERTY id: Integer
    PROPERTY name: String
    PROPERTY price: Decimal

    FUNCTION basic_validate():
      ASSERT name IS_NOT_NULL
      ASSERT price >= 0
    END
  END CLASS

CUSTOM FILE: models/product.ext
  IMPORT ProductGenerated

  CLASS Product EXTENDS ProductGenerated:
    // Custom business logic
    FUNCTION calculate_discounted_price(discount_percent):
      RETURN this.price * (1 - discount_percent / 100)
    END

    FUNCTION validate():
      super.basic_validate()

      // Additional custom validation
      IF this.name.length < 3:
        THROW ValidationError("Name too short")
      END IF
    END
  END CLASS
```

## Guidelines for Effective Generators

| Guideline | Rationale |
|-----------|-----------|
| **Make generators obvious** | Clear markers prevent accidental edits |
| **Version your schemas** | Schema changes are breaking changes |
| **Test the generator** | Generators are code too |
| **Provide escape hatches** | Protected regions or extension points |
| **Document the input format** | Schemas are contracts |
| **Regenerate in CI/CD** | Catch drift between specs and code |
| **Keep it simple** | Complex generators are hard to maintain |

## Summary Table

| Aspect | Passive Generators | Active Generators |
|--------|-------------------|-------------------|
| **Frequency** | One-time | Repeated on change |
| **Output ownership** | Developer | Generator |
| **Manual editing** | Expected | Forbidden |
| **Use case** | Scaffolding, boilerplate | Schema-driven layers |
| **Sync requirement** | None | Continuous |
| **Example** | Project templates | ORM models, API clients |

## Key Principles

1. **Don't Repeat Yourself** - If you're writing similar code repeatedly, generate it
2. **Single Source of Truth** - Derive everything from one authoritative schema
3. **Passive for Kickstarts** - Use passive generation for project setup
4. **Active for Consistency** - Use active generation for schema-driven code
5. **Mark Generated Code** - Always clearly identify what's generated
6. **Test Your Generators** - Generators are production code
7. **Provide Customization** - Protected regions or inheritance for custom logic

---

*Based on concepts from "The Pragmatic Programmer" by David Thomas and Andrew Hunt.*
