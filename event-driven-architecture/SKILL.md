---
name: event-driven-architecture
description: Complete event-driven architecture implementation with message queues, event buses, pub/sub patterns, saga patterns, and distributed systems for scalable, decoupled applications.
tags: [event-driven, message-queue, pub-sub, saga, distributed-systems, microservices]
version: 1.0.0
author: SoftwDocs
---

# Event-Driven Architecture

## Overview

A comprehensive skill for building event-driven architectures. Covers message queues, event buses, pub/sub patterns, saga patterns for distributed transactions, event sourcing, and best practices for scalable, decoupled systems.

## When to Use This Skill

- Building microservices architectures
- Implementing asynchronous processing
- Creating decoupled systems
- Handling distributed transactions
- Building real-time data pipelines
- Implementing event sourcing

## Core Technologies

### Message Queues
- RabbitMQ for AMQP messaging
- AWS SQS for queue-based messaging
- Redis Pub/Sub for simple messaging

### Event Streaming
- Apache Kafka for event streaming
- AWS Kinesis for real-time streaming
- NATS for lightweight messaging

### Event Buses
- AWS EventBridge for serverless events
- NATS JetStream for persistence
- RabbitMQ exchanges for routing

## Implementation Patterns

### Pattern 1: RabbitMQ Event Bus

Event bus with RabbitMQ:

```typescript
// event-bus/rabbitmq.ts
import amqp from 'amqplib';

class RabbitMQEventBus {
  private connection: any;
  private channel: any;

  async connect(url: string) {
    this.connection = await amqp.connect(url);
    this.channel = await this.connection.createChannel();
    
    // Set up exchanges
    await this.channel.assertExchange('events', 'topic', { durable: true });
  }

  async publish(event: string, data: any, routingKey: string) {
    const message = Buffer.from(JSON.stringify(data));
    
    await this.channel.publish(
      'events',
      routingKey,
      message,
      { persistent: true }
    );
  }

  async subscribe(routingKey: string, handler: (data: any) => void) {
    const queue = await this.channel.assertQueue('', { exclusive: true });
    
    await this.channel.bindQueue(queue.queue, 'events', routingKey);
    
    await this.channel.consume(queue.queue, (msg: any) => {
      const data = JSON.parse(msg.content.toString());
      handler(data);
      this.channel.ack(msg);
    });
  }

  async disconnect() {
    await this.connection.close();
  }
}

export const eventBus = new RabbitMQEventBus();

// Event definitions
export enum EventType {
  USER_CREATED = 'user.created',
  USER_UPDATED = 'user.updated',
  USER_DELETED = 'user.deleted',
  ORDER_CREATED = 'order.created',
  ORDER_PAID = 'order.paid',
  ORDER_SHIPPED = 'order.shipped',
}

// Event publisher
export class EventPublisher {
  async publishUserCreated(user: any) {
    await eventBus.publish(EventType.USER_CREATED, user, 'user.created');
  }

  async publishOrderCreated(order: any) {
    await eventBus.publish(EventType.ORDER_CREATED, order, 'order.created');
  }
}

// Event subscriber
export class EventSubscriber {
  async setupSubscribers() {
    await eventBus.subscribe('user.*', async (data) => {
      console.log('User event:', data);
      await handleUserEvent(data);
    });

    await eventBus.subscribe('order.*', async (data) => {
      console.log('Order event:', data);
      await handleOrderEvent(data);
    });
  }
}
```

### Pattern 2: AWS SQS Message Queue

Queue-based messaging with AWS SQS:

