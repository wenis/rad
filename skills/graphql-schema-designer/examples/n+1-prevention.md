# N+1 Problem and DataLoader

## The N+1 Problem

When resolving nested fields, GraphQL can cause N+1 database queries:

```graphql
query {
  users {        # 1 query to fetch users
    id
    name
    posts {      # N queries (one per user!)
      title
    }
  }
}
```

**Bad implementation:**
```typescript
User: {
  posts: async (parent) => {
    // Called once per user!
    return db.query('SELECT * FROM posts WHERE author_id = ?', [parent.id]);
  }
}
```

If there are 100 users → 101 total queries (1 for users + 100 for posts)

## Solution: DataLoader

DataLoader batches and caches database queries.

### Node.js/TypeScript Implementation

**Install:**
```bash
npm install dataloader
```

**Create DataLoaders:**
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
```

**Add to context:**
```typescript
context: async ({ req }) => ({
  user: await getUserFromToken(req.headers.authorization),
  loaders: createLoaders(db),
}),
```

**Use in resolvers:**
```typescript
User: {
  posts: async (parent, args, context) => {
    // DataLoader batches all these calls!
    return context.loaders.postsByAuthorLoader.load(parent.id);
  },
},

Post: {
  author: async (parent, args, context) => {
    return context.loaders.userLoader.load(parent.author_id);
  },
},
```

**Result:** Same query now runs only 2 queries total:
- 1 query to fetch all users
- 1 batched query to fetch all posts for those users

### Python/Strawberry Implementation

**Install:**
```bash
pip install strawberry-graphql[dataloader]
```

**Create DataLoaders:**
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

**Use in schema:**
```python
@strawberry.type
class User:
    id: strawberry.ID
    email: str
    name: str

    @strawberry.field
    async def posts(self, info) -> List['Post']:
        return await info.context.posts_by_author_loader.load(self.id)

@strawberry.type
class Post:
    id: strawberry.ID
    title: str
    author_id: strawberry.Private[int]

    @strawberry.field
    async def author(self, info) -> User:
        return await info.context.user_loader.load(self.author_id)
```

## Best Practices

✅ **DO:**
- Always use DataLoader for relational fields
- Create one DataLoader per entity type
- Add DataLoaders to context
- Return results in same order as keys

❌ **DON'T:**
- Query in field resolvers without DataLoader
- Create new DataLoader per request
- Skip batching for "small" queries (they add up!)
- Return cached data across users (security issue)

## Testing DataLoader

```typescript
describe('DataLoader batching', () => {
  it('batches user queries', async () => {
    const mockDb = {
      query: jest.fn().mockResolvedValue([
        { id: 1, name: 'User 1' },
        { id: 2, name: 'User 2' },
      ]),
    };

    const loaders = createLoaders(mockDb);

    // Request multiple users
    const [user1, user2] = await Promise.all([
      loaders.userLoader.load(1),
      loaders.userLoader.load(2),
    ]);

    // Should call db.query only ONCE
    expect(mockDb.query).toHaveBeenCalledTimes(1);
    expect(mockDb.query).toHaveBeenCalledWith(
      'SELECT * FROM users WHERE id IN (?)',
      [[1, 2]]
    );
  });
});
```
