# bam.data.graph

GraphQL type generation and query resolution for the Bam framework.

## Overview

`bam.data.graph` provides components for generating GraphQL types and query contexts from CLR types. It uses the GraphQL .NET library to produce strongly-typed `ObjectGraphType` subclasses from Bam data types, along with a query context that wires up field resolvers backed by a `DaoRepository`.

The `GraphQLTypeGenerator` takes a configuration object specifying source types (from an assembly and namespace), generates Handlebars-templated C# source code for each type as a GraphQL type class, and produces a query context schema class. The generated code can either be written to disk as source files or compiled directly into an assembly at runtime using Roslyn.

The `GraphQueryResolver` provides an abstract base class for resolving GraphQL queries against a Bam `IRepository`. It translates GraphQL field resolution contexts into repository queries, logging warnings when multiple results are returned for a top-level query.

**Note**: Several source files are excluded from compilation in the .csproj (`GraphQLTypeGenerator.cs`, `GraphQueryResolver.cs`, `PropertyModel.cs`, `TypeModel.cs`, `SchemaModel.cs`, `SemanticAssemblyInfo.cs`), suggesting this project may be in a transitional state. The namespace used is `Bam.Net.Data.GraphQL` (the older `Bam.Net` naming convention).

## Key Classes

| Class | Description |
|---|---|
| `GraphQLTypeGenerator` | Generates GraphQL type classes and query context source code from CLR types using Handlebars templates and Roslyn compilation. |
| `GraphQueryResolver` | Abstract base `ObjectGraphType` that resolves GraphQL queries against a Bam `IRepository`. |
| `GraphQLGenerationConfig` | Configuration for code generation: source assembly path, source/target namespaces, output directory, and schema name. |
| `TypeModel` | Model representing a CLR type for template rendering: type name, namespace, properties, and query arguments. |
| `PropertyModel` | Model representing a property for template rendering, mapping CLR types to GraphQL types (Int, Float, Bool, Id, String, ListGraphType). |
| `QueryArgumentModel` | Model representing a GraphQL query argument with graph type name and separator for template rendering. |
| `SchemaModel` | Model for the query context template: schema name, data types, using statements, and target namespace. |

## Dependencies

### Project References
- bam.base
- bam.data.objects
- bam.data.repositories
- bam.data.schema
- bam.data
- bam.generators

### Package References
- GraphQL 2.4.0

### Target Framework
- net10.0

### NuGet Package Configuration
- Package ID: bam.data.graph
- Version: 2.0.0
- Authors: Bryan Apellanes
- Company: Three Headz
- Configured for NuGet package generation on build

## Usage Examples

### Generating GraphQL types from configuration
```csharp
var config = new GraphQLGenerationConfig
{
    TypeAssembly = "./MyApp.dll",
    FromNameSpace = "MyApp.Data",
    ToNameSpace = "MyApp.GraphQL",
    SchemaName = "MyAppSchema",
    WriteSourceTo = "./GeneratedGraphQL/"
};

var generator = new GraphQLTypeGenerator(config);
generator.WriteSource(config.WriteSourceTo);
```

### Compiling GraphQL types to an assembly
```csharp
var generator = new GraphQLTypeGenerator(config);
byte[] assemblyBytes;
Assembly graphQLAssembly = generator.CompileAssembly(out assemblyBytes);
```

### Implementing a GraphQL query resolver
```csharp
public class MyQueryResolver : GraphQueryResolver
{
    public MyQueryResolver(IRepository repo) : base(repo)
    {
        Field<CustomerGraphType>(
            "customer",
            arguments: new QueryArguments(
                new QueryArgument<StringGraphType> { Name = "Name" }
            ),
            resolve: ctx => Resolve(ctx)
        );
    }
}
```

## Known Gaps / Not Yet Implemented

- **Excluded source files**: `GraphQLTypeGenerator.cs`, `GraphQueryResolver.cs`, `PropertyModel.cs`, `TypeModel.cs`, `SchemaModel.cs`, and `SemanticAssemblyInfo.cs` are all listed under `<Compile Remove>` in the .csproj. These files exist in the repository but are not compiled, suggesting the project may be undergoing refactoring or migration from the `Bam.Net` namespace to the `Bam` namespace.
- **Legacy namespace**: All source files use the `Bam.Net.Data.GraphQL` namespace rather than the current `Bam.Data.Graph` convention.
- **GraphQL version**: The project uses GraphQL 2.4.0, which is significantly older than current releases. The API surface (e.g., `ResolveFieldContext`, `Dictionary<string, object>` arguments) may not align with newer GraphQL .NET versions.
- **CLR-to-GraphQL type mapping**: The `GetGraphTypeName` method only handles `int`, `float`, `bool`, and `string`. Other types (long, decimal, DateTime, custom types) all fall through to `StringGraphType`.
