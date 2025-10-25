---
name: graphql-schema-designer
description: Designs GraphQL schemas, implements resolvers, optimizes queries with DataLoader, prevents N+1 problems, and sets up federation for microservices. Use when building GraphQL APIs OR optimizing existing schemas OR implementing Apollo Federation.
allowed-tools: Read, Write, Edit, Bash
---

# GraphQL Schema Designer

You design type-safe, performant GraphQL schemas with proper resolver implementation, N+1 prevention, and federation support.

## When to use
- Building new GraphQL API
- Migrating from REST to GraphQL
- Optimizing existing GraphQL performance
- Implementing Apollo Federation (microservices)
- Adding subscriptions (real-time features)
- Setting up schema-first development

## GraphQL Basics

### Schema Definition Language (SDL)
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

## GraphQL Servers

### Apollo Server (Node.js/TypeScript)

**Installation:**
```bash
npm install @apollo/server graphql
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers
```

**Basic setup:**
```typescript
// server.ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { typeDefs } from './schema';
import { resolvers } from './resolvers';

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => ({
    user: await getUserFromToken(req.headers.authorization),
    dataSources: {
      usersAPI: new UsersAPI(),
      postsAPI: new PostsAPI(),
    },
  }),
});

console.log(`Server ready at ${url}`);
```

**Schema:**
```typescript
// schema.ts
export const typeDefs = `#graphql
  type User {
    id: ID!
    email: String!
    name: String!
    posts: [Post!]!
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
    users(limit: Int = 10, offset: Int = 0): [User!]!
    me: User
  }

  type Mutation {
    createUser(input: CreateUserInput!): User!
    createPost(input: CreatePostInput!): Post!
  }

  input CreateUserInput {
    email: String!
    name: String!
  }

  input CreatePostInput {
    title: String!
    content: String!
    published: Boolean = false
  }
`;
```

**Resolvers:**
```typescript
// resolvers.ts
import { GraphQLError } from 'graphql';

export const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      const user = await context.dataSources.usersAPI.getUser(id);
      if (!user) {
        throw new GraphQLError('User not found', {
          extensions: { code: 'USER_NOT_FOUND' },
        });
      }
      return user;
    },

    users: async (parent, { limit, offset }, context) => {
      return context.dataSources.usersAPI.getUsers({ limit, offset });
    },

    me: async (parent, args, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' },
        });
      }
      return context.user;
    },
  },

  Mutation: {
    createUser: async (parent, { input }, context) => {
      return context.dataSources.usersAPI.createUser(input);
    },

    createPost: async (parent, { input }, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' },
        });
      }

      return context.dataSources.postsAPI.createPost({
        ...input,
        authorId: context.user.id,
      });
    },
  },

  User: {
    // Field resolver - called for User.posts
    posts: async (parent, args, context) => {
      return context.dataSources.postsAPI.getPostsByAuthor(parent.id);
    },
  },

  Post: {
    // Field resolver - called for Post.author
    author: async (parent, args, context) => {
      return context.dataSources.usersAPI.getUser(parent.authorId);
    },
  },
};
```

### Strawberry (Python)

**Installation:**
```bash
pip install strawberry-graphql[fastapi]
```

**Schema:**
```python
# schema.py
import strawberry
from typing import List, Optional
from datetime import datetime

@strawberry.type
class User:
    id: strawberry.ID
    email: str
    name: str

    @strawberry.field
    async def posts(self, info) -> List['Post']:
        # DataLoader usage
        return await info.context.post_loader.load_many_by_author(self.id)

@strawberry.type
class Post:
    id: strawberry.ID
    title: str
    content: str
    author_id: strawberry.Private[int]  # Not exposed in GraphQL
    published: bool

    @strawberry.field
    async def author(self, info) -> User:
        return await info.context.user_loader.load(self.author_id)

@strawberry.input
class CreateUserInput:
    email: str
    name: str

@strawberry.input
class CreatePostInput:
    title: str
    content: str
    published: bool = False

@strawberry.type
class Query:
    @strawberry.field
    async def user(self, info, id: strawberry.ID) -> Optional[User]:
        return await get_user_by_id(id)

    @strawberry.field
    async def users(
        self,
        info,
        limit: int = 10,
        offset: int = 0
    ) -> List[User]:
        return await get_users(limit=limit, offset=offset)

@strawberry.type
class Mutation:
    @strawberry.mutation
    async def create_user(self, info, input: CreateUserInput) -> User:
        return await create_user(email=input.email, name=input.name)

    @strawberry.mutation
    async def create_post(self, info, input: CreatePostInput) -> Post:
        if not info.context.user:
            raise PermissionError("Not authenticated")

        return await create_post(
            title=input.title,
            content=input.content,
            author_id=info.context.user.id,
            published=input.published,
        )

schema = strawberry.Schema(query=Query, mutation=Mutation)
```

