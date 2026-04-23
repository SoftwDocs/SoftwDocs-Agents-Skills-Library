---
name: graphql-api-architect
description: Complete GraphQL API architecture with Apollo Server, schema design, resolvers, subscriptions, authentication, caching, and best practices for production GraphQL APIs.
tags: [graphql, api, apollo, subscriptions, caching, schema-design]
version: 1.0.0
author: SoftwDocs
---

# GraphQL API Architect

## Overview

A comprehensive skill for building production-grade GraphQL APIs. Covers schema design, resolver implementation, subscriptions, authentication, caching, error handling, and performance optimization using Apollo Server and best practices.

## When to Use This Skill

- Building GraphQL APIs from scratch
- Migrating REST APIs to GraphQL
- Implementing real-time features with subscriptions
- Designing complex GraphQL schemas
- Optimizing GraphQL query performance
- Implementing authentication and authorization

## Core Technologies

### Server
- Apollo Server for GraphQL server
- GraphQL Yoga for alternative implementation
- Type-safe schema with TypeScript

### Tools
- GraphQL Code Generator for type generation
- Apollo Client for frontend integration
- DataLoader for batching and caching

### Features
- Subscriptions for real-time updates
- Apollo Federation for microservices
- GraphQL caching strategies

## Implementation Patterns

### Pattern 1: Schema Design with TypeScript

Type-safe GraphQL schema definition:

```typescript
// schema/types.ts
import { gql } from 'apollo-server';

export const typeDefs = gql`
  type User {
    id: ID!
    email: String!
    name: String!
    posts: [Post!]!
    createdAt: DateTime!
    updatedAt: DateTime!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
    comments: [Comment!]!
    publishedAt: DateTime!
    createdAt: DateTime!
    updatedAt: DateTime!
  }

  type Comment {
    id: ID!
    content: String!
    author: User!
    post: Post!
    createdAt: DateTime!
    updatedAt: DateTime!
  }

  type Query {
    me: User
    user(id: ID!): User
    users(limit: Int, offset: Int): [User!]!
    post(id: ID!): Post
    posts(limit: Int, offset: Int): [Post!]!
    searchPosts(query: String!): [Post!]!
  }

  type Mutation {
    login(email: String!, password: String!): AuthPayload!
    register(email: String!, password: String!, name: String!): AuthPayload!
    createPost(title: String!, content: String!): Post!
    updatePost(id: ID!, title: String, content: String): Post!
    deletePost(id: ID!): Boolean!
    createComment(postId: ID!, content: String!): Comment!
  }

  type Subscription {
    postCreated: Post!
    commentAdded(postId: ID!): Comment!
  }

  type AuthPayload {
    token: String!
    user: User!
  }

  scalar DateTime
`;

// schema/resolvers.ts
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();

export const resolvers = {
  Query: {
    me: async (_: any, __: any, { user }: any) => {
      if (!user) throw new Error('Not authenticated');
      return getUserById(user.id);
    },
    user: async (_: any, { id }: { id: string }) => {
      return getUserById(id);
    },
    users: async (_: any, { limit = 10, offset = 0 }: { limit: number; offset: number }) => {
      return getUsers(limit, offset);
    },
    post: async (_: any, { id }: { id: string }) => {
      return getPostById(id);
    },
    posts: async (_: any, { limit = 10, offset = 0 }: { limit: number; offset: number }) => {
      return getPosts(limit, offset);
    },
    searchPosts: async (_: any, { query }: { query: string }) => {
      return searchPosts(query);
    },
  },

  Mutation: {
    login: async (_: any, { email, password }: { email: string; password: string }) => {
      const user = await authenticateUser(email, password);
      const token = generateJWT(user);
      return { token, user };
    },
    register: async (_: any, { email, password, name }: { email: string; password: string; name: string }) => {
      const user = await createUser({ email, password, name });
      const token = generateJWT(user);
      return { token, user };
    },
    createPost: async (_: any, { title, content }: { title: string; content: string }, { user }: any) => {
      if (!user) throw new Error('Not authenticated');
      const post = await createPost({ title, content, authorId: user.id });
      pubsub.publish('POST_CREATED', { postCreated: post });
      return post;
    },
    updatePost: async (_: any, { id, title, content }: { id: string; title?: string; content?: string }, { user }: any) => {
      if (!user) throw new Error('Not authenticated');
      return updatePost(id, { title, content });
    },
    deletePost: async (_: any, { id }: { id: string }, { user }: any) => {
      if (!user) throw new Error('Not authenticated');
      await deletePost(id);
      return true;
    },
    createComment: async (_: any, { postId, content }: { postId: string; content: string }, { user }: any) => {
      if (!user) throw new Error('Not authenticated');
      const comment = await createComment({ postId, content, authorId: user.id });
      pubsub.publish('COMMENT_ADDED', { commentAdded: comment, postId });
      return comment;
    },
  },

  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator(['POST_CREATED']),
    },
    commentAdded: {
      subscribe: (_: any, { postId }: { postId: string }) => {
        return pubsub.asyncIterator([`COMMENT_ADDED_${postId}`]);
      },
    },
  },

  User: {
    posts: async (user: any) => {
      return getPostsByAuthorId(user.id);
    },
  },

  Post: {
    author: async (post: any) => {
      return getUserById(post.authorId);
    },
    comments: async (post: any) => {
      return getCommentsByPostId(post.id);
    },
  },

  Comment: {
    author: async (comment: any) => {
      return getUserById(comment.authorId);
    },
    post: async (comment: any) => {
      return getPostById(comment.postId);
    },
  },
};
```

