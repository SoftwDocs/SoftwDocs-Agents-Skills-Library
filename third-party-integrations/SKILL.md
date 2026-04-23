---
name: third-party-integrations
description: Complete implementation patterns for third-party service integrations including Stripe payments, Twilio communications, SendGrid emails, AWS services, and OAuth authentication with proper error handling and security.
tags: [integrations, stripe, twilio, sendgrid, aws, oauth, payments]
version: 1.0.0
author: SoftwDocs
---

# Third-Party Integrations

## Overview

A comprehensive skill for integrating third-party services into applications. Covers Stripe payments, Twilio SMS/voice, SendGrid emails, AWS services, OAuth authentication, webhooks, and best practices for secure, reliable integrations.

## When to Use This Skill

- Implementing payment processing with Stripe
- Adding SMS/voice communications with Twilio
- Sending transactional emails with SendGrid
- Integrating AWS services (S3, SNS, SES)
- Implementing OAuth authentication
- Handling webhooks from third-party services
- Building multi-service integrations

## Core Integrations

### Payments
- Stripe for payment processing
- Subscription management
- Webhook handling

### Communications
- Twilio for SMS and voice
- SendGrid for email
- Push notifications

### Cloud Services
- AWS S3 for file storage
- AWS SNS for notifications
- AWS SES for email

### Authentication
- OAuth 2.0 flows
- JWT token management
- Social login providers

## Implementation Patterns

### Pattern 1: Stripe Payment Integration

Complete payment processing with Stripe:

```typescript
// services/stripe.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-01-27',
});

export interface CreatePaymentIntentParams {
  amount: number;
  currency: string;
  customerId?: string;
  metadata?: Record<string, string>;
}

export async function createPaymentIntent(params: CreatePaymentIntentParams) {
  try {
    const paymentIntent = await stripe.paymentIntents.create({
      amount: params.amount,
      currency: params.currency,
      customer: params.customerId,
      metadata: params.metadata,
      automatic_payment_methods: {
        enabled: true,
      },
    });
    
    return paymentIntent;
  } catch (error) {
    console.error('Stripe payment intent creation failed:', error);
    throw new Error('Failed to create payment intent');
  }
}

export async function confirmPayment(paymentIntentId: string) {
  try {
    const paymentIntent = await stripe.paymentIntents.retrieve(paymentIntentId);
    
    if (paymentIntent.status === 'succeeded') {
      return { success: true, paymentIntent };
    }
    
    return { success: false, paymentIntent };
  } catch (error) {
    console.error('Stripe payment confirmation failed:', error);
    throw new Error('Failed to confirm payment');
  }
}

export async function createCustomer(params: {
  email: string;
  name: string;
  metadata?: Record<string, string>;
}) {
  try {
    const customer = await stripe.customers.create({
      email: params.email,
      name: params.name,
      metadata: params.metadata,
    });
    
    return customer;
  } catch (error) {
    console.error('Stripe customer creation failed:', error);
    throw new Error('Failed to create customer');
  }
}

export async function createSubscription(params: {
  customerId: string;
  priceId: string;
  trialPeriodDays?: number;
}) {
  try {
    const subscription = await stripe.subscriptions.create({
      customer: params.customerId,
      items: [{ price: params.priceId }],
      trial_period_days: params.trialPeriodDays,
      payment_behavior: 'default_incomplete',
      payment_settings: {
        save_default_payment_method: 'on_subscription',
      },
    });
    
    return subscription;
  } catch (error) {
    console.error('Stripe subscription creation failed:', error);
    throw new Error('Failed to create subscription');
  }
}

export async function cancelSubscription(subscriptionId: string) {
  try {
    const subscription = await stripe.subscriptions.cancel(subscriptionId);
    return subscription;
  } catch (error) {
    console.error('Stripe subscription cancellation failed:', error);
    throw new Error('Failed to cancel subscription');
  }
}

// API route for payment intent
// app/api/payment-intent/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createPaymentIntent } from '@/services/stripe';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { amount, currency, customerId } = body;
    
    if (!amount || !currency) {
      return NextResponse.json(
        { error: 'Amount and currency are required' },
        { status: 400 }
      );
    }
    
    const paymentIntent = await createPaymentIntent({
      amount: Math.round(amount * 100), // Convert to cents
      currency,
      customerId,
    });
    
    return NextResponse.json({ clientSecret: paymentIntent.client_secret });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create payment intent' },
      { status: 500 }
    );
  }
}

// Webhook handler
// app/api/stripe-webhook/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Stripe from 'stripe';
import { handlePaymentSucceeded } from '@/services/webhooks';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;
  
  let event: Stripe.Event;
  
  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (error) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }
  
  switch (event.type) {
    case 'payment_intent.succeeded':
      await handlePaymentSucceeded(event.data.object as Stripe.PaymentIntent);
      break;
    case 'payment_intent.payment_failed':
      await handlePaymentFailed(event.data.object as Stripe.PaymentIntent);
      break;
    case 'customer.subscription.created':
      await handleSubscriptionCreated(event.data.object as Stripe.Subscription);
      break;
    case 'customer.subscription.deleted':
      await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
      break;
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
  
  return NextResponse.json({ received: true });
}
```

