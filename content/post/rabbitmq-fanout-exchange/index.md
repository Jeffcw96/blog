+++
author = "Jeff Chang"
title= "RabbitMQ fanout exchange" 
date= "2022-04-26"
description= "A message queue is a form of asynchronous service-to-service communication used in serverless and microservices architectures. One of the good example is by using RabbitMQ. In this article, we will demonstrate how to perform a publish and subscribe process via RabbitMQ" 
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

We are going to demonstrate the `Fanout exchange` message queue flow as shown in the figure below
![exchange flow](flow.png)

## Code demonstration

We will run `producer.js` and `comsumer.js` separately.

### producer.js<a name="producer-code"></a>

<!-- prettier-ignore -->
{{< highlight js>}}
const amqp = require("amqplib");
async function fanoutExchange(){
    try{
        const rabbitmqUrl = "amqp://localhost:5672";
        const connection = await amqp.connect(rabbitmqUrl);
        const exchange = "transports";
        const exchangeType = "fanout";
        const routingKey = "";
        const options = {};
        const payload = {
            vehicleType: "car",
            numberOfPassenger: 3,
        };
        let channel = await connection.createChannel();
        await channel.assertExchange(exchange, exchangeType, options);
        channel.publish(
            exchange,
            routingKey,
            Buffer.from(JSON.stringify(payload)),
            options
        );
    }catch(error){
        console.error(error)
    }
}
fanoutExchange()
{{< /highlight >}}

### consumer.js<a name="consumer-code"></a>

<!-- prettier-ignore -->
{{< highlight js>}}
const amqp = require("amqplib");
async function fanoutExchangeConsumer(){
    try{
        const rabbitmqUrl = "amqp://localhost:5672";
        const connection = await amqp.connect(rabbitmqUrl);
        const exchange = "transports";
        const exchangeType = "fanout";
        const routingKey = "";
        const options = {};
        let channel = await connection.createChannel();
        await channel.assertExchange(exchange, exchangeType, options);
        const { queue } = await channel.assertQueue("", options);
        channel.bindQueue(queue, exchange, routingKey);
        channel.consume(queue, (data) => {
            console.log("Received", JSON.parse(data.content.toString()));
            channel.ack(data, false, true);
        });
    }catch(error){
        console.error(error)
    }
}
fanoutExchangeConsumer()
{{< /highlight >}}

**Before we start:**

- To have better visualization, please run the `producer.js` in 1 terminal and `consumer.js` in 2 terminals. You can either use the [split terminal in Visual Studio Code](https://code.visualstudio.com/docs/editor/integrated-terminal#_grouping) or run the files in your machine terminals.
- If you're not using the example from the provided [Git Repository](https://github.com/Jeffcw96/rabbit-mq). Please ensure you run the `consumer.js` in 2 terminals followed by running `producer.js` in 1 terminal

### Result for this example

<video controls muted style="width:100%">
  <source src="example.mp4" type="video/mp4">
  <source src="example.ogg" type="video/ogg">
</video>

## Steps to demo from [Git Repository](https://github.com/Jeffcw96/rabbit-mq)

- Open a new terminal
- `npm run dev` to start the server
- Open **2** new terminals and run the following command:
  - `npm run consumer process=exchange exchange=transports exchangeType=fanout routingKey=''`
- Make a POST request to `http://localhost:3000/api/transports/exchange/fanout` with your custom payload.

### Result

<video controls muted style="width:100%">
  <source src="gitrepo.mp4" type="video/mp4">
  <source src="gitrepo.ogg" type="video/ogg">
</video>

## Explanations

- **Producer**

  1. Before the process started, we need to first connect to the RabbitMQ.
  2. The next step is to make connection with the channel and start creating our desired exchange by using `assertExchange()` method
     - In here we specify the following argument's value:
       - exchange name is `transports` in this example
       - exchange type is `fanout`
       - we didn't specify the options in this example but feel free to have a look on the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertexchange)
  3. And now, we can start publishing our message to the exchange by using [`publish()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_publish) method with the following arguments:
     - we can use back the exchange name we specified earlier **'transports'**
     - `routingKey`, we could just leave it as empty as it doesn't apply anything for `fanout` exchange type
     - `payload`, Publish message only accepts **buffer** payload. To achieve this, we can first **stringify** our payload if it's an object and convert it into a Buffer by using [`Buffer.from`](https://www.w3schools.com/nodejs/met_buffer_from.asp)
     - `options`, we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channel_publish)

- **Comsumer**
  1. Before the process started, we need to first connect to the RabbitMQ.
  2. The next step is to make connection with the channel and start creating our desired exchange by using `assertExchange()` method
     - In here we specify the following argument's value:
       - exchange name is `transports` in this example
       - exchange type is `fanout`
       - we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertexchange)
  3. And now we can start to create our queue and bind with the exchange we created earlier.
     - As you can observe from the code, we are putting it as an empty string when creating the queue `channel.assertQueue("",options)`. This is because we do not need to bother with the queue name when binding the exchange and queue. By putting an empty string for the queue name, it will return us a unique and random queue name such as `amq.gen-JzTY20BRgKO-HjmUJj0wLg`
     - There are some useful options we can take note of when creating the queue:
       - `durable` : if true, the queue still stays alive even if we restart the broker.
       - `exclusive` : if true, the queue will be deleted when connection is closed.
       - `expires` : specify the time in a millisecond to delete the queue when no consumer is connecting to the queue.
       - checkout other options [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertqueue)
       - We could just leave the `routing key` an empty for the binding between exchange and queue because it does not apply anything for the `fanout` exchange.
       - Check out the [direct](/p/rabbitmq-direct-exchange/) and [topic](#) exchange for the use case of `routing key`
  4. After the binding process is finished, we can now consume the message whenever there is an incoming message from the broker by using [`.consume()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_consume) method
  5. When we received the message, we can now **acknowledge** the message by using [`ack()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_ack) method so that the broker will know we successfully retrieve the message.
