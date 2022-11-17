# GraphQL Pokémon

[GraphQL Pokémon](https://graphql-pokemon2.vercel.app/) という API で初代ポケモンを検索することができる。

- GitHub: [lucasbento/graphql-pokemon](https://github.com/lucasbento/graphql-pokemon)

## Chrome extension

(GraphQL Playground for Chrome)[https://chrome.google.com/webstore/detail/graphql-playground-for-ch/kjhjcgclphafojaeeickcokfbhlegecd?hl=ja] という、GraphQL の API をかんたんに実行できる。

スタンドアロン版もある。

- GitHub: [graphql/graphql-playground](https://github.com/graphql/graphql-playground)

## 実行

### Schema

```gql
pokemons(
    first: Int!
): [Pokemon]


type Pokemon {
    id: ID!
    number: String
    name: String
    weight: PokemonDimension
    height: PokemonDimension
    classification: String
    types: [String]
    resistant: [String]
    attacks: PokemonAttack
    weaknesses: [String]
    fleeRate: Float
    maxCP: Int
    evolutions: [Pokemon]
    evolutionRequirements: PokemonEvolutionRequirement
    maxHP: Int
    image: String
}

pokemon(
    id: String
    name: String
): Pokemon

type Pokemon {
    id: ID!
    number: String
    name: String
    weight: PokemonDimension
    height: PokemonDimension
    classification: String
    types: [String]
    resistant: [String]
    attacks: PokemonAttack
    weaknesses: [String]
    fleeRate: Float
    maxCP: Int
    evolutions: [Pokemon]
    evolutionRequirements: PokemonEvolutionRequirement
    maxHP: Int
    image: String
}
```

### alias

```gql
query showPokemonWithAlias {
  charmeleon: pokemon(name: "Charmeleon") {
    maxHP
    weight {
      minimum
      maximum
    }
    weaknesses
    evolutionRequirements {
      amount
      name
    }
  }

  pokemons(first: 8) {
    id
    name
  }
}
```

結果

```json
{
  "data": {
    "charmeleon": {
      "maxHP": 1557,
      "weight": {
        "minimum": "16.63kg",
        "maximum": "21.38kg"
      },
      "weaknesses": [
        "Water",
        "Ground",
        "Rock"
      ],
      "evolutionRequirements": {
        "amount": 100,
        "name": "Charmander candies"
      }
    },
    "pokemons": [
      {
        "id": "UG9rZW1vbjowMDE=",
        "name": "Bulbasaur"
      },
      {
        "id": "UG9rZW1vbjowMDI=",
        "name": "Ivysaur"
      },
      ...
  }
}
```

### fragment

```gql
query comparisonPokemon {
  leftComparison: pokemon(id: "UG9rZW1vbjowMDE=") {
    ...comparisonFields
  }
  rightComparison: pokemon(id: "UG9rZW1vbjowMDI=") {
    ...comparisonFields
  }
}

fragment comparisonFields on Pokemon {
  name
  classification
  height {
    maximum
  }
}
```

結果

```json
{
  "data": {
    "leftComparison": {
      "name": "Bulbasaur",
      "classification": "Seed Pokémon",
      "height": {
        "maximum": "0.79m"
      }
    },
    "rightComparison": {
      "name": "Ivysaur",
      "classification": "Seed Pokémon",
      "height": {
        "maximum": "1.13m"
      }
    }
  }
}
```

### variable & directive

```gql
# variable と directive について

# default var
query showPokemonWithVariable(
  $name: String = "Squirtle"
  $withFriends: Boolean!
) {
  pokemon(name: $name) {
    # directive
    # @include は 含めるかどうか、@skip は含めないかどうか。
    name # @deprecated(reason: "it's too old.")
    fleeRate @include(if: $withFriends)
  }
}

# {
#   "name": "Squirtle"
# }
```

結果

```json
{
  "error": {
    "errors": [
      {
        "message": "Variable \"$withFriends\" of required type \"Boolean!\" was not provided.",
        "locations": [
          {
            "line": 1,
            "column": 59
          }
        ],
        "stack": "GraphQLError: Variable \"$withFriends\" of required type \"Boolean!\" was not provided.\n    at getVariableValues (/var/task/node_modules/graphql/execution/values.js:72:15)\n    at buildExecutionContext (/var/task/node_modules/graphql/execution/execute.js:186:54)\n    at execute (/var/task/node_modules/graphql/execution/execute.js:109:17)\n    at result (/var/task/node_modules/koa-graphql/dist/index.js:188:46)\n    at new Promise (<anonymous>)\n    at Object.middleware$ (/var/task/node_modules/koa-graphql/dist/index.js:138:20)\n    at tryCatch (/var/task/node_modules/koa-graphql/node_modules/regenerator-runtime/runtime.js:65:40)\n    at Generator.invoke [as _invoke] (/var/task/node_modules/koa-graphql/node_modules/regenerator-runtime/runtime.js:299:22)\n    at Generator.prototype.<computed> [as next] (/var/task/node_modules/koa-graphql/node_modules/regenerator-runtime/runtime.js:117:21)\n    at onFulfilled (/var/task/node_modules/co/index.js:65:19)"
      }
    ]
  }
}
```
