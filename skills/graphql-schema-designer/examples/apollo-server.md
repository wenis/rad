# Apollo Server (Node.js/TypeScript) Implementation

Complete Apollo Server setup with TypeScript, including authentication and data sources.

## Installation

```bash
npm install @apollo/server graphql
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers
```

## Basic Server Setup

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

## Schema Definition

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

## Resolvers

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

## TypeScript Types with Codegen

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
import { execute, parse } from 'graphql';

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
