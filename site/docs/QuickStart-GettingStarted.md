---
title: Getting Started
layout: ../_core/DocsLayout
category: Quick Start
permalink: /docs/getting-started/
next: /docs/videos/
---

Let's build a basic GraphQL server from scratch. We'll be using the **[graphql-js](https://github.com/graphql/graphql-js)** Javascript reference implementation of GraphQL for this example.

Our server will be simple; it will have one type, a `User`, where a user has two fields; an `id` and a `name`. For an example of a more complex server and additional features,
check out the **[walkthrough](../intro)** section of the docs.

## Setup

Start by making a folder for our demo server:

```sh
mkdir graphql-demo
cd graphql-demo
```

Our example server requires [Node.js](https://nodejs.org/en/). Additionally we need three packages for our server:

1. **[graphql](https://github.com/graphql/graphql-js)**, the reference implementation of GraphQL in JS.
2. **[express](https://github.com/strongloop/express)**, a basic web framework.
3. **[express-graphql](https://github.com/graphql/express-graphql)**, middleware for express to make it easy to expose a GraphQL server.

Install these three packages using [npm](https://docs.npmjs.com/getting-started/installing-node):

```sh
npm init -f
npm install graphql express express-graphql --save
```

## Data

Our server will consist of two files, `data.json` and `index.js`.

Create these files:

```sh
touch data.json
touch index.js
```

Now define the data for our users in `data.json`:

```json
{
  "1": {
    "id": "1",
    "name": "Dan"
  },
  "2": {
    "id": "2",
    "name": "Lee"
  },
  "3": {
    "id": "3",
    "name": "Nick"
  }
}
```

## Server

Next we'll create a very basic GraphQL schema to describe our data;
then we'll allow that schema to be queried over HTTP.

Insert the following into `index.js` (be sure to read the comments):

```js
// Import the required libraries
var graphql = require('graphql');
var graphqlHTTP = require('express-graphql');
var express = require('express');

// Import our data from above
var data = require('./data.json');

// Define our user type with two string fields: `id` and `name`.
// The type of User is GraphQLObjectType, which has child fields
// with their own types.
var userType = new graphql.GraphQLObjectType({
  name: 'User',
  fields: {
    id: { type: graphql.GraphQLString },
    name: { type: graphql.GraphQLString },
  }
});

// Define our schema with one top-level field, `user`, that
// takes an `id` argument and returns the User with that ID.
// Note that the `query` is an GraphQLObjectType, just like User.
// The `user` field, however, is of userType, which we defined above.
var schema = new graphql.GraphQLSchema({
  query: new graphql.GraphQLObjectType({
    name: 'Query',
    fields: {
      user: {
        type: userType,
        // `args` describes the arguments that the `user` query accepts
        args: {
          id: { type: graphql.GraphQLString }
        },
        // The resolve function describes how to "resolve" or fulfill
        // the incoming query.
        // In this case we use the `id` argument from above as a key
        // to get the User from `data`
        resolve: function (_, args) {
          return data[args.id];
        }
      }
    }
  })
});

express()
  .use('/graphql', graphqlHTTP({ schema: schema, pretty: true }))
  .listen(3000);

console.log('GraphQL server running on http://localhost:3000/graphql');
```

<script data-inline>
var graphql = require('graphql');

var data = {
  "1": {
    "id": "1",
    "name": "Dan"
  },
  "2": {
    "id": "2",
    "name": "Lee"
  },
  "3": {
    "id": "3",
    "name": "Nick"
  }
};

var userType = new graphql.GraphQLObjectType({
  name: 'User',
  fields: {
    id: { type: graphql.GraphQLString },
    name: { type: graphql.GraphQLString },
  }
});

var schema = new graphql.GraphQLSchema({
  query: new graphql.GraphQLObjectType({
    name: 'Query',
    fields: {
      user: {
        type: userType,
        args: {
          id: { type: graphql.GraphQLString }
        },
        resolve: function (_, args) {
          return data[args.id];
        }
      }
    }
  })
});

global.schema = schema;
</script>

That's it - your basic GraphQL server is done! Start it by running

```sh
node index.js
```

The server should announce that it is running at
[localhost:3000/graphql](http://localhost:3000/graphql).
If you navigate to this address you will receive this notice:

```javascript
{
  "errors": [
    {
      "message": "Must provide query string."
    }
  ]
}
```

We know our server is running - now we just need to send it a query!

## Queries

Below is a very simple query you can make against your schema. To the right is
the result your server should deliver. Take a moment to read the query and the
result. Note that the query has the same basic "shape" as the result. Where the
result is JSON, the query is simply the keys of the JSON.

<script data-inline>
  import MiniGraphiQL from '../_core/MiniGraphiQL';
  renderHere(<MiniGraphiQL schema={global.schema} query={ `
{
  user(id: "1") {
    name
  }
}
`} />);
</script>

You can edit the above query; the result will automatically update when you do.
If you make a syntax mistake it will be highlighted in red. Try replacing
`id: "1"` with `id: "2"`; replace `name` with `id` or `name id`.

Now that you know what a GraphQL query looks like you can query your own server.
Let's start with the simple query

```
{
  user(id: "1") {
    name
  }
}
```

You must send this to your server via a URL query string, so it must be
URL-encoded. The above query URL-encoded is `%7Buser(id:%221%22)%7Bname%7D%7D`.
(You can URL-encode any string in JavaScript with the global `encodeURI`
function: `encodeURI(string)`.) Send this to your server by loading the page
http://localhost:3000/graphql?query=%7Buser(id:%221%22)%7Bname%7D%7D - your
server should respond with

```javascript
{
  "data": {
    "user": {
      "name": "Dan"
    }
  }
}
```

Most modern browsers will URL-encode for you, so you can try just loading
http://localhost:3000/graphql?query={user(id:"1"){name}} (note that whitespace)
is not important in GraphQL (similar to JSON).

Congratulations! You've built your first GraphQL server. Try different queries,
changing the data, or even adding new fields to the schema.