### Pattern 2: Twilio SMS Integration

SMS and voice communication with Twilio:

```typescript
// services/twilio.ts
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

export async function sendSMS(params: {
  to: string;
  body: string;
}) {
  try {
    const message = await client.messages.create({
      from: process.env.TWILIO_PHONE_NUMBER,
      to: params.to,
      body: params.body,
    });
    
    return message;
  } catch (error) {
    console.error('Twilio SMS failed:', error);
    throw new Error('Failed to send SMS');
  }
}

export async function sendVerificationCode(phone: string, code: string) {
  const message = `Your verification code is: ${code}. Valid for 10 minutes.`;
  return sendSMS({ to: phone, body: message });
}

export async function makeCall(params: {
  to: string;
  url: string;
}) {
  try {
    const call = await client.calls.create({
      from: process.env.TWILIO_PHONE_NUMBER,
      to: params.to,
      url: params.url,
    });
    
    return call;
  } catch (error) {
    console.error('Twilio call failed:', error);
    throw new Error('Failed to make call');
  }
}

// API route for sending SMS
// app/api/send-sms/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { sendSMS } from '@/services/twilio';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { to, message } = body;
    
    if (!to || !message) {
      return NextResponse.json(
        { error: 'Phone number and message are required' },
        { status: 400 }
      );
    }
    
    const result = await sendSMS({ to, body: message });
    
    return NextResponse.json({ success: true, sid: result.sid });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to send SMS' },
      { status: 500 }
    );
  }
}
```

### Pattern 3: SendGrid Email Integration

Transactional email with SendGrid:

```typescript
// services/sendgrid.ts
import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY!);

export interface SendEmailParams {
  to: string;
  subject: string;
  html: string;
  from?: string;
  templateId?: string;
  dynamicTemplateData?: Record<string, any>;
}

export async function sendEmail(params: SendEmailParams) {
  try {
    const msg = {
      to: params.to,
      from: params.from || process.env.SENDGRID_FROM_EMAIL!,
      subject: params.subject,
      html: params.html,
      templateId: params.templateId,
      dynamicTemplateData: params.dynamicTemplateData,
    };
    
    await sgMail.send(msg);
    return { success: true };
  } catch (error) {
    console.error('SendGrid email failed:', error);
    throw new Error('Failed to send email');
  }
}

export async function sendWelcomeEmail(email: string, name: string) {
  return sendEmail({
    to: email,
    subject: 'Welcome to Our Platform',
    html: `
      <h1>Welcome, ${name}!</h1>
      <p>Thank you for signing up. We're excited to have you on board.</p>
      <p>Best regards,<br>The Team</p>
    `,
  });
}

export async function sendPasswordResetEmail(email: string, resetLink: string) {
  return sendEmail({
    to: email,
    subject: 'Password Reset Request',
    html: `
      <h1>Password Reset</h1>
      <p>You requested a password reset. Click the link below to reset your password:</p>
      <a href="${resetLink}">Reset Password</a>
      <p>This link will expire in 1 hour.</p>
    `,
  });
}

export async function sendOrderConfirmationEmail(
  email: string,
  orderId: string,
  amount: number
) {
  return sendEmail({
    to: email,
    subject: `Order Confirmation #${orderId}`,
    html: `
      <h1>Order Confirmed</h1>
      <p>Your order #${orderId} has been confirmed.</p>
      <p>Total: $${amount.toFixed(2)}</p>
      <p>Thank you for your purchase!</p>
    `,
  });
}

