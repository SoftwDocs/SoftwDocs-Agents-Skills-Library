---
name: real-time-communication
description: Complete real-time communication implementation with WebSockets, Socket.IO, WebRTC, push notifications, and live collaboration features for modern web applications.
tags: [websocket, socket.io, webrtc, real-time, collaboration, push-notifications]
version: 1.0.0
author: SoftwDocs
---

# Real-Time Communication

## Overview

A comprehensive skill for building real-time communication features in web applications. Covers WebSockets, Socket.IO, WebRTC for video/audio, push notifications, live collaboration, and best practices for scalable real-time systems.

## When to Use This Skill

- Building chat applications
- Implementing live collaboration features
- Adding video/audio calling
- Creating real-time dashboards
- Implementing push notifications
- Building multiplayer games
- Live data streaming

## Core Technologies

### WebSockets
- Native WebSocket API
- Socket.IO for enhanced features
- WS library for Node.js

### WebRTC
- Peer-to-peer video/audio
- Screen sharing
- Data channels

### Push Notifications
- Web Push API
- Service Workers
- Firebase Cloud Messaging

## Implementation Patterns

### Pattern 1: Socket.IO Server

Real-time server with Socket.IO:

```typescript
// server/index.ts
import { createServer } from 'http';
import { Server } from 'socket.io';
import express from 'express';
import cors from 'cors';

const app = express();
app.use(cors());

const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: {
    origin: process.env.CLIENT_URL,
    methods: ['GET', 'POST'],
  },
});

// Authentication middleware
io.use(async (socket, next) => {
  const token = socket.handshake.auth.token;
  
  try {
    const user = await verifyToken(token);
    socket.data.user = user;
    next();
  } catch (error) {
    next(new Error('Authentication error'));
  }
});

// Connection handling
io.on('connection', (socket) => {
  const user = socket.data.user;
  console.log(`User ${user.id} connected`);

  // Join user's personal room
  socket.join(`user:${user.id}`);

  // Join team rooms
  user.teamIds?.forEach((teamId: string) => {
    socket.join(`team:${teamId}`);
  });

  // Handle disconnection
  socket.on('disconnect', () => {
    console.log(`User ${user.id} disconnected`);
  });

  // Chat messages
  socket.on('send_message', async (data) => {
    const { roomId, message } = data;
    
    const savedMessage = await saveMessage({
      userId: user.id,
      roomId,
      content: message,
    });

    io.to(`room:${roomId}`).emit('new_message', savedMessage);
  });

  // Typing indicators
  socket.on('typing_start', (data) => {
    socket.to(`room:${data.roomId}`).emit('user_typing', {
      userId: user.id,
      userName: user.name,
    });
  });

  socket.on('typing_stop', (data) => {
    socket.to(`room:${data.roomId}`).emit('user_stopped_typing', {
      userId: user.id,
    });
  });

  // Join room
  socket.on('join_room', (roomId) => {
    socket.join(`room:${roomId}`);
    socket.to(`room:${roomId}`).emit('user_joined', {
      userId: user.id,
      userName: user.name,
    });
  });

  // Leave room
  socket.on('leave_room', (roomId) => {
    socket.leave(`room:${roomId}`);
    socket.to(`room:${roomId}`).emit('user_left', {
      userId: user.id,
      userName: user.name,
    });
  });
});

const PORT = process.env.PORT || 3001;
httpServer.listen(PORT, () => {
  console.log(`Socket.IO server running on port ${PORT}`);
});
```

### Pattern 2: Socket.IO Client

Client-side Socket.IO implementation:

```typescript
// client/socket.ts
import { io, Socket } from 'socket.io-client';

class SocketService {
  private socket: Socket | null = null;

  connect(token: string) {
    this.socket = io(process.env.SOCKET_URL!, {
      auth: { token },
      transports: ['websocket'],
    });

    this.socket.on('connect', () => {
      console.log('Connected to server');
    });

    this.socket.on('disconnect', () => {
      console.log('Disconnected from server');
    });

    this.socket.on('connect_error', (error) => {
      console.error('Connection error:', error);
    });

    return this.socket;
  }

  disconnect() {
    this.socket?.disconnect();
    this.socket = null;
  }

  joinRoom(roomId: string) {
    this.socket?.emit('join_room', roomId);
  }

  leaveRoom(roomId: string) {
    this.socket?.emit('leave_room', roomId);
  }

  sendMessage(roomId: string, message: string) {
    this.socket?.emit('send_message', { roomId, message });
  }

  onNewMessage(callback: (message: Message) => void) {
    this.socket?.on('new_message', callback);
  }

  onUserTyping(callback: (data: { userId: string; userName: string }) => void) {
    this.socket?.on('user_typing', callback);
  }

  onUserStoppedTyping(callback: (data: { userId: string }) => void) {
    this.socket?.on('user_stopped_typing', callback);
  }

  onUserJoined(callback: (data: { userId: string; userName: string }) => void) {
    this.socket?.on('user_joined', callback);
  }

  onUserLeft(callback: (data: { userId: string; userName: string }) => void) {
    this.socket?.on('user_left', callback);
  }
}

export const socketService = new SocketService();

// React hook
import { useEffect, useState } from 'react';

export function useSocket(token: string) {
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const socket = socketService.connect(token);
    
    socket.on('connect', () => setConnected(true));
    socket.on('disconnect', () => setConnected(false));

    return () => {
      socketService.disconnect();
    };
  }, [token]);

  return { connected, socket: socketService };
}
```

