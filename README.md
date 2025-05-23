# fastify sqs-consumer
[![NPM version](https://img.shields.io/npm/v/@fgiova/fastify-sqs-consumer.svg?style=flat)](https://www.npmjs.com/package/@fgiova/fastify-rest-gateway)
![CI workflow](https://github.com/fgiova/fastify-sqs-consumer/actions/workflows/node.js.yml/badge.svg)
[![TypeScript](https://img.shields.io/badge/%3C%2F%3E-TypeScript-%230074c1.svg)](http://www.typescriptlang.org/)
## Description
This plugin for fastify 5.x allows you to consume messages from AWS SQS queues.
On fastify shutdown a simple wait function is called to wait for the end of the processing of the messages in progress.

**Warning**<br>
To use this plugin, you must have correctly configured your [AWS credentials](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html).

## Install
```bash
npm i @fgiova/fastify-sqs-consumer
```
### Usage
```js
const fastify = require("fastify")()

fastify.register(require("fastify-sqs-consumer"), [
    {
        url: "https://sqs.us-east-1.amazonaws.com/123456789012/MyQueue",
        waitTimeSeconds: 20,
        timeout: 10_000,
        batchSize: 10,
        handlerFunction: async (message, fastify) => {
            return true;
        }
    }
]);
```
### Options
Options are an array of objects with the following properties (one more for each queue):

| Option                 | Type      | Description                                                                                                                                                                                                                                                                                                           |
|------------------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| url*                   | string    | The URL of the Amazon SQS queue from which messages are received.                                                                                                                                                                                                                                                     |
| waitTimeSeconds        | number    | The duration (in seconds, default 20s) for which the call waits for a message to arrive in the queue before returning. If a message is available, the call returns sooner than WaitTimeSeconds. If no messages are available and the wait time expires, the call returns successfully with an empty list of messages. |
| timeout                | number    | The duration before the message is considered as failed: default 90000ms.                                                                                                                                                                                                                                             |
| batchSize              | number    | The maximum number of messages to return. Amazon SQS never returns more messages than this value (however, fewer messages might be returned). Valid values: 1 to 10. Default: 1.                                                                                                                                      |
| handlerFunction*       | function  | The function that will be called for each message.                                                                                                                                                                                                                                                                    |
| messageAttributeNames  | string[]  | Array of caught message's attributes for each message                                                                                                                                                                                                                                                                 |
| attributeNames         | string[]  | Array of caught queue's attributes for each message                                                                                                                                                                                                                                                                   |
| events                 | object    | Events functions for the consumer (detail in next table)                                                                                                                                                                                                                                                              |
| sqs                    | SQSClient | Initialized SQS Client (useful for testing sessions)                                                                                                                                                                                                                                                                  |

### handlerFunction

Handler function is an async function called for each message received from the queue.
Each Error thrown by the function is caught and the message is not deleted from the queue.
Otherwise, the message is deleted from the queue. 

#### Events

| Event name        | Arguments                         | Description                                                                                                                                          |
|-------------------|-----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| error             | error Object                      | Fired when a general error occurs                                                                                                                    |
| processingError   | error Object, message SQS.Message | Fired when an error occurs pending message handling                                                                                                  |
| timeoutError      | error Object, message SQS.Message | Fired when processing of message exceed timeout duration<br/> If function return false message are discarded, otherwise message is bounced to queue. |
| messageReceived   | message SQS.Message               | Fired when message is received                                                                                                                       |
| messageProcessed  | message SQS.Message               | Fired when message is successfully processed                                                                                                         |
| responseProcessed | void                              | Fired after one batch of items (up to batchSize) has been successfully processed.                                                                    |
| stopped           | void                              | Fired when the consumer finally stops its work.                                                                                                      |
| empty             | void                              | Fired when the queue is empty (All messages have been consumed).                                                                                     |

## License
Licensed under [MIT](./LICENSE).

### Acknowledgements
This project is kindly sponsored by: isendu Srl [www.isendu.com](https://www.isendu.com)
