+++
author = "Jeff Chang"
title= "RabbitMQ direct exchange"
date= "2022-05-05"
description= "In the last article, we were demonstrating the implementation of RabbitMQ fanout exchange which behaves the pattern like publish and subscribe. In this article, we will be demonstrating the other exchange type called Direct exchange" 
tags = [
    "nodejs","javascript","rabbitmq"
]
categories = [
    "NodeJs","Javascript","RabbitMQ"
]
image = "cover.jpg"
+++

Prerequisite:

- Please ensure you've [installed RabbitMQ](https://www.rabbitmq.com/download.html) in your local machine.
  - Alternatively, you may run RabbitMQ docker container
  - `docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management`
- We will using RabbitMQ Javascript client [amqplib](https://amqp-node.github.io/amqplib/). Please ensure you've setup a new project with this package installed.
  - Alternatively, you can also clone this [project](https://github.com/Jeffcw96/rabbit-mq) to have application-ready demonstrations.

We are going to demonstrate the `Direct exchange` message queue flow as shown in the figure below
![exchange flow](flow.png)

## Code demonstration

We will run `producer.js` and `comsumer.js` separately.

### producer.js<a name="producer-code"></a>

<!-- prettier-ignore -->
{{< highlight js>}}
const amqp = require("amqplib");
async function directExchange() {
  try {
    const rabbitmqUrl = "amqp://localhost:5672";
    const connection = await amqp.connect(rabbitmqUrl);
    const exchange = "jobs";
    const exchangeType = "direct";
    const routingKey = "software_engineer";
    const options = {};
    const payload = {
      Name: "Jeff",
      yearOfExperience: 2,
    };
    let channel = await connection.createChannel();
    await channel.assertExchange(exchange, exchangeType, options);
    channel.publish(
      exchange,
      routingKey,
      Buffer.from(JSON.stringify(payload)),
      options
    );
  } catch (error) {
    console.error(error);
  }
}
directExchange();

{{< /highlight >}}

### consumer.js<a name="consumer-code"></a>

<!-- prettier-ignore -->
{{< highlight js>}}
const amqp = require("amqplib");
async function directExchangeConsumer() {
  try {
    const rabbitmqUrl = "amqp://localhost:5672";
    const connection = await amqp.connect(rabbitmqUrl);
    const exchange = "jobs";
    const exchangeType = "direct";
    const routingKey = process.argv[2];
    const options = {};
    let channel = await connection.createChannel();
    await channel.assertExchange(exchange, exchangeType, options);
    const { queue } = await channel.assertQueue("", options);
    channel.bindQueue(queue, exchange, routingKey);
    channel.consume(queue, (data) => {
      console.log("Received", JSON.parse(data.content.toString()));
      channel.ack(data, false, true);
    });
  } catch (error) {
    console.error(error);
  }
}
directExchangeConsumer();
{{< /highlight >}}

**Before we start:**

- To have better visualization, please run the `producer.js` in 1 terminal and `consumer.js` in 2 terminals. You can either use the [split terminal in Visual Studio Code](https://code.visualstudio.com/docs/editor/integrated-terminal#_grouping) or run the files in your machine terminals.
- If you're not using the example from the provided [Git Repository](https://github.com/Jeffcw96/rabbit-mq). Please ensure you run the `consumer.js` in 2 terminals followed by running `producer.js` in 1 terminal and indicate your `routing_key` value when running the `consumer.js` script e.g : `node consumer.js software_engineer`.
  - example here uses `process.argv[2]` to retrieve the value from the command line and assign it as `routing_key`

### Result for this example

<video controls muted style="width:100%">
  <source src="example.mp4" type="video/mp4">
  <source src="example.ogg" type="video/ogg">
</video>

## Steps to demo from [Git Repository](https://github.com/Jeffcw96/rabbit-mq)

- Open a new terminal
- `npm run dev` to start the server
- Open **2** new terminals and run the following command:
  - `npm run consumer`
- Specify your `routing_key` as shown in below:

  ```
    {
      "payload":{
          "name": "Jeff",
          "yearOfExperience": 2,
      },
      "properties":{
          "routing_key":"software_engineer"
      }
    }
  ```

- Make a POST request to `http://localhost:3000/api/direct/exchange/jobs` with your custom payload.

### Result

<video controls muted style="width:100%">
  <source src="gitrepo.mp4" type="video/mp4">
  <source src="gitrepo.ogg" type="video/ogg">
</video>

## Explanations

- **Producer**

  1. Before process started, we need to first connect to the RabbitMQ.
  2. The next step is to make a connection with the channel and start creating our desired exchange by using `assertExchange()` method
     - In here we specify the following argument's value:
       - exchange name is `jobs` in this example
       - exchange type is `direct`
       - we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertexchange)
  3. And now, we can start publishing our message to the exchange by using [`publish()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_publish) method with the following arguments:
     - we can use back the exchange name we specified earlier **'jobs'**
     - `routingKey` in this case played an important property as in the consumer binding queue process, the broker would need to refer to this value and bind to the corresponding queue.
     - `payload`, Publish message only accepts **buffer** payload. To achieve this, we can first **stringify** our payload if it's an object and convert it into a Buffer by using [`Buffer.from`](https://www.w3schools.com/nodejs/met_buffer_from.asp)
     - `options`, we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channel_publish)

- **Comsumer**
  1. Before the process is started, we need to first connect to the RabbitMQ.
  2. The next step is to make a connection with the channel and start creating our desired exchange by using `assertExchange()` method
     - In here we specify the following argument's value:
       - exchange name is `jobs` in this example
       - exchange type is `direct`
       - we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertexchange)
  3. And now we can start to creating our queue and bind with the exchange we created earlier.
     - As you can observe from the code, we are putting it as an empty string when creating the queue `channel.assertQueue("",options)`. This is because we do not need to bother with the queue name when binding the exchange and queue. By putting an empty string for the queue name, it will return us a unique and random queue name such as `amq.gen-JzTY20BRgKO-HjmUJj0wLg`
     - There are some useful options we can take note of when creating the queue :
       - `durable` : if true, the queue still stays alive even if we restart the broker.
       - `exclusive` : if true, queue will be deleted when the connection is closed.
       - `expires` : specify the time in a millisecond to delete the queue when there is no consumer connecting to the queue.
       - checkout other options [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertqueue)
       - `routingKey` property is very important for `direct` exchange as it will refer to the message properties sent from the producer and bind the exchange with its queue if their `routingKey` value is matched.
  4. After the binding process is finished, we can now consume the message whenever there is an incoming message from the broker by using [`.consume()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_consume) method
  5. When we received the message, we can now **acknowledge** the message by using [`ack()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_ack) method so that the broker will know we successfully retrieve the message.