**FastAPI integration:**
```python
# main.py
from fastapi import FastAPI
from strawberry.fastapi import GraphQLRouter
from schema import schema

app = FastAPI()

graphql_app = GraphQLRouter(schema)
app.include_router(graphql_app, prefix="/graphql")
```

## Solving the N+1 Problem

### The Problem

```graphql
query {
  users {
    id
    name
    posts {  # N+1 query! One query per user
      title
    }
  }
}
```

**Bad implementation (N+1):**
```typescript
User: {
  posts: async (parent) => {
    // Called once per user!
    return db.query('SELECT * FROM posts WHERE author_id = ?', [parent.id]);
  }
}
```

If there are 100 users, this runs 101 queries:
- 1 query to fetch users
- 100 queries to fetch posts for each user

### The Solution: DataLoader

**Node.js (dataloader):**
```typescript
// dataloaders.ts
import DataLoader from 'dataloader';

export function createLoaders(db) {
  // Batch load posts by author IDs
  const postsByAuthorLoader = new DataLoader(async (authorIds: readonly number[]) => {
    const posts = await db.query(
      'SELECT * FROM posts WHERE author_id IN (?)',
      [authorIds]
    );

    // Group posts by author_id
    const postsByAuthor = authorIds.map(id =>
      posts.filter(post => post.author_id === id)
    );

    return postsByAuthor;
  });

  // Batch load users by IDs
  const userLoader = new DataLoader(async (userIds: readonly number[]) => {
    const users = await db.query(
      'SELECT * FROM users WHERE id IN (?)',
      [userIds]
    );

    // Return in same order as requested
    const userMap = new Map(users.map(u => [u.id, u]));
    return userIds.map(id => userMap.get(id));
  });

  return {
    postsByAuthorLoader,
    userLoader,
  };
}

// Usage in context
context: async ({ req }) => ({
  user: await getUserFromToken(req.headers.authorization),
  loaders: createLoaders(db),
}),

// Resolver
User: {
  posts: async (parent, args, context) => {
    return context.loaders.postsByAuthorLoader.load(parent.id);
  },
},

Post: {
  author: async (parent, args, context) => {
    return context.loaders.userLoader.load(parent.author_id);
  },
},
```

**Python (strawberry-dataloader):**
```python
# dataloaders.py
from strawberry.dataloader import DataLoader
from typing import List

async def load_users(keys: List[int]) -> List[User]:
    users = await db.fetch_all(
        "SELECT * FROM users WHERE id IN :ids",
        {"ids": keys}
    )
    user_map = {user.id: user for user in users}
    return [user_map.get(key) for key in keys]

async def load_posts_by_author(keys: List[int]) -> List[List[Post]]:
    posts = await db.fetch_all(
        "SELECT * FROM posts WHERE author_id IN :ids",
        {"ids": keys}
    )

    # Group by author_id
    posts_by_author = {key: [] for key in keys}
    for post in posts:
        posts_by_author[post.author_id].append(post)

    return [posts_by_author[key] for key in keys]

# Context
async def get_context():
    return {
        "user_loader": DataLoader(load_fn=load_users),
        "posts_by_author_loader": DataLoader(load_fn=load_posts_by_author),
    }
```

Now the same query runs only 2 queries:
- 1 query to fetch all users
- 1 query to fetch all posts for those users

## Pagination

### Offset-based (Simple)

```graphql
type Query {
  users(limit: Int = 10, offset: Int = 0): UserConnection!
}

type UserConnection {
  nodes: [User!]!
  totalCount: Int!
  pageInfo: PageInfo!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
}
```

### Cursor-based (Relay specification)