### Pattern 3: WebRTC Video Calling

Peer-to-peer video/audio with WebRTC:

```typescript
// server/webrtc.ts
import { Server } from 'socket.io';

export function setupWebRTC(io: Server) {
  io.on('connection', (socket) => {
    // Join call room
    socket.on('join_call', (roomId: string) => {
      socket.join(`call:${roomId}`);
      
      const otherUsers = io.sockets.adapter.rooms.get(`call:${roomId}`);
      const users = Array.from(otherUsers || []).filter(id => id !== socket.id);
      
      socket.emit('all_users', users);
    });

    // WebRTC signaling
    socket.on('offer', (data) => {
      socket.to(`call:${data.roomId}`).emit('offer', {
        offer: data.offer,
        callerId: socket.id,
      });
    });

    socket.on('answer', (data) => {
      socket.to(`call:${data.roomId}`).emit('answer', {
        answer: data.answer,
        calleeId: socket.id,
      });
    });

    socket.on('ice_candidate', (data) => {
      socket.to(`call:${data.roomId}`).emit('ice_candidate', {
        candidate: data.candidate,
        senderId: socket.id,
      });
    });

    // Leave call
    socket.on('leave_call', (roomId: string) => {
      socket.leave(`call:${roomId}`);
      socket.to(`call:${roomId}`).emit('user_left_call', socket.id);
    });
  });
}

// client/webrtc.ts
export class WebRTCService {
  private peerConnection: RTCPeerConnection | null = null;
  private localStream: MediaStream | null = null;
  private remoteStream: MediaStream | null = null;

  async initLocalStream() {
    this.localStream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });
    return this.localStream;
  }

  createPeerConnection(config: RTCConfiguration) {
    this.peerConnection = new RTCPeerConnection(config);

    // Add local stream
    this.localStream?.getTracks().forEach(track => {
      this.peerConnection?.addTrack(track, this.localStream!);
    });

    // Handle remote stream
    this.peerConnection.ontrack = (event) => {
      this.remoteStream = event.streams[0];
    };

    // Handle ICE candidates
    this.peerConnection.onicecandidate = (event) => {
      if (event.candidate) {
        socket.emit('ice_candidate', {
          candidate: event.candidate,
          roomId: currentRoomId,
        });
      }
    };

    return this.peerConnection;
  }

  async createOffer() {
    const offer = await this.peerConnection?.createOffer();
    await this.peerConnection?.setLocalDescription(offer);
    return offer;
  }

  async createAnswer(offer: RTCSessionDescriptionInit) {
    await this.peerConnection?.setRemoteDescription(offer);
    const answer = await this.peerConnection?.createAnswer();
    await this.peerConnection?.setLocalDescription(answer);
    return answer;
  }

  async setRemoteDescription(description: RTCSessionDescriptionInit) {
    await this.peerConnection?.setRemoteDescription(description);
  }

  async addIceCandidate(candidate: RTCIceCandidateInit) {
    await this.peerConnection?.addIceCandidate(candidate);
  }

  cleanup() {
    this.localStream?.getTracks().forEach(track => track.stop());
    this.peerConnection?.close();
    this.localStream = null;
    this.remoteStream = null;
    this.peerConnection = null;
  }
}
```

### Pattern 4: Push Notifications

Web Push API with Service Workers:

