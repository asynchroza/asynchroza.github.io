---
title: Migrate Comfortably. Abstract APIs With The Facade Pattern.
date: 2024-04-20
categories: [Architecture]
tags: [design patterns, architecture, javascript]
pin: false
math: false
mermaid: false
---

I was recently tasked with the job of migrating one of our services, deployed as an edge lambda, to the latest available LTS version of Node. Unfortunately, this change wasn't just about bumping a couple of packages up; that was part of it, but all of those bumps were major versions introducing new SDK APIs and other breaking changes.

When we develop software, we move fast, we try to account for deadlines, but somehow always end up cutting corners, to have our product ready by the time we had promised Marketing our feature is going to be up and running. In order to avoid such corners coming back to bite us, we can introduce additional layers of abstraction on top of the SDK APIs we use.

One of the issues I encountered while migrating the service was that I had to rewrite not only the handling functionality but the unit tests surrounding it. This could've easily been avoided if we had introduced the Facade Pattern and had the underlying calls wrapped in an abstraction layer that never changes (or rarely does), while the code underneath can mold into different shapes and forms. Our service is supposed to work absolutely the same way, same data in and out, same API requests; why should we rewrite the code in our services?

One of the breaking changes was that the AWS SDK is now a lot more modular. You install only the dependencies you need:

```typescript
// from
const SDK = require('aws-sdk');
const lambda = new AWS.Lambda({...})
await lambda.invoke({...})

// to
const { Lambda, InvokeCommand } = require('@aws-sdk/client-lambda');
const lambda = new Lambda({...})
await lambda.send({...})
```

Rather than accessing this code directly, the Facade Pattern would provide the following abstraction:

```typescript
class FacadeLambdaService {
    constructor(config) {
        this.lambda = new AWS.Lambda(config);
    }

    async invoke(payload) {
        try {
            const lambdaResult = return await this.lambda.invoke(payload).promise();
            const resultPayload = JSON.parse(lambdaResult.Payload);
            const resultBody = resultPayload.body && JSON.parse(resultPayload.body);

            return {payload: resultPayload, parsedBody: resultBody}
        } catch (e) {
            throw new Error(`Error invoking Lambda function: ${e.message}`)
        }
    }
}
```

In this code snippet, we encapsulate the primary functionality within the `invoke` method and set up the client in the constructor. As a result, our codebase takes on the following structure:

```typescript
const lambda = new FacadeLambdaService({region: 'eu-east-1'})

const {parsedBody} = await lambda
    .invoke({
      FunctionName: 'generate-proxy-token',
      Payload: JSON.stringify({
        body: {
          appId,
        },
      }),
    })
```

The only modification that would have been necessary is as follows:

```typescript
class FacadeLambdaService {
    constructor(config) {
        // same configuration, but new way of initialization
        this.lambda = new Lambda(config);
    }

    async invoke(payload) {
        try {
            // same payload, but different way of invoking
            const command = new InvokeCommand(payload);
            const lambdaResult = await this.lambda.send(command);

            // breaking change
            const resultPayload = JSON.parse(Buffer.from(lambdaResult.Payload).toString());

            // same functionality
            const resultBody = resultPayload.body && JSON.parse(resultPayload.body);
        } catch (e) {
            throw new Error(`Error invoking Lambda function: ${e.message}`)
        }
    }
}
```

Now, the codebase remains intact, with only necessary modifications confined to the facade. This minimizes the time spent on identifying newly introduced bugs, tracing occurrences of the old SDK and rewriting code. However, additional effort is needed to devise a suitable interface for mocking implementations, particularly considering the entirely new approach to mocking AWS services. You can create a facade class in a similar manner introducing the logic for mocking invocations, but I would usually suggest prioritizing integration tests for units that involve network requests, and limiting unit tests to functions that modify data objects without relying on external services.
