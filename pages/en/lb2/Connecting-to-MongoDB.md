---
title: "Connecting to MongoDB"
lang: en
layout: page
keywords: LoopBack
tags:
sidebar: lb2_sidebar
permalink: /doc/en/lb2/Connecting-to-MongoDB.html
summary:
---

{% include important.html content="

This example application describes using the StrongLoop `slc loopback` tool. You can also recreate the application using API Connect `apic` tool.
Simply substitute the API Connect command for the corresponding StrongLoop command. For more information, see
[Converting StrongLoop commands to API Connect commands](https://docs.strongloop.com/display/APIC/Command-line+reference#Command-linereference-CorrespondencebetweenStrongLoopcommandstoAPIConnectcommands).

" %}

# loopback-example-database

A tutorial for basic database related features.

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Running the example](#running-the-example)
* [Tutorial - MongoDB](#tutorial---mongodb)

## Overview

### Topics covered

* Data sources
* Creating
* Configuring
* Models
* Creating
* Automigration
* Instance introspection (Discovery)

### Database specific tutorials

Database specific tutorials are on separate branches. The master branch contains the tutorial for MongoDB.

<table>
  <thead>
    <tr>
      <th align="left">Branch</th>
      <th align="left">Connector</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="left"><a href="https://github.com/strongloop/loopback-example-database">master</a></td>
      <td align="left">MongoDB</td>
    </tr>
    <tr>
      <td align="left"><a href="https://github.com/strongloop/loopback-example-database/tree/mssql">mssql</a></td>
      <td align="left">Microsoft SQL Server</td>
    </tr>
    <tr>
      <td align="left"><a href="https://github.com/strongloop/loopback-example-database/tree/mysql">mysql</a></td>
      <td align="left">MySQL</td>
    </tr>
    <tr>
      <td align="left"><a href="https://github.com/strongloop/loopback-example-database/tree/oracle">oracle</a></td>
      <td align="left">Oracle</td>
    </tr>
    <tr>
      <td align="left"><a href="https://github.com/strongloop/loopback-example-database/tree/postgresql">postgresql</a></td>
      <td align="left">PostgreSQL</td>
    </tr>
  </tbody>
</table>

For example, to view the MySQL example:

```
git clone https://github.com/strongloop/loopback-example-database
cd loopback-example-database
git checkout mysql
```

## Prerequisites

Before starting this tutorial, make sure you have the following installed:

* Node
* NPM
* [StrongLoop Controller](https://github.com/strongloop/strongloop)

## Running the example

```
git clone https://github.com/strongloop/loopback-example-database
cd loopback-example-database
npm install
npm start
```

## Tutorial - MongoDB

### 1. Create a new LoopBack app

#### App info

* Name: `loopback-example-database`
* Dir to contain the project: `loopback-example-database`

```
slc loopback loopback-example-database
... # follow the prompts
```

### 2. Install the LoopBack MongoDB connector

```
cd loopback-example-database
npm install --save loopback-connector-mongodb
```

### 3. Create a data source

#### Data source info

* Data source name: `accountDS`
* Select the connector for `accountDS`: `MongoDB`

```
slc loopback:datasource accountDS
... # follow the prompts
```

This creates a new data source named `accountDS` that uses the MongoDB connector.

### 4. Configure the data source

For the purposes of this example, we will use a preconfigured StrongLoop MongoDB server.
Edit `server/datasources.json` to set the MongoDB configs:

```javascript
{
  ...
  "accountDS": {
    "name": "accountDS",
    "connector": "mongodb",
    "host": "demo.strongloop.com",
    "port": 27017,
    "database": "demo",
    "username": "demo",
    "password": "L00pBack"
  }
}
```

> Feel free to use your own local MongoDB instance. Simply change the configs
> above to match your own.

### 5. Create a new model

#### Model Info

* Model name: `Account`
* Attach `Account` to: `accountDS (mongodb)`
* Base class: `PersistedModel`
* Expose via REST: `Yes`
* Custom plural form: _Leave blank_
* Properties:
* `email`
  * String
  * Not required
* `createdAt`
  * Date
  * Not required
* `lastModifiedAt`
  * Date
  * Not required

```
slc loopback:model Account
... # follow the prompts
```

### 6. Create the collection with sample data - Automigration

With the `account` model configured, we can generate the corresponding MongoDB collection using the info from the `Account` metadata in
[`common/models/account.json`](common/models/account.json)
via
[auto-migration](https://docs.strongloop.com/display/public/LB/Implementing+auto-migration).

Start by creating a dir to store general-purpose scripts:

`mkdir bin`

Inside that dir, create a script named [`automigrate.js`](bin/automigrate.js).
To create the `Account` collection and create two sample accounts, run:

`node bin/automigrate.js`

> **WARNING**
>
> The `automigrate` function creates a new collection if it doesn't exist. If
> the collection already exists, **it will be destroyed and it's data will be
> deleted**. If you want to keep this data, use `autoupdate` instead.

You should see:

```
Created: { email: 'baz@qux.com',
  createdAt: Thu Oct 22 2015 17:58:09 GMT-0700 (PDT),
  lastModifiedAt: Thu Oct 22 2015 17:58:09 GMT-0700 (PDT),
  id: 562986213ea33440575c6588 }
Created: { email: 'foo@bar.com',
  createdAt: Thu Oct 22 2015 17:58:09 GMT-0700 (PDT),
  lastModifiedAt: Thu Oct 22 2015 17:58:09 GMT-0700 (PDT),
  id: 562986213ea33440575c6587 }
```

> If you are using Node 4, it is safe to ignore `Swagger: skipping unknown type "ObjectId"`. This warning will be addressed in a future update.

### 7. View data using the explorer

Projects scaffolded via `slc loopback` come with `loopback-component-explorer` preconfigured.
From the project root, start the server:

`node .`

Then to view the existing account data, browse to `localhost:3000/explorer` and
click:

* `GET /Accounts`
* `Try it out!`

You should see:

```javascript
[{
  "email": "foo@bar.com",
  "createdAt": "2015-10-23T00:58:09.280Z",
  "lastModifiedAt": "2015-10-23T00:58:09.280Z",
  "id": "562986213ea33440575c6587"
}, {
  "email": "baz@qux.com",
  "createdAt": "2015-10-23T00:58:09.280Z",
  "lastModifiedAt": "2015-10-23T00:58:09.280Z",
  "id": "562986213ea33440575c6588"
}]
```

> Try out some of the other endpoints to get a feel for how explorer works.

### 8. Add a script to perform instance introspection (Discovery)

> [Discovery](https://docs.strongloop.com/display/public/LB/Discovering+models+from+relational+databases)
> is the process of reverse engineering a LoopBack model from an existing database schema.

The LoopBack MongoDB connector does not support discovery.
However, you can use _instance introspection_, which creates a LoopBack model from an existing JavaScript object.

To do this, create a script named [`instance-introspections.js`](bin/instance-introspection.js)
in the `bin` dir. Then run:

`node bin/instance-introspection`

You should see:

```javascript
Created: { email: 'bob.doe@ibm.com',
  createdAt: Thu Oct 22 2015 19:38:20 GMT-0700 (PDT),
  lastModifiedAt: Thu Oct 22 2015 19:38:20 GMT-0700 (PDT),
  id: 56299d9d71c7f600719ca39f }
```

> See the [official docs](http://docs.strongloop.com/display/LB/Creating+models+from+unstructured+data)
> for more info.

* * *

[More LoopBack examples](https://github.com/strongloop/loopback-example)