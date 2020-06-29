---
layout: post
author: jensendarren
title: Installing Hasura Graph QL Engine
permalink: /2019/12/installing-hasura-graphql-engine
categories:
  Databases, Graph QL, Hasura, Postgres
tags:
  - databases
  - APIs
  - Hasura
  - Graph QL
---

### Docker

First up, install Docker Desktop on your computer. The best thing would be to head over to the official [Docker website](https://www.docker.com/products/docker-desktop) for on how to do that.

### Docker-Compose

Then install Docker Compose and fire up the database. [Here](https://github.com/devbootstrap/installing-hasura-on-docker) is a repo containing a `docker-compose.yml` file with the full configuration for Postgres, Hasura and PG Admin. Just download that repo and run the usual `docker-compose up` in your terminal to startup everything.

### Hasura

Install Hasura GraphQL Engine in your **local environment** using Docker Compose. I recommend following the [Installing Hasura on Docker](https://github.com/devbootstrap/installing-hasura-on-docker) tutorial from [DevBootstrap](https://www.devbootstrap.com).

### Basic Graph QL query

Let's assume that you have a Postgres database with the following table structure already created:

```
profile (
  id INT PRIMARY KEY,
  name TEXT
)
```

Then you can use the Hasura Console to test querieng the database. For example if you want to fetch all the rows in the table via Graph QL, you could issue the following query:

```
query {
  profile {
    id
    name
  }
}
```

### Video Tutorial

Head over to the [DevBootstrap YouTube Channel](https://www.youtube.com/watch?v=a4NlZw9OXz8) for a video tutorial that covers this topic.

### Conclusion

If you are considering using Graph QL in your next project, be sure to checkout Hasura as an option!