### Pattern 2: Apollo Server Setup

Production-ready Apollo Server configuration:

```typescript
// server/index.ts
import { ApolloServer } from 'apollo-server-express';
import { ApolloServerPluginDrainHttpServer } from 'apollo-server-core';
import express from 'express';
import http from 'http';
import { typeDefs, resolvers } from './schema';
import { context } from './context';
import { formatError } from './errors';

async function startApolloServer() {
  const app = express();
  const httpServer = http.createServer(app);

  const server = new ApolloServer({
    typeDefs,
    resolvers,
    context,
    formatError,
    plugins: [
      ApolloServerPluginDrainHttpServer({ httpServer }),
    ],
    introspection: process.env.NODE_ENV !== 'production',
    csrfPrevention: true,
    cache: 'bounded',
  });

  await server.start();
  server.applyMiddleware({ app, path: '/graphql' });

  const PORT = process.env.PORT || 4000;
  await new Promise<void>((resolve) => 
    httpServer.listen({ port: PORT }, resolve)
  );

  console.log(`🚀 Server ready at http://localhost:${PORT}${server.graphqlPath}`);
}

startApolloServer();

// server/context.ts
import { verifyToken } from './auth';

export const context = async ({ req }: any) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  let user = null;
  if (token) {
    try {
      user = await verifyToken(token);
    } catch (error) {
      console.error('Invalid token:', error);
    }
  }

  return {
    user,
    req,
  };
};

// server/errors.ts
import { ApolloError, AuthenticationError, ForbiddenError } from 'apollo-server-express';

export const formatError = (error: any) => {
  if (error instanceof AuthenticationError) {
    return new AuthenticationError('You must be logged in');
  }
  
  if (error instanceof ForbiddenError) {
    return new ForbiddenError('You do not have permission');
  }
  
  if (error instanceof ApolloError) {
    return error;
  }
  
  console.error('Unexpected error:', error);
  return new ApolloError('An unexpected error occurred');
};
```

### Pattern 3: DataLoader for N+1 Problem

Batching and caching with DataLoader:

```typescript
// loaders/userLoader.ts
import DataLoader from 'dataloader';

const userLoader = new DataLoader(async (ids: readonly string[]) => {
  const users = await getUsersByIds(ids);
  const userMap = new Map(users.map(user => [user.id, user]));
  return ids.map(id => userMap.get(id));
});

export { userLoader };

// loaders/postLoader.ts
const postLoader = new DataLoader(async (ids: readonly string[]) => {
  const posts = await getPostsByIds(ids);
  const postMap = new Map(posts.map(post => [post.id, post]));
  return ids.map(id => postMap.get(id));
});

export { postLoader };

// Updated resolvers with DataLoader
export const resolvers = {
  User: {
    posts: async (user: any, _: any, { userLoader }: any) => {
      return userLoader.load(user.id);
    },
  },

  Post: {
    author: async (post: any, _: any, { userLoader }: any) => {
      return userLoader.load(post.authorId);
    },
    comments: async (post: any) => {
      return getCommentsByPostId(post.id);
    },
  },
};
```

### Pattern 4: GraphQL Subscriptions

Real-time updates with subscriptions:

```typescript
// server/subscriptions.ts
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();

export const POST_CREATED = 'POST_CREATED';
export const COMMENT_ADDED = 'COMMENT_ADDED';

export { pubsub };

// In resolvers
export const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator([POST_CREATED]),
    },
    commentAdded: {
      subscribe: (_: any, { postId }: { postId: string }) => {
        return pubsub.asyncIterator([`${COMMENT_ADDED}_${postId}`]);
      },
    },
  },
};

// Publishing events
await pubsub.publish(POST_CREATED, { postCreated: newPost });
await pubsub.publish(`${COMMENT_ADDED}_${postId}`, { commentAdded: newComment });