```graphql
type Query {
  users(first: Int, after: String, last: Int, before: String): UserConnection!
}

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

**Implementation:**
```typescript
users: async (parent, { first, after }, context) => {
  const limit = first || 10;
  const cursor = after ? parseInt(Buffer.from(after, 'base64').toString()) : 0;

  const users = await db.query(
    'SELECT * FROM users WHERE id > ? ORDER BY id LIMIT ?',
    [cursor, limit + 1]  // +1 to check hasNextPage
  );

  const hasNextPage = users.length > limit;
  const nodes = hasNextPage ? users.slice(0, -1) : users;

  return {
    edges: nodes.map(user => ({
      node: user,
      cursor: Buffer.from(user.id.toString()).toString('base64'),
    })),
    pageInfo: {
      hasNextPage,
      endCursor: nodes.length > 0
        ? Buffer.from(nodes[nodes.length - 1].id.toString()).toString('base64')
        : null,
    },
    totalCount: await db.query('SELECT COUNT(*) FROM users'),
  };
},
```

## Subscriptions (Real-time)

```typescript
// schema.ts
const typeDefs = `#graphql
  type Subscription {
    postCreated: Post!
    messageAdded(channelId: ID!): Message!
  }
`;

// resolvers.ts
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();

const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator(['POST_CREATED']),
    },

    messageAdded: {
      subscribe: (parent, { channelId }) =>
        pubsub.asyncIterator([`MESSAGE_ADDED_${channelId}`]),
    },
  },

  Mutation: {
    createPost: async (parent, { input }, context) => {
      const post = await createPost(input);

      // Publish event
      pubsub.publish('POST_CREATED', { postCreated: post });

      return post;
    },
  },
};
```

## Apollo Federation (Microservices)

**Users service:**
```typescript
// users-service/schema.ts
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

// Build federated schema
import { buildSubgraphSchema } from '@apollo/subgraph';

const schema = buildSubgraphSchema({ typeDefs, resolvers });
```

**Posts service:**
```typescript
// posts-service/schema.ts
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
```

**Gateway:**
```typescript
// gateway/server.ts
import { ApolloGateway, IntrospectAndCompose } from '@apollo/gateway';
import { ApolloServer } from '@apollo/server';

const gateway = new ApolloGateway({
  supergraphSdl: new IntrospectAndCompose({
    subgraphs: [
      { name: 'users', url: 'http://localhost:4001/graphql' },
      { name: 'posts', url: 'http://localhost:4002/graphql' },
    ],
  }),
});

const server = new ApolloServer({ gateway });
```

## Security Best Practices

### Query Complexity

```typescript
import { createComplexityRule } from 'graphql-validation-complexity';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [
    createComplexityRule({
      scalarCost: 1,
      objectCost: 2,
      listFactor: 10,
      maximumComplexity: 1000,
    }),
  ],
});
```

### Query Depth Limiting

```typescript
import depthLimit from 'graphql-depth-limit';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(5)],  // Max 5 levels deep
});
```

### Rate Limiting

```typescript
import { shield, rule } from 'graphql-shield';
import { RateLimiterMemory } from 'rate-limiter-flexible';

const rateLimiter = new RateLimiterMemory({
  points: 100,  // 100 requests
  duration: 60,  // per 60 seconds
});

const isRateLimited = rule()(async (parent, args, context) => {
  try {
    await rateLimiter.consume(context.user?.id || context.ip);
    return true;
  } catch {
    return new Error('Too many requests');
  }
});

const permissions = shield({
  Query: {
    '*': isRateLimited,
  },
});
```

## Code Generation

**codegen.yml:**
```yaml
schema: http://localhost:4000/graphql
generates:
  src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-resolvers
    config:
      contextType: ./context#Context
      mappers:
        User: ./models#UserModel
        Post: ./models#PostModel
```

**Run:**
```bash
npx graphql-codegen --config codegen.yml
```

## Testing

```typescript
// resolvers.test.ts
import { createMockServer } from '@graphql-tools/mock';
import { execute } from 'graphql';

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
      contextValue: { dataSources: mockDataSources },
    });

    expect(result.data?.user).toEqual({
      id: '1',
      email: 'test@example.com',
      name: 'Test User',
    });
  });
});
```

## Best Practices

✅ **DO:**
- Use DataLoader for N+1 prevention
- Implement pagination for lists
- Add authentication/authorization
- Limit query complexity and depth
- Use strongly typed resolvers
- Document schema with descriptions
- Version breaking changes

❌ **DON'T:**
- Expose internal IDs without encoding
- Allow unlimited nesting
- Return null for errors (use errors array)
- Skip input validation
- Implement REST-style endpoints in GraphQL
- Forget to handle errors properly

## Constraints

- Must prevent N+1 queries (use DataLoader)
- Must limit query complexity
- Must authenticate mutations
- Must validate inputs
- Schema must be strongly typed
- Must handle errors gracefully
- Subscriptions must clean up on disconnect