// API route for sending email
// app/api/send-email/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { sendEmail } from '@/services/sendgrid';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { to, subject, html } = body;
    
    if (!to || !subject || !html) {
      return NextResponse.json(
        { error: 'To, subject, and html are required' },
        { status: 400 }
      );
    }
    
    await sendEmail({ to, subject, html });
    
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to send email' },
      { status: 500 }
    );
  }
}
```

### Pattern 4: AWS S3 Integration

File storage with AWS S3:

```typescript
// services/aws-s3.ts
import { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const BUCKET_NAME = process.env.AWS_S3_BUCKET_NAME!;

export async function uploadFile(params: {
  key: string;
  body: Buffer | string;
  contentType: string;
}) {
  try {
    const command = new PutObjectCommand({
      Bucket: BUCKET_NAME,
      Key: params.key,
      Body: params.body,
      ContentType: params.contentType,
    });
    
    await s3Client.send(command);
    
    return {
      success: true,
      url: `https://${BUCKET_NAME}.s3.${process.env.AWS_REGION}.amazonaws.com/${params.key}`,
    };
  } catch (error) {
    console.error('S3 upload failed:', error);
    throw new Error('Failed to upload file');
  }
}

export async function getFile(params: { key: string }) {
  try {
    const command = new GetObjectCommand({
      Bucket: BUCKET_NAME,
      Key: params.key,
    });
    
    const response = await s3Client.send(command);
    return response.Body;
  } catch (error) {
    console.error('S3 get failed:', error);
    throw new Error('Failed to get file');
  }
}

export async function getPresignedUrl(params: { key: string; expiresIn?: number }) {
  try {
    const command = new GetObjectCommand({
      Bucket: BUCKET_NAME,
      Key: params.key,
    });
    
    const url = await getSignedUrl(s3Client, command, {
      expiresIn: params.expiresIn || 3600,
    });
    
    return url;
  } catch (error) {
    console.error('S3 presigned URL failed:', error);
    throw new Error('Failed to generate presigned URL');
  }
}

export async function deleteFile(params: { key: string }) {
  try {
    const command = new DeleteObjectCommand({
      Bucket: BUCKET_NAME,
      Key: params.key,
    });
    
    await s3Client.send(command);
    
    return { success: true };
  } catch (error) {
    console.error('S3 delete failed:', error);
    throw new Error('Failed to delete file');
  }
}

// API route for file upload
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { uploadFile } from '@/services/aws-s3';

export async function POST(request: NextRequest) {
  try {
    const formData = await request.formData();
    const file = formData.get('file') as File;
    
    if (!file) {
      return NextResponse.json(
        { error: 'No file provided' },
        { status: 400 }
      );
    }
    
    const buffer = Buffer.from(await file.arrayBuffer());
    const key = `uploads/${Date.now()}-${file.name}`;
    
    const result = await uploadFile({
      key,
      body: buffer,
      contentType: file.type,
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to upload file' },
      { status: 500 }
    );
  }
}
```

### Pattern 5: OAuth 2.0 Integration

Social login with OAuth:

```typescript
// services/oauth.ts
import { OAuth2Client } from 'google-auth-library';

const googleClient = new OAuth2Client(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET
);

export async function verifyGoogleToken(idToken: string) {
  try {
    const ticket = await googleClient.verifyIdToken({
      idToken,
      audience: process.env.GOOGLE_CLIENT_ID,
    });
    
    const payload = ticket.getPayload();
    
    if (!payload) {
      throw new Error('Invalid token payload');
    }
    
    return {
      email: payload.email,
      name: payload.name,
      picture: payload.picture,
      sub: payload.sub,
    };
  } catch (error) {
    console.error('Google token verification failed:', error);
    throw new Error('Failed to verify Google token');
  }
}

// GitHub OAuth
export async function getGitHubAccessToken(code: string) {
  const response = await fetch('https://github.com/login/oauth/access_token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Accept: 'application/json',
    },
    body: JSON.stringify({
      client_id: process.env.GITHUB_CLIENT_ID,
      client_secret: process.env.GITHUB_CLIENT_SECRET,
      code,
    }),
  });
  
  const data = await response.json();
  return data.access_token;
}

export async function getGitHubUser(accessToken: string) {
  const response = await fetch('https://api.github.com/user', {
    headers: {
      Authorization: `Bearer ${accessToken}`,
    },
  });
  
  return response.json();
}

