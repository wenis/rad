# Apollo Federation - Microservices

Apollo Federation allows you to split your GraphQL schema across multiple services.

## Architecture

```
Gateway (Unified Schema)
  ├── Users Service (users, auth)
  ├── Posts Service (posts, comments)
  └── Analytics Service (metrics, stats)
```

## Users Service

```typescript
// users-service/schema.ts
import { buildSubgraphSchema } from '@apollo/subgraph';

const typeDefs = `#graphql
  type User @key(fields: "id") {
    id: ID!
    email: String!
    name: String!
  }

  extend type Post @key(fields: "id") {
    id: ID! @external
    author: User
  }

  type Query {
    user(id: ID!): User
    users: [User!]!
  }
`;

const resolvers = {
  User: {
    __resolveReference: async (reference) => {
      return getUserById(reference.id);
    },
  },

  Post: {
    author: async (post) => {
      return getUserById(post.authorId);
    },
  },
};

export const schema = buildSubgraphSchema({ typeDefs, resolvers });
```

## Posts Service

```typescript
// posts-service/schema.ts
import { buildSubgraphSchema } from '@apollo/subgraph';

const typeDefs = `#graphql
  type Post @key(fields: "id") {
    id: ID!
    title: String!
    content: String!
    authorId: ID!
  }

  extend type User @key(fields: "id") {
    id: ID! @external
    posts: [Post!]!
  }

  type Query {
    post(id: ID!): Post
    posts: [Post!]!
  }
`;

const resolvers = {
  Post: {
    __resolveReference: async (reference) => {
      return getPostById(reference.id);
    },
  },

  User: {
    posts: async (user) => {
      return getPostsByAuthorId(user.id);
    },
  },
};

export const schema = buildSubgraphSchema({ typeDefs, resolvers });
```

## Gateway

```typescript
// gateway/server.ts
import { ApolloGateway, IntrospectAndCompose } from '@apollo/gateway';
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const gateway = new ApolloGateway({
  supergraphSdl: new IntrospectAndCompose({
    subgraphs: [
      { name: 'users', url: 'http://localhost:4001/graphql' },
      { name: 'posts', url: 'http://localhost:4002/graphql' },
    ],
  }),
});

const server = new ApolloServer({ gateway });

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
});

console.log(`Gateway ready at ${url}`);
```

## Key Directives

- `@key(fields: "id")` - Defines entity that can be referenced across services
- `@external` - Field owned by another service
- `extend type` - Add fields to type from another service
- `__resolveReference` - Resolver for fetching entity by key

## Cross-Service Queries

Users can query across services seamlessly:

```graphql
query {
  user(id: "1") {     # Fetched from users-service
    id
    name
    posts {           # Fetched from posts-service
      title
      content
    }
  }
}
```

The gateway automatically:
1. Sends `user(id: "1")` to users-service
2. Gets back `{ id: "1", name: "..." }`
3. Sends `posts` query to posts-service with `user.id`
4. Combines results

## Best Practices

✅ **DO:**
- Define clear service boundaries
- Use `@key` on shared entities
- Implement `__resolveReference` for all entities
- Version breaking changes carefully

❌ **DON'T:**
- Create circular dependencies between services
- Expose internal IDs without encoding
- Skip authentication in subgraphs
- Forget to handle cross-service errors
