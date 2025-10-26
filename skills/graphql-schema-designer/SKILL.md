---
name: graphql-schema-designer
description: Designs GraphQL schemas with type definitions, implements resolvers with TypeScript/Python, optimizes queries using DataLoader to prevent N+1 problems, sets up Apollo Federation for microservices, and implements real-time subscriptions. Use when building new GraphQL APIs, optimizing existing schemas, migrating from REST to GraphQL, implementing Apollo/Strawberry/GraphQL-Yoga, adding real-time features, or preventing query performance issues.
allowed-tools: Read, Write, Edit, Bash
---

# GraphQL Schema Designer

You design type-safe, performant GraphQL schemas with proper resolver implementation, N+1 prevention, and federation support.

## When to use
- Building new GraphQL API
- Migrating from REST to GraphQL
- Optimizing existing GraphQL performance (N+1 problems)
- Implementing Apollo Federation (microservices)
- Adding subscriptions (real-time features)
- Setting up schema-first development

## GraphQL Basics

### Schema Definition Language (SDL)

GraphQL schemas define types, queries, and mutations:

```graphql
type User {
  id: ID!
  email: String!
  name: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  published: Boolean!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
  post(id: ID!): Post
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
}

input CreateUserInput {
  email: String!
  name: String!
}

input UpdateUserInput {
  email: String
  name: String
}
```

**Key concepts:**
- `!` means required (non-nullable)
- `[Post!]!` means required list of required posts
- `input` types for mutations
- `Query` for reading data
- `Mutation` for writing data

### Basic Resolver Pattern

```typescript
const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      return context.db.users.findById(id);
    },

    users: async (parent, { limit, offset }, context) => {
      return context.db.users.findMany({ limit, offset });
    },
  },

  Mutation: {
    createUser: async (parent, { input }, context) => {
      return context.db.users.create(input);
    },
  },

  User: {
    // Field resolver for User.posts
    posts: async (parent, args, context) => {
      // Use DataLoader to prevent N+1!
      return context.loaders.postsByAuthor.load(parent.id);
    },
  },

  Post: {
    // Field resolver for Post.author
    author: async (parent, args, context) => {
      return context.loaders.user.load(parent.authorId);
    },
  },
};
```

**Resolver signature:** `(parent, args, context, info) => result`
- `parent` - Result from parent resolver
- `args` - Arguments passed to field
- `context` - Shared context (db, loaders, user, etc.)
- `info` - Query metadata (rarely used)

## The N+1 Problem (Critical!)

**Problem:** Nested queries can trigger N+1 database queries:

```graphql
query {
  users {        # 1 query
    name
    posts {      # N queries (one per user!)
      title
    }
  }
}
```

**Solution:** Always use DataLoader!

```typescript
// dataloaders.ts
import DataLoader from 'dataloader';

export function createLoaders(db) {
  return {
    user: new DataLoader(async (ids) => {
      const users = await db.query(
        'SELECT * FROM users WHERE id IN (?)',
        [ids]
      );
      const userMap = new Map(users.map(u => [u.id, u]));
      return ids.map(id => userMap.get(id));
    }),

    postsByAuthor: new DataLoader(async (authorIds) => {
      const posts = await db.query(
        'SELECT * FROM posts WHERE author_id IN (?)',
        [authorIds]
      );
      // Group by author_id
      const grouped = authorIds.map(id =>
        posts.filter(p => p.author_id === id)
      );
      return grouped;
    }),
  };
}

// Add to context
context: async () => ({
  loaders: createLoaders(db),
}),
```

**For detailed DataLoader implementation, see:** `examples/n+1-prevention.md`

## Detailed Implementation Guides

For framework-specific implementations, see:

### Server Implementations
- **Apollo Server (TypeScript/Node.js):** `examples/apollo-server.md`
  - Complete setup with auth, data sources, testing
  - Code generation with graphql-codegen

- **Strawberry (Python):** `examples/strawberry-python.md`
  - FastAPI integration
  - Python DataLoader patterns

### Advanced Features
- **N+1 Prevention:** `examples/n+1-prevention.md`
  - Detailed DataLoader patterns
  - Batching strategies
  - Testing DataLoader