```typescript
// queue/sqs.ts
import { SQSClient, SendMessageCommand, ReceiveMessageCommand, DeleteMessageCommand } from '@aws-sdk/client-sqs';

const sqsClient = new SQSClient({});

export class SQSQueue {
  constructor(private queueUrl: string) {}

  async sendMessage(message: any): Promise<void> {
    const command = new SendMessageCommand({
      QueueUrl: this.queueUrl,
      MessageBody: JSON.stringify(message),
      DelaySeconds: 0,
    });

    await sqsClient.send(command);
  }

  async receiveMessage(): Promise<any | null> {
    const command = new ReceiveMessageCommand({
      QueueUrl: this.queueUrl,
      MaxNumberOfMessages: 1,
      WaitTimeSeconds: 20,
    });

    const response = await sqsClient.send(command);
    
    if (response.Messages && response.Messages.length > 0) {
      const message = response.Messages[0];
      const body = JSON.parse(message.Body!);
      
      // Delete message after processing
      await this.deleteMessage(message.ReceiptHandle!);
      
      return body;
    }
    
    return null;
  }

  async deleteMessage(receiptHandle: string): Promise<void> {
    const command = new DeleteMessageCommand({
      QueueUrl: this.queueUrl,
      ReceiptHandle: receiptHandle,
    });

    await sqsClient.send(command);
  }
}

// Usage
const orderQueue = new SQSQueue(process.env.ORDER_QUEUE_URL!);

async function processOrders() {
  while (true) {
    const order = await orderQueue.receiveMessage();
    
    if (order) {
      await processOrder(order);
    }
  }
}
```

### Pattern 3: Saga Pattern

Distributed transaction with saga pattern:

```typescript
// saga/order-saga.ts
export class OrderSaga {
  async execute(order: any) {
    const sagaId = crypto.randomUUID();
    
    try {
      // Step 1: Create order
      const createdOrder = await this.createOrder(order);
      
      // Step 2: Reserve inventory
      const inventory = await this.reserveInventory(createdOrder);
      
      // Step 3: Process payment
      const payment = await this.processPayment(createdOrder);
      
      // Step 4: Confirm order
      await this.confirmOrder(createdOrder);
      
      return { success: true, order: createdOrder };
    } catch (error) {
      // Compensating transactions
      await this.compensate(sagaId);
      throw error;
    }
  }

  private async createOrder(order: any) {
    return await orderService.create(order);
  }

  private async reserveInventory(order: any) {
    try {
      return await inventoryService.reserve(order.items);
    } catch (error) {
      await this.cancelOrder(order.id);
      throw error;
    }
  }

  private async processPayment(order: any) {
    try {
      return await paymentService.charge(order);
    } catch (error) {
      await this.releaseInventory(order.items);
      await this.cancelOrder(order.id);
      throw error;
    }
  }

  private async confirmOrder(order: any) {
    return await orderService.confirm(order.id);
  }

  private async compensate(sagaId: string) {
    // Execute compensating transactions
    console.log(`Compensating saga ${sagaId}`);
  }

  private async cancelOrder(orderId: string) {
    await orderService.cancel(orderId);
  }

  private async releaseInventory(items: any[]) {
    await inventoryService.release(items);
  }
}

// Choreography-based saga
export class ChoreographySaga {
  async start(order: any) {
    // Publish order created event
    await eventBus.publish('order.created', order);
  }

  async handleOrderCreated(event: any) {
    try {
      // Reserve inventory
      await inventoryService.reserve(event.items);
      
      // Publish inventory reserved event
      await eventBus.publish('inventory.reserved', event);
    } catch (error) {
      // Publish inventory failed event
      await eventBus.publish('inventory.failed', event);
    }
  }

  async handleInventoryReserved(event: any) {
    try {
      // Process payment
      await paymentService.charge(event);
      
      // Publish payment succeeded event
      await eventBus.publish('payment.succeeded', event);
    } catch (error) {
      // Publish payment failed event
      await eventBus.publish('payment.failed', event);
    }
  }

  async handlePaymentSucceeded(event: any) {
    // Confirm order
    await orderService.confirm(event.id);
    
    // Publish order confirmed event
    await eventBus.publish('order.confirmed', event);
  }

  async handlePaymentFailed(event: any) {
    // Release inventory
    await inventoryService.release(event.items);
    
    // Cancel order
    await orderService.cancel(event.id);
  }
}
```

### Pattern 4: Event Sourcing

Storing events as the source of truth:

