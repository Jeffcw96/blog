+++
author = "Jeff Chang"
title= "RabbitMQ topic exchange" 
date= "2022-05-28"
description= "In the last 2 articles, we have explained 2 types of exchange in RabbitMQ which are direct and fanout exchange.<br><br>They are pretty cool as with direct exchange, you are able to personalize your exchange and publish the message as long as the routing key is matched. Whereas for fanout exchange, it behaves like a pub/sub mechanism where you get to publish your message to every consumer as long as their queue is bound to the exchange.<br><br>In this article, we will be introducing another exchange type called topic exchange and this exchange type is very useful as it allows you to attach with certain rules in your routing key so that it will filter out and only publish to the matching rule queue." 
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

## Code demonstration

We will run `publisher.js` and `comsumer.js` separately.

### publisher.js<a name="publisher-code"></a>

<!-- prettier-ignore -->
{{< highlight js>}}
const amqp = require("amqplib");
async function topicExchangePublisher() {
  try {
    const rabbitmqUrl = "amqp://localhost:3031";
    const connection = await amqp.connect(rabbitmqUrl);
    const exchange = "animals";
    const exchangeType = "topic";
    const routingKey = "animal.mammal.tiger";
    const options = {};
    const payload = {
      numberOfAnimal: 10,
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
topicExchangePublisher();
{{< /highlight >}}

### consumer.js<a name="consumer-code"></a>

<!-- prettier-ignore -->
{{< highlight js>}}
const amqp = require("amqplib");
async function topicExchangeConsumer() {
  try {
    const rabbitmqUrl = "amqp://localhost:3031";
    const connection = await amqp.connect(rabbitmqUrl);
    const exchange = "animals";
    const exchangeType = "topic";
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
topicExchangeConsumer();

{{< /highlight >}}

**Before we start:**

- To have better visualization, please run the `publisher.js` in 1 terminal and `consumer.js` in 2 terminals. You can either use the [split terminal in Visual Studio Code](https://code.visualstudio.com/docs/editor/integrated-terminal#_grouping) or run the files in your machine terminals.
- If you're not using the example from the provided [Git Repository](https://github.com/Jeffcw96/rabbit-mq). Please ensure you run the `consumer.js` in 2 terminals followed by running `publisher.js` in 1 terminal as shown in below:
  - `node consumer.js animal.#`
  - `node consumer.js "animal.*.tiger"`
  - `node producder`

### Result for this example

<video controls muted style="width:100%">
  <source src="example.mov" type="video/mp4">
  <source src="example.ogg" type="video/ogg">
</video>

## Steps to demo from [Git Repository](https://github.com/Jeffcw96/rabbit-mq)

- Open a new terminal
- `npm run dev` to start the server
- Open **2** new terminals and run the following command:

  - `npm run consumer`

  ```js
    {
      "payload":{
          "numberOfAnimal": 10
      },
      "properties":{
          "routing_key":"animal.mammal.tiger"
      }
    }
  ```

- Make a POST request to `http://localhost:3000/api/topic/exchange/animals` with the payload above.
- **Note :** If you wanted to add your new custom data such as `routing_key`, `exchange name` and etc, please ensure you have read the instructions in [ReadMe](https://github.com/Jeffcw96/rabbit-mq#how-to-use)

### Result

<video controls muted style="width:100%">
  <source src="gitrepo.mov" type="video/mp4">
  <source src="gitrepo.ogg" type="video/ogg">
</video>

## Explanations

- **Publisher**

  1. Before process started, we need to first connect to the RabbitMQ.
  2. The next step is to make a connection with the channel and start creating our desired exchange by using `assertExchange()` method
     - In here we specify the following argument's value:
       - exchange name is `animals` in this example
       - exchange type is `topic`
       - we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertexchange)
  3. And now, we can start publishing our message to the exchange by using [`publish()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_publish) method with the following arguments:
     - we can use back the exchange name we specified earlier **'animals'**
     - `routingKey` in this case played an important property as in the consumer binding queue process, the broker would need to refer to this value and bind to the corresponding queue.
     - `payload`, Publish message only accepts **buffer** payload. To achieve this, we can first **stringify** our payload if it's an object and convert it into a Buffer by using [`Buffer.from`](https://www.w3schools.com/nodejs/met_buffer_from.asp)
     - `options`, we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channel_publish)

- **Comsumer**
  1. Before the process is started, we need to first connect to the RabbitMQ.
  2. The next step is to make a connection with the channel and start creating our desired exchange by using `assertExchange()` method
     - In here we specify the following argument's value:
       - exchange name is `animals` in this example
       - exchange type is `topic`
       - we didn't specify the options in this example but feel free to have a look at the available options at [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertexchange)
  3. And now we can start to creating our queue and bind with the exchange we created earlier.
     - As you can observe from the code, we are putting it as an empty string when creating the queue `channel.assertQueue("",options)`. This is because we do not need to bother with the queue name when binding the exchange and queue. By putting an empty string for the queue name, it will return us a unique and random queue name such as `amq.gen-JzTY20BRgKO-HjmUJj0wLg`
     - There are some useful options we can take note of when creating the queue :
       - `durable` : if true, the queue still stays alive even if we restart the broker.
       - `exclusive` : if true, queue will be deleted when the connection is closed.
       - `expires` : specify the time in a millisecond to delete the queue when there is no consumer connecting to the queue.
       - checkout other options [here](https://amqp-node.github.io/amqplib/channel_api.html#channelassertqueue)
       - `routingKey` property is very important for `topic` exchange as it will refer to the message properties sent from the publisher and bind the exchange with its queue if their `routingKey` condition is matched.
  4. There are few important rule we need to adhere for `topic` exchange:
     - Topic exchange routing key must be a list of words, delimited by dots
     - `*` (star) can substitute for exactly one word.
     - `#` (hash) can substitute for zero or more words.
  5. From example above, the `routingKey` used by the publisher is known as `animal.mammal.tiger`. We will be able to receive the message if the routing key for consumer is `animal.*.tiger` or `animal.#`, and not receiving any message if the routing key for consumer is `animal.insect.*`. This is because:
     - `animal.*.tiger`: The `*` (star) could be replaced by any kind of string. Even if the **publisher** provide `animal.huge.tiger` and it will still works.
     - `animal.#`: This will works because as long as the routing key from publisher is contains `animal` as the first character in the words.
     - `animal.insect.*`: This will not work because the character `insect` does not fulfilled the condition for `animal.mammal.tiger` unless the `routingKey` used by the publisher is updated to `animal.insect.tiger`.
  6. After the binding process is finished, we can now consume the message whenever there is an incoming message from the broker by using [`.consume()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_consume) method
  7. When we received the message, we can now **acknowledge** the message by using [`ack()`](https://amqp-node.github.io/amqplib/channel_api.html#channel_ack) method so that the broker will know we successfully retrieve the message.