- **Apollo Federation:** `examples/federation.md`
  - Microservices architecture
  - Cross-service queries
  - Gateway setup

- **Pagination:** `examples/pagination.md`
  - Offset-based pagination
  - Cursor-based (Relay spec)
  - Implementation patterns

- **Subscriptions:** `examples/subscriptions.md`
  - Real-time updates with WebSockets
  - PubSub patterns
  - Subscription lifecycle

### Security & Best Practices
- **Security:** `reference/security.md`
  - Query complexity limiting
  - Query depth limiting
  - Rate limiting
  - Authentication patterns

## Instructions

When building a GraphQL API:

1. **Design schema first:**
   - Define types based on domain model
   - Create queries for reading
   - Create mutations for writing
   - Add inputs for mutation parameters

2. **Implement resolvers:**
   - Query resolvers fetch data
   - Mutation resolvers modify data
   - Field resolvers for computed/relational fields
   - **Always use DataLoader for relational fields**

3. **Prevent N+1 queries:**
   - Create DataLoaders for each entity type
   - Add loaders to context
   - Use loaders in field resolvers
   - Test that queries are batched

4. **Add security:**
   - Authenticate mutations
   - Authorize field access
   - Limit query complexity
   - Limit query depth
   - Add rate limiting

5. **Add pagination:**
   - Use offset-based for simple cases
   - Use cursor-based for production
   - Always return totalCount
   - Implement hasNextPage

6. **Test thoroughly:**
   - Unit test resolvers
   - Integration test queries/mutations
   - Test error cases
   - Test authentication/authorization

7. **Document schema:**
   - Add descriptions to types
   - Add descriptions to fields
   - Document inputs
   - Version breaking changes

## Common Patterns

### Authentication

```typescript
type Query {
  me: User  # Current authenticated user
}

type Mutation {
  login(email: String!, password: String!): AuthPayload!
}

type AuthPayload {
  token: String!
  user: User!
}
```

### Error Handling

```typescript
import { GraphQLError } from 'graphql';

user: async (parent, { id }, context) => {
  const user = await context.db.users.findById(id);
  if (!user) {
    throw new GraphQLError('User not found', {
      extensions: {
        code: 'USER_NOT_FOUND',
        id,
      },
    });
  }
  return user;
},
```

### Pagination Response

```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

## Best Practices

✅ **DO:**
- Use DataLoader for N+1 prevention (critical!)
- Implement pagination for lists
- Add authentication/authorization
- Limit query complexity and depth
- Use strongly typed resolvers
- Document schema with descriptions
- Version breaking changes
- Handle errors with GraphQLError
- Use input types for mutations
- Add field-level authorization

❌ **DON'T:**
- Query in field resolvers without DataLoader
- Allow unlimited nesting
- Return null for errors (use errors array)
- Skip input validation
- Implement REST-style endpoints in GraphQL
- Forget to handle errors properly
- Expose internal IDs without encoding
- Allow unauthenticated mutations
- Skip pagination on lists
- Create N+1 queries

## Testing

```typescript
describe('User resolvers', () => {
  it('fetches user by ID', async () => {
    const query = `
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          email
          name
        }
      }
    `;

    const result = await execute({
      schema,
      document: parse(query),
      variableValues: { id: '1' },
      contextValue: { db: mockDb, loaders: mockLoaders },
    });

    expect(result.data?.user).toEqual({
      id: '1',
      email: 'test@example.com',
      name: 'Test User',
    });
  });
});
```

## Tools & Resources

**Development:**
- GraphQL Playground / GraphiQL for testing
- Apollo Sandbox for testing federated schemas
- graphql-codegen for TypeScript types

**Monitoring:**
- Apollo Studio for schema analytics
- Query complexity monitoring
- Performance tracing

**Libraries:**
- Node.js: `@apollo/server`, `dataloader`, `graphql`
- Python: `strawberry-graphql`, `ariadne`, `graphene`
- Go: `gqlgen`, `graphql-go`

## Constraints

- Must prevent N+1 queries (use DataLoader)
- Must limit query complexity
- Must authenticate mutations
- Must validate inputs
- Schema must be strongly typed
- Must handle errors gracefully
- Subscriptions must clean up on disconnect
- Must implement pagination for lists
- Must add field-level authorization where needed