// OAuth callback handler
// app/api/auth/callback/google/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifyGoogleToken } from '@/services/oauth';
import { createOrGetUser } from '@/services/auth';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { idToken } = body;
    
    if (!idToken) {
      return NextResponse.json(
        { error: 'ID token is required' },
        { status: 400 }
      );
    }
    
    const googleUser = await verifyGoogleToken(idToken);
    const user = await createOrGetUser({
      email: googleUser.email!,
      name: googleUser.name!,
      avatar: googleUser.picture,
      provider: 'google',
      providerId: googleUser.sub,
    });
    
    const token = await generateJWT(user);
    
    return NextResponse.json({ user, token });
  } catch (error) {
    return NextResponse.json(
      { error: 'Authentication failed' },
      { status: 500 }
    );
  }
}
```

## Webhook Security

```typescript
// utils/webhook.ts
import crypto from 'crypto';

export function verifyWebhookSignature(
  payload: string,
  signature: string,
  secret: string
): boolean {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(payload);
  const digest = hmac.digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(digest)
  );
}

// Usage with Stripe
export function verifyStripeWebhook(payload: string, signature: string): boolean {
  return verifyWebhookSignature(
    payload,
    signature,
    process.env.STRIPE_WEBHOOK_SECRET!
  );
}

// Usage with custom webhooks
export function verifyCustomWebhook(payload: string, signature: string): boolean {
  return verifyWebhookSignature(
    payload,
    signature,
    process.env.WEBHOOK_SECRET!
  );
}
```

## Error Handling

```typescript
// utils/integration-errors.ts
export class IntegrationError extends Error {
  constructor(
    public service: string,
    public originalError: any,
    message: string
  ) {
    super(message);
    this.name = 'IntegrationError';
  }
}

export function handleIntegrationError(error: any, service: string) {
  console.error(`${service} integration error:`, error);
  
  if (error.response) {
    // API error
    throw new IntegrationError(
      service,
      error,
      `${service} API error: ${error.response.data?.message || error.message}`
    );
  }
  
  if (error.request) {
    // Network error
    throw new IntegrationError(
      service,
      error,
      `${service} network error: Unable to reach service`
    );
  }
  
  // Other error
  throw new IntegrationError(
    service,
    error,
    `${service} error: ${error.message}`
  );
}
```

## Rate Limiting

```typescript
// utils/rate-limiter.ts
import { LRUCache } from 'lru-cache';

const rateLimitCache = new LRUCache<string, number>({
  max: 1000,
  ttl: 60000, // 1 minute
});

export function checkRateLimit(
  identifier: string,
  maxRequests: number = 100
): boolean {
  const current = rateLimitCache.get(identifier) || 0;
  
  if (current >= maxRequests) {
    return false;
  }
  
  rateLimitCache.set(identifier, current + 1);
  return true;
}

// Usage in API routes
export function withRateLimit(handler: any) {
  return async (request: NextRequest) => {
    const ip = request.headers.get('x-forwarded-for') || 'unknown';
    
    if (!checkRateLimit(ip, 100)) {
      return NextResponse.json(
        { error: 'Rate limit exceeded' },
        { status: 429 }
      );
    }
    
    return handler(request);
  };
}
```

## Best Practices

### ✅ Do

- Store API keys in environment variables
- Implement webhook signature verification
- Use retry logic for transient failures
- Log all integration errors
- Implement rate limiting
- Use webhooks for async events
- Cache responses when appropriate
- Monitor integration health

### ❌ Don't

- Hardcode API keys
- Skip error handling
- Ignore rate limits
- Log sensitive data
- Skip webhook verification
- Use synchronous calls for long operations
- Forget to clean up resources
- Ignore deprecation warnings

## Environment Variables

```bash
# Stripe
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PUBLISHABLE_KEY=pk_live_...

# Twilio
TWILIO_ACCOUNT_SID=AC...
TWILIO_AUTH_TOKEN=...
TWILIO_PHONE_NUMBER=+1234567890

# SendGrid
SENDGRID_API_KEY=SG....
SENDGRID_FROM_EMAIL=noreply@example.com

# AWS
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1
AWS_S3_BUCKET_NAME=my-bucket

# OAuth
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...

# Webhooks
WEBHOOK_SECRET=...
```

## Resources

- [Stripe Documentation](https://stripe.com/docs)
- [Twilio Documentation](https://www.twilio.com/docs)
- [SendGrid Documentation](https://docs.sendgrid.com)
- [AWS SDK Documentation](https://docs.aws.amazon.com/sdk-for-javascript)
- [OAuth 2.0 Specification](https://oauth.net/2/)
