## Blackbird Database Framework Usage

### Database and Transaction Management
- Create databases: `Blackbird.Database(path: url.path)`
- Write operations: `database.transaction { core in ... }` (async)
- Tables are created automatically when first used - no manual table creation needed

### Model Definition Best Practices
- Models conform to `BlackbirdModel` protocol
- Use `@BlackbirdColumn` property wrapper for database columns
- Define primary key: `static var primaryKey: [BlackbirdColumnKeyPath]`
- Define indexes: `static var indexes: [[BlackbirdColumnKeyPath]]` (replaces manual SQL)
- Use meaningful values as primary keys when possible (e.g., enum raw values)

### Write Operations (within transactions)
```swift
// Save models
try model.writeIsolated(to: database, core: core)

// Delete with conditions - use "matching" not "where"
try Model.deleteIsolated(from: database, core: core, matching: \.$column == value)
```

### Read Operations (async, outside transactions)
```swift
// Basic queries - use "matching" not "where"
let results = try await Model.read(from: database, matching: \.$column == value)

// Sorted queries
let sorted = try await Model.read(from: database, orderBy: .ascending(\.$column))

// Within transactions (isolated) - use "matching" not "where"
let results = try Model.readIsolated(from: database, core: core, matching: condition)
```

### Custom Type Extensions
Custom types need `BlackbirdStorableAsInteger` and `BlackbirdColumnWrappable`:
```swift
extension CustomType: BlackbirdStorableAsInteger, BlackbirdColumnWrappable {
    func unifiedRepresentation() -> Int64 { return Int64(self.rawValue) }
    static func from(unifiedRepresentation: Int64) -> CustomType { ... }
    static func fromValue(_ value: Blackbird.Value) -> CustomType? { ... }
}
```

### Performance Considerations
- Use transactions for batch operations
- Indexes are handled automatically via model declarations
- Primary keys should use natural identifiers when available
