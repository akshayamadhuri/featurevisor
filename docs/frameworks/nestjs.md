---
title: NestJS
description: Learn how to integrate Featurevisor in Nest.js applications
ogImage: /img/og/docs-frameworks-nest.png
---

# Set up Featurevisor SDK instance in a NestJS application using a custom middleware, including TypeScript integration.

## Hello World application

Before diving into Featurevisor integration, let's create a simple Hello World [NestJS](https://nestjs.com/) application.

Install the Nest CLI:

```bash
$ npm install -g @nestjs/cli
```

Create a new NestJS project:

```bash
$ nest new featurevisor-nest-app
$ cd featurevisor-nest-app
```
## Featurevisor Integration

Start by installing the Featurevisor SDK:

```bash
$ npm install --save @featurevisor/sdk
```

Now, let's create an instance of the SDK and use it in our application:

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { createInstance } from '@featurevisor/sdk';
import { CommonConfigService } from './common-config.service'; // Import the configuration service
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
   // Create an instance of the SDK using the configuration values
  const f = createInstance({
    datafileUrl: CommonConfigService.DATAFILE_URL,
    refreshInterval: CommonConfigService.REFRESH_INTERVAL,
  });
  // Use the middleware
  app.use((req, res, next) => {
    req.f = f;
    next();
  });
  await app.listen(3000);
}
bootstrap();
```

## Middleware

To ensure the same Featurevisor SDK instance is available in all routes, create a custom middleware in Nest.js.

Create a new file named featurevisor.middleware.ts:

```typescript
// featurevisor.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { FeaturevisorInstance, createInstance } from '@featurevisor/sdk';
const DATAFILE_URL = 'https://cdn.yoursite.com/datafile.json';
const REFRESH_INTERVAL = 60 * 5; // every 5 minutes
const f: FeaturevisorInstance = createInstance({
  datafileUrl: DATAFILE_URL,
  refreshInterval: REFRESH_INTERVAL,
});
export function featurevisorMiddleware(req: Request, res: Response, next: NextFunction) {
  req.f = f;
  next();
}
```

Update the main.ts file to use the middleware:

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { featurevisorMiddleware } from './featurevisor.middleware';
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(featurevisorMiddleware);
  await app.listen(3000);
}
bootstrap();
```

## Using Featurevisor in a Route

You can now utilize the Featurevisor SDK instance in your routes:

```typescript
// cats.controller.ts
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';
import { FeaturevisorInstance } from '@featurevisor/sdk';
@Controller('cats')
export class CatsController {
  constructor(private readonly f: FeaturevisorInstance) {}
  @Get()
  getCats(@Req() req: Request) {
    const featureKey = 'myFeature';
    const context = { userId: 'user-123' };
    const isEnabled = req.f.isEnabled(featureKey, context);
    if (isEnabled) {
      return 'Hello Cats!';
    } else {
      return 'Not enabled for cats yet!';
    }
  }
}
```

## TypeScript Usage

For TypeScript users, extend the Request interface to include the f property for Featurevisor SDK's instance.

Create a new featurevisor.d.ts file in the src folder:

```typescript
// src/featurevisor.d.ts
import { FeaturevisorInstance } from '@featurevisor/sdk';
declare module 'express' {
  interface Request {
    f: FeaturevisorInstance;
  }
}
```