```typescript
// server/push.ts
import webpush from 'web-push';

// Configure VAPID keys
webpush.setVapidDetails(
  'mailto:your-email@example.com',
  process.env.VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
);

export async function sendPushNotification(subscription: PushSubscription, data: any) {
  try {
    await webpush.sendNotification(subscription, JSON.stringify(data));
  } catch (error) {
    console.error('Push notification failed:', error);
  }
}

// API route to subscribe
// app/api/subscribe/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { saveSubscription } from '@/services/push';

export async function POST(request: NextRequest) {
  const subscription = await request.json();
  await saveSubscription(subscription);
  return NextResponse.json({ success: true });
}

// API route to send notification
// app/api/notify/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getSubscriptions, sendPushNotification } from '@/services/push';

export async function POST(request: NextRequest) {
  const { title, body, data } = await request.json();
  const subscriptions = await getSubscriptions();

  const notifications = subscriptions.map(subscription =>
    sendPushNotification(subscription, { title, body, data })
  );

  await Promise.all(notifications);

  return NextResponse.json({ success: true });
}

// Service worker registration
// public/sw.js
self.addEventListener('install', (event) => {
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  event.waitUntil(self.clients.claim());
});

self.addEventListener('push', (event) => {
  const data = event.data.json();
  
  const options = {
    body: data.body,
    icon: '/icon.png',
    badge: '/badge.png',
    data: data.data,
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(
    self.clients.openWindow(event.notification.data.url)
  );
});

// Client-side subscription
// components/PushNotification.tsx
import { useEffect } from 'react';

export function PushNotification() {
  useEffect(() => {
    if ('serviceWorker' in navigator && 'PushManager' in window) {
      navigator.serviceWorker.register('/sw.js').then((registration) => {
        return registration.pushManager.subscribe({
          userVisibleOnly: true,
          applicationServerKey: process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY,
        });
      }).then((subscription) => {
        fetch('/api/subscribe', {
          method: 'POST',
          body: JSON.stringify(subscription),
        });
      });
    }
  }, []);

  return null;
}
```

### Pattern 5: Live Collaboration

Real-time collaborative editing:

```typescript
// server/collaboration.ts
import { Server } from 'socket.io';

export function setupCollaboration(io: Server) {
  const documents = new Map<string, any>();

  io.on('connection', (socket) => {
    // Join document room
    socket.on('join_document', (documentId: string) => {
      socket.join(`doc:${documentId}`);
      
      const document = documents.get(documentId) || { content: '', version: 0 };
      socket.emit('document_state', document);
    });

    // Document updates
    socket.on('document_update', (data) => {
      const { documentId, operation, version } = data;
      const document = documents.get(documentId);

      if (document && document.version === version) {
        // Apply operation
        document.content = applyOperation(document.content, operation);
        document.version += 1;
        documents.set(documentId, document);

        // Broadcast to other users
        socket.to(`doc:${documentId}`).emit('document_update', {
          operation,
          version: document.version,
        });
      } else {
        // Version conflict, send current state
        socket.emit('document_state', document);
      }
    });

    // Cursor position
    socket.on('cursor_move', (data) => {
      const { documentId, position, userId, userName } = data;
      socket.to(`doc:${documentId}`).emit('cursor_update', {
        userId,
        userName,
        position,
      });
    });

    // Leave document
    socket.on('leave_document', (documentId: string) => {
      socket.leave(`doc:${documentId}`);
      socket.to(`doc:${documentId}`).emit('user_left_document', socket.data.user.id);
    });
  });
}

// Operation transformer for conflict resolution
function applyOperation(content: string, operation: any): string {
  // Implement operational transformation
  // This is a simplified example
  if (operation.type === 'insert') {
    return content.slice(0, operation.position) + 
           operation.text + 
           content.slice(operation.position);
  }
  
  if (operation.type === 'delete') {
    return content.slice(0, operation.position) + 
           content.slice(operation.position + operation.length);
  }
  
  return content;
}
```

## Best Practices

### ✅ Do

- Use authentication for all connections
- Implement reconnection logic
- Handle connection errors gracefully
- Use rooms for targeted messages
- Implement rate limiting
- Scale horizontally with Redis adapter
- Use WebRTC for peer-to-peer
- Implement proper error handling

### ❌ Don't

- Skip authentication
- Ignore connection limits
- Broadcast to all clients unnecessarily
- Forget to handle disconnections
- Use polling instead of WebSockets
- Ignore scalability concerns
- Forget to clean up resources
- Skip error logging

## Scaling

```typescript
// Using Redis adapter for horizontal scaling
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));
```

## Resources

- [Socket.IO Documentation](https://socket.io/docs/)
- [WebRTC Documentation](https://webrtc.org/)
- [Web Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)
- [Operational Transformation](https://en.wikipedia.org/wiki/Operational_transformation)
