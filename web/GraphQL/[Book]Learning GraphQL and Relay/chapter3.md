# Chapter3. The GraphQL Schema

- A GraphQL schema is what we write to represent the capabilities of a GraphQL server.
- The schema defines the types and directives we want the server to support.

## The schema object
### An example schema
```graphql
  const queryType = new GraphQLObjectType({
    name: 'RootQuery',
    fields: {
      hello: {
        // ...
      },
      diceRoll: {
        // ...
      },
      usersCount: {
        // ...
      }
    }
  })
  const mySchema = new GraphQLSchema({
    query: queryType
  });
```
The constructor of the `GraphQLSchema` class expects a configuration object:
```graphql
  class GraphQLSchema {
    constructor(config:
    GraphQLSchemaConfig)
  }

  // the config object can be defined as below
  // here, the object defines those operations

  type GraphQLSchemaConfig = {
    query: GraphQLObjectType;
    mutation?: ?GraphQLObjectType;
  }
```
Note that a `?` after an object property marks that property as optional, and a `?` before a type marks the type with the possibility of a null value

```graphql
class GraphQLObjectType {
  constructor(config:
  GraphQLObjectTypeConfig
  )
}

type GraphQLObjectTypeConfig = {
  name: string;
  description?: ?string;
  fileds: GraphQLFieldConfigMapThunk | GraphQLFieldConfigMap;
  interfaces?: ...;
}

type GraphQLFieldConfigMapThunk = () => 
GraphQLFieldConfigMap;
```

Note: use `thunk` function expression syntax when two types need to refer to each other, or when a type needs to refer to itself in a field.

## The GraphQL type system
- GraphQL schemas in a language-agnostic way: "GraphQL schema language"
- GraphQL is a strong type language
- The most basic components of a GraphQL schema are object types, which just represent a kind of object you can fetch from your service
```graphql
// `Character` is a GraphQL Object Type
type Character {
  name: String!
  appearsIn: [Episode]!
  length(unit: LengthUnit = METER): Float
}
```
- An output type in GraphQL can be one of the following three things:
   Custom type
   Scalar type
   Object Type

- Types of Query and Mutation: Most types in your schema will just be normal object types, but there are two types that are special within a schema: query and mutation
- For example:
```graphql
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```
```graphql
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```
- Scalar Types: GraphQL comes with a set of default scalar types (Int, Float, String, Boolean, ID)

- Enumeration Types:
When the scalar value that we want to represent for a field has a list of possible values in a set, and it can only be one of those values.

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

- Type modifiers: `non-null` and `list` 
```graphql
type Character {
  name: [String!]
  appearsIn: [Episode]!
}
```

- Interface: An Interface is an abstract type that includes a certain set of fields that a type must include to implement the interface.
```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```
- Unions: An Union is also an abstract type
```graphql
union SearchResult = Human | Droid | Starship
{
  search(text: "an") {
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
``` 

## The GraphQL introspective API
- Use **_introspective API_** to ask about the GraphQL schema, all of the types and what directives that schema supports

## Validation rules for the GraphQL executor
- The GraphQL executor will only excute requests that pass all validation rules
- If there are any errors during the validation phase, a list of errors would be returned instead of any response
- For example: if a field excepts a numeric argument and we send it a string instead

## The resolve function and how to use it with promises in queries and mutations
## API versioning in GraphQL
- Use new fields and keep the old fields
- If we want to stop supporting old fields in a schema: deprecate a GraphQL field by adding a `deprecationReason` property in it