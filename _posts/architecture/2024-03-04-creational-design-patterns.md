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

The Factory Method design pattern... The first time I encountered this design pattern, I didn't grasp the fact that when they refer to a `method`, they mean a class method, not just a way of instantiating objecs. Well, in a sense, it is related to creating objects, but the main point here is that we establish what is known as an abstract factory. This abstract factory essentially provides a hook for creating said objects.

Why do we need this? The Factory Method allows us to separate the object construction logic from the code that actually uses these objects.

In Quickbase, we recently worked on extending the collection of assets our marketplace supports. We have an application where you can upload an asset and make a request towards the backend to have this asset published to different environments.

For the sake of a good example, I rewrote the factory pattern we used there to conform to the design pattern we're trying to explain here.

```typescript
abstract class AssetPublishingService {
  abstract publish(): void;
}

class ImagePublishingService extends AssetPublishingService {
  constructor() {
    super();
  }

  publish(): void {
    console.log("Publishing image");
  }
}

class VideoPublishingService extends AssetPublishingService {
  constructor() {
    super();
  }

  publish(): void {
    console.log("Publishing video");
  }
}

abstract class AssetPublishingServiceFactory {
  abstract createService(): AssetPublishingService;
}

class ImagePublishingServiceFactory extends AssetPublishingServiceFactory {
  createService(): AssetPublishingService {
    return new ImagePublishingService();
  }
}

class VideoPublishingServiceFactory extends AssetPublishingServiceFactory {
  createService(): AssetPublishingService {
    return new VideoPublishingService();
  }
}
```

We first define an abstract base class called `AssetPublishingService` which has an abstract method called `createService`. This method will be used by the subclasses to create the corresponding services.

Then, we define the two factories which will accordignly create new objects either of the type `VideoPublishingService` or `ImagePublishingService`.

Sounds good. But why?

```typescript
function publishAsset(factory: AssetPublishingServiceFactory) {
  const service = factory.createService();

  service.publish(); // implementation which differs under the hood

  // update upload status in admin app for uploading assets
  // send notifications to clients that a new asset is available in their environment
}

// ...

if (request.type === "image") {
  publishAsset(new ImagePublishingServiceFactory());
} else if (request.type === "video") {
  publishAsset(new VideoPublishingServiceFactory());
}
```

This allows for flexibility and extensibility in the system, as new services can be created without modifying the code which handles the common functionality for publishing assets. Hence, if we decide to implement logic for publishing assets of type `WAV` or `MP3`, the only thing we will have to define is the new asset class with its publishing method accordingly, as well as a new factory. The code in `publishAsset` stays the same.

## Prototype

The prototype, or so called _clone_ pattern, is simply just deep cloning an object.

Picture a scenario where you and a friend are using a shared calculator for your accounting exercises. The history stored in the calculator is important to both of you. However, as you both need to leave, it's crucial to ensure that each of you retains a complete and unaltered copy of the calculator's history.

In an unusual twist, these calculators exist on the same machine, and you access them via the internet. Therefore, a simple `const calculator2 = calculator` won't suffice, as it merely creates a reference to the original object. Any updates made to the copy (calculator2) will automatically reflect in the original calculator because they share the same reference.

To prevent inadvertently overwriting the history, you must create a new instance of the calculator. This involves reinitializing all variables and copying the information over to the new instance, ensuring the integrity of the individual calculators' histories.

```typescript
class Calculator {
  private history: string[] = [];

  public clone(): Calculator {
    const cloned = new Calculator();
    cloned.history = [...this.history];
    return cloned;
  }

  public add(x: number, y: number): number {
    const result = x + y;
    this.history.push(`added ${x} to ${y} got ${result}`);
    return result;
  }

  public getHistory(): string {
    return this.history.join("\n");
  }
}

const calculator = new Calculator();

calculator.add(1, 2);
calculator.add(3, 4);
calculator.add(5, 6);
```

In the `clone` method, observe that we utilize the spread operator for the `history`. This approach ensures the creation of a fresh array instance rather than merely referencing the original `history`. If there were additional class members, a similar procedure would be necessary to ensure their independent instantiation as well.

```typescript
const calculator = new Calculator();
const calculatorClone = calculator.clone();

console.log(calculator === calculatorClone); // false

calculator.add(7, 8);
console.log(calculator.getHistory() === calculatorClone.getHistory()); // false
```

## Builder

> TBA

## Abstract Factory

> TBA