```typescript
// event-sourcing/event-store.ts
interface Event {
  id: string;
  aggregateId: string;
  eventType: string;
  data: any;
  timestamp: Date;
  version: number;
}

export class EventStore {
  private events: Event[] = [];

  async saveEvent(event: Event): Promise<void> {
    this.events.push(event);
  }

  async getEvents(aggregateId: string): Promise<Event[]> {
    return this.events.filter(e => e.aggregateId === aggregateId);
  }

  async replayEvents(aggregateId: string): Promise<any> {
    const events = await this.getEvents(aggregateId);
    
    let state = {};
    
    for (const event of events) {
      state = this.applyEvent(state, event);
    }
    
    return state;
  }

  private applyEvent(state: any, event: Event): any {
    switch (event.eventType) {
      case 'USER_CREATED':
        return { ...state, ...event.data };
      case 'USER_UPDATED':
        return { ...state, ...event.data };
      case 'USER_DELETED':
        return null;
      default:
        return state;
    }
  }
}

// Aggregate root
export class UserAggregate {
  private events: Event[] = [];
  private state: any = {};

  applyEvent(event: Event) {
    this.events.push(event);
    this.state = this.applyEventToState(this.state, event);
  }

  private applyEventToState(state: any, event: Event): any {
    switch (event.eventType) {
      case 'USER_CREATED':
        return { ...state, ...event.data };
      case 'USER_UPDATED':
        return { ...state, ...event.data };
      default:
        return state;
    }
  }

  getState() {
    return this.state;
  }

  getUncommittedEvents() {
    return this.events;
  }

  markEventsAsCommitted() {
    this.events = [];
  }
}
```

### Pattern 5: CQRS Pattern

Command Query Responsibility Segregation:

```typescript
// cqrs/command-handler.ts
export class CommandHandler {
  async handle(command: any) {
    switch (command.type) {
      case 'CREATE_USER':
        return this.handleCreateUser(command);
      case 'UPDATE_USER':
        return this.handleUpdateUser(command);
      case 'DELETE_USER':
        return this.handleDeleteUser(command);
    }
  }

  private async handleCreateUser(command: any) {
    const user = await userService.create(command.data);
    
    // Publish event
    await eventBus.publish('user.created', user);
    
    return user;
  }

  private async handleUpdateUser(command: any) {
    const user = await userService.update(command.id, command.data);
    
    // Publish event
    await eventBus.publish('user.updated', user);
    
    return user;
  }

  private async handleDeleteUser(command: any) {
    await userService.delete(command.id);
    
    // Publish event
    await eventBus.publish('user.deleted', { id: command.id });
  }
}

// cqrs/query-handler.ts
export class QueryHandler {
  async handle(query: any) {
    switch (query.type) {
      case 'GET_USER':
        return this.handleGetUser(query);
      case 'GET_USERS':
        return this.handleGetUsers(query);
      case 'SEARCH_USERS':
        return this.handleSearchUsers(query);
    }
  }

  private async handleGetUser(query: any) {
    return await readModel.getUser(query.id);
  }

  private async handleGetUsers(query: any) {
    return await readModel.getUsers(query.filters);
  }

  private async handleSearchUsers(query: any) {
    return await readModel.searchUsers(query.term);
  }
}

// Read model projection
export class UserProjection {
  async handleUserCreated(event: any) {
    await readModel.createUser(event);
  }

  async handleUserUpdated(event: any) {
    await readModel.updateUser(event);
  }

  async handleUserDeleted(event: any) {
    await readModel.deleteUser(event.id);
  }
}
```

## Best Practices

### ✅ Do

- Use idempotent event handlers
- Implement dead-letter queues
- Monitor event throughput
- Use event versioning
- Implement retry logic
- Use proper serialization
- Document event schemas
- Implement event validation

### ❌ Don't

- Lose events
- Ignore ordering requirements
- Skip error handling
- Use events for everything
- Forget about monitoring
- Ignore event size limits
- Skip schema validation
- Use synchronous processing

## Event Schema

```typescript
// event-schema.ts
import { z } from 'zod';

export const UserCreatedEventSchema = z.object({
  id: z.string().uuid(),
  eventType: z.literal('user.created'),
  data: z.object({
    email: z.string().email(),
    name: z.string(),
  }),
  timestamp: z.date(),
  version: z.number(),
});

export const OrderCreatedEventSchema = z.object({
  id: z.string().uuid(),
  eventType: z.literal('order.created'),
  data: z.object({
    userId: z.string().uuid(),
    items: z.array(z.object({
      productId: z.string().uuid(),
      quantity: z.number(),
    })),
    total: z.number(),
  }),
  timestamp: z.date(),
  version: z.number(),
});
```

## Resources

- [RabbitMQ Documentation](https://www.rabbitmq.com/docs)
- [AWS SQS Documentation](https://docs.aws.amazon.com/sqs/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Event Sourcing Pattern](https://martinfowler.com/eaaDev/EventSourcing.html)
