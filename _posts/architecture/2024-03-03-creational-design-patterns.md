---
title: Creational Design Patterns - In simple terms
date: 2024-03-03
categories: [Architecture]
tags: [design patterns, architecture, javascript]
pin: false
math: false
mermaid: false
---

## Singleton

The singleton pattern involves creating a constructor that always returns a reference to the first created object of the class. These are helpful when you want to share state between different parts of the software.

```typescript
type Global = {
  logger?: Logger;
};

const global: Global = { logger: undefined };

class Logger {
  constructor() {
    if (global.logger) {
      return global.logger;
    }

    global.logger = this;
    return this;
  }

  info(message: any) {
    console.log(`[INFO - ${new Date().toDateString()}] ${message}`);
  }
}

const logger = new Logger();

logger.info("Hello, world!");
```

Now, let’s read through the code above. While there are various libraries for handling logging, there are instances where additional logic is necessary. Therefore, to avoid creating multiple instances of the logger and generating unnecessary work for the garbage collector, we can simply retain the same instance and use it in all parts of our application.

![Execute singleton](assets/img/singleton-bun-example.png)

Here we compare the two references, and as you can see, JavaScript returns `true` meaning that we have fetched the same object when using the constructor both times.

{: .prompt-tip }

> Here's an [example](https://www.prisma.io/docs/orm/more/help-and-troubleshooting/help-articles/nextjs-prisma-client-dev-practices) of a real life use case of a singleton class when initializing a Prisma client

## Factory Method

> TBA

## Prototype

> TBA

## Builder

> TBA

## Abstract Factory

> TBA
