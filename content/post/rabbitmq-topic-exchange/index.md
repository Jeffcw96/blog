+++
author = "Jeff Chang"
title= "RabbitMQ topic exchange" 
date= "2022-05-25"
description= "In the last 2 articles, we have explained 2 types of exchange in RabbitMQ which are direct and fanout exchange. They are pretty cool as with direct exchange, you are able to personalize your exchange and publish the message as long as the routing key is matched. Whereas for fanout exchange, it behaves like a pub/sub mechanism where you get to publish your message to every consumer as long as their queue is bound to the exchange. In this article, we will be introducing another exchange type called topic exchange" 
tags = [
    "nodejs","javascript","rabbitmq"
]
categories = [
    "NodeJs","Javascript","RabbitMQ"
]
image = "cover.jpg"
+++

Topic exchange is very useful as it allows you to attach with certain rules in your routing key so that it will filter out and only publish to the matching rule queue.

Prerequisite:

- Please ensure you've [installed RabbitMQ](https://www.rabbitmq.com/download.html) in your local machine.
  - Alternatively, you may run RabbitMQ docker container
  - `docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management`
- We will using RabbitMQ Javascript client [amqplib](https://amqp-node.github.io/amqplib/). Please ensure you've setup a new project with this package installed.
  - Alternatively, you can also clone this [project](https://github.com/Jeffcw96/rabbit-mq) to have application-ready demonstrations.
