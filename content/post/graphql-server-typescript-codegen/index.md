+++
author = "Jeff Chang"
title = "Organize your GraphQL modules codebase and create a type safe environment in GraphQL resolvers"
date = "2023-07-17"
description = "Let's create a type safe environment in GraphQL resolvers with GraphQL codegen and Typescript!"
tags = [
    "nodejs", "graphql", "typescript"
]
categories = [
    "Node","Typescript", "GraphQL"
]
image= "cover.jpg"
+++

## Table of contents

- [Introduction](#introduction)
- [Objective](#objective)
- [Organize resolvers and type-def](#organize-file)
- [Generate resolvers type](#resolver-type-safe)
- [Resolvers type verifications](#verifications)

### Introduction<a name="introduction"></a>

Before we get started, it's high recommend to clone the [Github repo](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen) and we can follow along with the article contents.

The first step after we downloaded the repo is to checkout the the **starter** folder and run `npm install` to install the necessary dependencies

### Objective<a name="objective"></a>

Below are the directory of our start pack in this tutorial, where each modules have been organized in a different folder.

- **03\-graphql\-server\-typescript**
  - **data**
    - **final**
      - [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/data/final/index.ts)
    - **initial**
      - [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/data/initial/index.ts)
  - [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/index.ts)
  - **modules**
    - [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/index.ts)
    - **sport**
      - [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/sport/index.ts)
      - [resolvers.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/sport/resolvers.ts)
      - [type\-def.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/sport/type-def.ts)
    - **user**
      - [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/user/index.ts)
      - [resolvers.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/user/resolvers.ts)
      - [type\-def.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/user/type-def.ts)
  - [schema.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/schema.ts)
  - [tsconfig.json](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/tsconfig.json)
  - [package\-lock.json](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/package-lock.json)
  - [package.json](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/package.json)

Our goal in this tutorial are:

- Merge each of the resolvers and type-defs into [modules/index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/index.ts) file
- Generate the type for the resolvers to ensure it is type safe.

### Organize resolvers and type-def<a name="organize-file"></a>

We would need install a few dependency to help us speed up the process, which are:

- [@graphql-codegen/cli](https://www.npmjs.com/package/@graphql-codegen/cli)
- [@graphql-codegen/typescript](https://www.npmjs.com/package/@graphql-codegen/typescript)
- [@graphql-codegen/typescript-resolvers](https://www.npmjs.com/package/@graphql-codegen/typescript-resolvers)

Here is the command snippet you can copy to install these dependencies:
`npm install @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers -D`

Once these packages are ready, we could then proceed to merge our GraphQL resolvers and type-defs under [modules/index.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/index.ts) file

```ts
import { mergeResolvers, mergeTypeDefs } from "@graphql-tools/merge";
import { makeExecutableSchema } from "@graphql-tools/schema";
import { sport } from "./sport";
import { user } from "./user";

const modules = [sport, user];

const resolvers = mergeResolvers(
  modules.flatMap((m) => (m.resolvers ? [m.resolvers] : []))
);

export const typeDefs = mergeTypeDefs(modules.map((m) => m.typeDefs));

export const schema = makeExecutableSchema({
  resolvers,
  typeDefs,
});
```

Then we could specified our `schema` into our GraphQL server under the [index.ts](https://github.com/Jeffcw96/graphql-learning-journey/blob/master/03-graphql-server-typescript-codegen/starter/src/index.ts) file.

```ts
import { ApolloServer } from "apollo-server";
import { schema } from "src/modules";

const server = new ApolloServer({
  schema: schema,
});

server.listen().then(({ url }) => {
  console.log(`GRAPHQL server is running at ${url}`);
});
```

#### Explaination

1. If we take a look on one of the module index file, we are basically an object which contains it's `resolvers` and `typeDefs`, we then import them into the module's index file and start doing the merging process.
2. As you can see, we have a filtering process when we are merging the resolvers by using [flatMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap) to avoid any potential error if the resolver is not existed in any of the file.
3. We can then merge our resolvers and type-defs by using [`mergeResolvers`](https://the-guild.dev/graphql/tools/docs/schema-merging#merging-resolvers) and [`mergeTypeDefs`](https://the-guild.dev/graphql/tools/docs/schema-merging#merging-type-definitions) methods which then allows us to create our GraphQL schema by inputting these parameters and using [`makeExecutableSchema`](https://the-guild.dev/graphql/tools/docs/schema-directives#using-schema-directives) method.
4. Once our `schema` is ready, we could then connect it into our GraphQL Apollo server which replaced the `typeDefs` and `resolvers` fields input.
5. Now, we should we able to run our GraphQL server locally without getting any error.

### Generate resolvers type<a name="resolver-type-safe"></a>

Before we start relying on `GraphQL codegen` to generate our GraphQL resolvers type, we would need to define some rules under the `codegen.yml` file.
We can create a `codegen.yml` file in the root directory which parked under the same level of `package.json`, `package-lock.json` and `tsconfig.json`.

Here will be the content of our `codegen.yml` file:

```yaml
schema: ./src/schema.ts
require:
  - ts-node/register/transpile-only
  - tsconfig-paths/register
generates:
  ./src/modules/schema.d.ts:
    plugins:
      - "@graphql-codegen/typescript"
      - "@graphql-codegen/typescript-resolvers"
```

We could also update our `package.json` file to include the code generator command:

```json
{
    ...
    "scripts": {
        ...
        "prestart": "npm run codegen",
        "codegen": "graphql-codegen"
    },
}
```

#### Explaination

1. We first need to specify the location of our entire `typeDefs` of each module which is stored under the [schema.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/schema.ts) file.
2. Adding dependencies into the `require` field is to make sure GraphQL codegen actually picks up these dependencies when generating the types. In this case, this is to avoid the Typescript path alias and `import` keyword issue.
   - Adding `ts-node` is for make sure it doesn't get import issue when referring src/schema.ts as it's the same directory level of graphql server (index.ts)
   - Adding `tsconfig-paths/register` to make sure the graphql-codegen doesn't get path alias error when refering the src/modules/\*/resolvers files
3. We can then specified the location of type generated file which is `/src/modules/schema.d.ts` in this case under the `generates` field with the necessary plugins which are `@graphql-codegen/typescript` and `@graphql-codegen/typescript-resolvers` to support the Typescript in GraphQL resolvers.
4. Adding `codegen` command in package.json allows us to generate our type in anytime when we ran this command, while adding `prestart` command is to make sure our Apollo GraphQL server retrieved the latest version of the Resolver type before the server got started.

### Resolvers type verifications<a name="verifications"></a>

Once the `codegen` command has been run successfully, you should able to see there is a file called `schema.d.ts` generated under the `modules` directory.
We could then assign the resolver type in each of the modules. For example:

[modules/sport/resolvers.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/sport/resolvers.ts):

```ts
import { Sports } from 'src/data/initial';
import { Resolvers } from '../schema';

export const resolvers:Resolvers = {
  Query: {
    ...
  },
  Mutation: {
   ...
  },
};

```

[modules/user/resolvers.ts](https://github.com/Jeffcw96/graphql-learning-journey/tree/master/03-graphql-server-typescript-codegen/starter/src/modules/user/resolvers.ts):

```ts
import { Users } from 'src/data/initial';
import { Resolvers } from '../schema';

export const resolvers:Resolvers = {
  Query: {
    ...
  },
  Mutation: {
   ...
  },
};

```

Once we have defined our resolver type, we should immediately get a type error as the following:
`Types of property 'id' are incompatible. Type 'number' is not assignable to type 'string'`

This is a advantage of using Typescript rather than Javascript to avoid any potential issue in the dev environment. The GraphQL [ID](https://graphql.org/learn/schema/#scalar-types) type is refers to a STRING type but not a INTEGER type and the ID in our returned data is a INTEGER.

To solve this, we have prepared a new set of data where the `id` field is now in a STRING type. We could easily update our data directory from `src/data/initial` to `src/data/final` and it would resolve the issue