// Client-side subscription
import { gql, useSubscription } from '@apollo/client';

const POST_CREATED_SUBSCRIPTION = gql`
  subscription OnPostCreated {
    postCreated {
      id
      title
      content
      author {
        id
        name
      }
    }
  }
`;

function PostSubscription() {
  const { data, loading, error } = useSubscription(POST_CREATED_SUBSCRIPTION);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h3>New Post Created!</h3>
      <p>{data.postCreated.title}</p>
    </div>
  );
}
```

### Pattern 5: GraphQL Caching

Implementing caching strategies:

```typescript
// server/cache.ts
import { RedisCache } from 'apollo-server-cache-redis';
import { responseCachePlugin } from 'apollo-server-plugin-response-cache';

const cache = new RedisCache({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379'),
});

export const cachePlugin = responseCachePlugin({
  cache,
  sessionId: (requestContext) => {
    const token = requestContext.request.http.headers.get('authorization');
    return token || null;
  },
});

// In Apollo Server configuration
const server = new ApolloServer({
  typeDefs,
  resolvers,
  cache,
  plugins: [cachePlugin],
});

// Cache control in schema
export const typeDefs = gql`
  type Post @cacheControl(maxAge: 300) {
    id: ID!
    title: String!
    content: String!
    author: User!
  }

  type Query {
    post(id: ID!): Post @cacheControl(maxAge: 60, scope: PRIVATE)
    posts: [Post!]! @cacheControl(maxAge: 300)
  }
`;

// Client-side caching with Apollo Client
import { ApolloClient, InMemoryCache } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          posts: {
            keyArgs: ['limit', 'offset'],
            merge(existing = [], incoming) {
              return [...existing, ...incoming];
            },
          },
        },
      },
    },
  }),
});
```

## Authentication & Authorization

```typescript
// server/auth.ts
import jwt from 'jsonwebtoken';

export function generateJWT(user: any): string {
  return jwt.sign(
    { userId: user.id, email: user.email },
    process.env.JWT_SECRET!,
    { expiresIn: '7d' }
  );
}

export async function verifyToken(token: string): Promise<any> {
  try {
    return jwt.verify(token, process.env.JWT_SECRET!);
  } catch (error) {
    throw new AuthenticationError('Invalid token');
  }
}

// Authorization directive
export const typeDefs = gql`
  directive @auth on FIELD_DEFINITION
  directive @admin on FIELD_DEFINITION

  type Query {
    adminData: String @admin
    privateData: String @auth
  }
`;

export const resolvers = {
  Query: {
    adminData: () => 'Admin only data',
    privateData: () => 'Private data',
  },
};

export const directiveResolvers = {
  auth: (next: any, source: any, args: any, ctx: any) => {
    if (!ctx.user) {
      throw new AuthenticationError('You must be logged in');
    }
    return next();
  },
  admin: (next: any, source: any, args: any, ctx: any) => {
    if (!ctx.user || ctx.user.role !== 'admin') {
      throw new ForbiddenError('Admin access required');
    }
    return next();
  },
};
```

## Error Handling

```typescript
// server/errors.ts
import { ApolloError, UserInputError, AuthenticationError, ForbiddenError } from 'apollo-server-export';

export class ValidationError extends UserInputError {
  constructor(message: string, fields: Record<string, string>) {
    super(message);
    this.extensions = { fields };
  }
}

export class NotFoundError extends ApolloError {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`);
    this.extensions = { code: 'NOT_FOUND' };
  }
}

// Usage in resolvers
export const resolvers = {
  Mutation: {
    createPost: async (_: any, { title, content }: { title: string; content: string }, { user }: any) => {
      if (!user) throw new AuthenticationError('Not authenticated');
      
      if (!title || title.length < 3) {
        throw new ValidationError('Title must be at least 3 characters', {
          title: 'Title too short',
        });
      }
      
      return createPost({ title, content, authorId: user.id });
    },
  },
};
```

## Best Practices

### ✅ Do

- Use TypeScript for type safety
- Implement DataLoader for N+1 queries
- Use subscriptions for real-time features
- Implement proper error handling
- Add caching for expensive queries
- Use directives for authorization
- Document your schema
- Implement rate limiting

### ❌ Don't

- Ignore the N+1 problem
- Over-fetch or under-fetch data
- Skip authentication checks
- Use subscriptions for everything
- Forget to cache responses
- Ignore error boundaries
- Skip schema validation
- Forget to log errors

## Resources

- [Apollo Server Documentation](https://www.apollographql.com/docs/apollo-server/)
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/)
- [GraphQL Code Generator](https://www.graphql-code-generator.com/)
- [DataLoader Documentation](https://github.com/graphql/dataloader)
