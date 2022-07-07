---
layout: post
title: Weaviate 1.14 release
description: "Learn, what is new in Weaviate 1.14, the most reliable and observable Weaviate release yet!"
published: true
author: Sebastian Witalec
author-img: /img/people/icon/sebastian.jpg
hero-img: /img/blog/weaviate-1.14/reliability.png
toc: true
---

## What is new

We are excited to announce the release of Weaviate 1.14, the most reliable and observable Weaviate release yet. 

<!-- > This is possibly the most boring release that I am most excited about <span>Etienne Dilocker – CTO at SeMI</span> -->

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Later this week, we will release Weaviate v1.14, possibly the most boring release so far.😱 Yet I&#39;m incredibly excited about it and so should you. Why? A 🧵 (1/9)</p>&mdash; Etienne Dilocker (@etiennedi) <a href="https://twitter.com/etiennedi/status/1544689150217027584?ref_src=twsrc%5Etfw">July 6, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Besides many bug fixes and reliability improvements, we have a few neat features that you might find interesting. In short, this release covers:

* Reliability fixes and improvements
* Monitoring and Observability
* Support for non-cosine distances
* API changes

## Reliability fixes and improvements

![Reliability and fixes](/img/blog/weaviate-1.14/reliability.png)

At SeMI Technologies, Reliability is one of our core values, which means that we will always strive to make our software dependable, bug-free, and behave as expected. 

And yes, bug fixing is not always the most exciting topic, as we often get more excited about shiny new features. But for you to truly enjoy working with Weaviate, we need to make sure that no bugs are getting in the way.

Check out [the changelog](https://github.com/semi-technologies/weaviate/releases/tag/v1.14.0){:target="_blank"} to see the complete list of features and over 25 bug fixes.

### Critical bug fix in compaction logic

In this release we fixed a critical bug, which in rare situations could result in data loss.<br/>
The bug affected environments with frequent updates and deletes.

> This bug fix alone, makes it worth upgrading to Weaviate 1.14.

#### Background

We found a critical error in the compactioniong logic that could lead to the compaction operation either corrupting or completely losing data elements.

This could be obsereved through a variety of symptoms:
  * Retrieving an object by it's ID would lead to a different result than retrieving the object using a filter on the id property
  * Filters that should match a specific number of objects matched fewer objects than expected
  * Objects missing completely
  * Filters with `limit=1` would not return any results when there should be exactly one element, but increasing the limit would then include the object
  * Filters would return results with `null` ids

#### Example
In the first case, if you had an object with id: **my-id-123456**.

Calling the following GraphQL API with a filter on id would return the expected object.

```graphql
{
  Get {
    Article(where: {
        path: ["id"],
        operator: Equal,
        valueString: "my-id-123456"
      }) {
      title
    }
  }
}
```

However, calling the following REST API with the same id wouldn't get the object back.
```
GET /v1/objects/{my-id-123456}
```

#### The problem

So, if your data manipulation logic depended on the above operations to perform as expected, you update and delete operations might have been issued incorrectly.

### Improved performance for large scale data imports

Weaviate `1.14` significantly improves the performance of data imports for large datasets.

Note that the performance improvements should be noticeable for imports of over 10 million objects. Furthermore, this update enables you to import over 200 million objects into the database.

#### Problem
Before, the HNSW index would grow in constant intervals of 25,000 objects. This was fine for datasets under 25 million objects. But once the database got to around 25 million objects, adding new objects would be significantly slower. Then from 50–100m, the import process would slow down to a walking pace.

#### Solution
To address this problem, we changed how the HNSW index grows. We implemented a relative growth pattern, where the HNSW index size increases by either 25% or 25’000 objects (whichever is bigger).

![HNSW index growth chart](/img/blog/weaviate-1.14/hnsw-index-growth.jpg)

#### Test
After introducing the relative growth patterns, we've run a few tests.
We were able to import 200 million objects and more, while the import performance remained constant throughout the process.

[See more on github](https://github.com/semi-technologies/weaviate/pull/1976){:target="_blank"}.

### Full changelog

These are few of the many improvements and bug fixes that were included in this release.

Check out [the changelog](https://github.com/semi-technologies/weaviate/releases/tag/v1.14.0){:target="_blank"} to see the complete list.

## Monitoring and Observability

![Monitoring and Observability](/img/blog/weaviate-1.14/monitoring-and-observability.png)

One of the biggest challenges of running software in production is to understand what is happening under the hood.
That is especially important when something goes wrong, or we need to anticipate in advance when more resources are required.

![It doesn't work.... why?](/img/blog/weaviate-1.14/what-is-happening.jpg)
Without such insight, we end up looking at the black box, wondering what is going on.

### Announcement

With Weaviate `1.14` you can get a lot more insight into the resources and the performance of different aspects of your Weaviate instance in Production.

Now, you can expose Prometheus-compatible metrics for monitoring. Combine this with a standard Prometheus/Grafana setup to create visual dashboards for metrics around latencies, import speed, time spent on vector vs object storage, memory usage, and more.

<!-- TODO: replace the image link -->
<!-- ![Importing Data into Weaviate](/img/weaviate-sample-dashboard-importing.png "Importing Data Into Weaviate") -->

![Importing Data into Weaviate](https://raw.githubusercontent.com/semi-technologies/weaviate-io/docs/v1.14.0/img/weaviate-sample-dashboard-importing.png "Importing Data Into Weaviate")

### Example

In a hypothetical scenario, you might be importing a large dataset. At one point the import process might slow down. You could then check your dashboards, where you might see that the vector indexing process is still running fast, while the object indexing slowed down. <br/>
Then you could cross-reference it with another dashboard, to see that the slow down began when the import reached 120 million objects.<br/>
In two steps, you could narrow down the issue to a specific area, which would get you a lot closer to finding the solution. Or you could use that data to share it with the Weaviate team to get help.

### Try it yourserlf

Here is an [example project](https://github.com/semi-technologies/weaviate-examples/tree/main/monitoring-prometheus-grafana){:target="_blank"}, it contains:

* `docker-compose.yml` that spins up Weaviate (without any modules),
* a **Prometheus** instance,
* and a **Grafana** instance.

Just spin everything up, run a few queries and navigate to the Grafana instance in the browser to see the dashboard.

### Learn more

To learn more, see the [documentation](/developers/weaviate/current/more-resources/monitoring.html){:target="_blank"}.

## Support for non-cosine distances

![Support for non-cosine distances](/img/blog/weaviate-1.14/non-cosine-distances.png)

Weaviate v1.14 adds support for **L2** and **Dot Product** distances.<br/>
With this, you can now use datasets that support Cosine, L2 or Dot distances. This opens up a whole new world of use cases that were not possible before.<br/>
Additionally, this is all pluggable and very easy to add new distance metrics in the future.
### Background
In the past Weaviate used a single number that would control the distances between vectors and that was **certainty**. A certainty is a number between 0 and 1, which works perfectly for cosine distances, as cosine distances are limited to 360° and can be easily converted to a range of 0-1.

![L2 and Dot Product distance calculations](/img/blog/weaviate-1.14/distances.png)

However, some machine learning models are trained with other distance metrics, like L2 or Dot Product. If we look at euclidean-based distances, two points can be infinitely far away from each other, so translating that to a bound certainty of 0-1 is not possible.

### What is new
For this reason, we introduced a new field called **distance**, which you can choose to be based on L2 or Dot Product distances.

### Raw distance

The distance values provided are raw numbers, which allow you to interpret the results based on your specific use-case scenario.<br/>
For example, you can normalize and convert **distance values** to **certainty values** that fit the machine learning model you use and the kind of results you expect.

### Learn more

For more info, check out [the documentation](/developers/weaviate/current/vector-index-plugins/distances.html){:target="_blank"}.

### Contribute

Adding other distances is surprisingly easy, which could be a great way to contribute to the Weaviate project.

If that is something up your street, check out [the distancer code on github](https://github.com/semi-technologies/weaviate/tree/master/adapters/repos/db/vector/hnsw/distancer){:target="_blank"}, to see the implementation of other metrics.

Just make sure to include plenty of tests. Remember: “reliability, reliability, reliability”.

## Updated API endpoints to manipulate data objects of specific class

![Updated API endpoints](/img/blog/weaviate-1.14/updated-API-endpoints.png)

The REST API CRUD operations now require you to use both an **object ID** and the target **class name**.<br/>
This ensures that the operations are performed on the correct objects.

### Background

One of Weaviate's features is full CRUD support. CRUD operations enable the mutability of data objects and their vectors, which is a key difference between a vector database and an ANN library. In Weaviate, every data object has an ID (UUID). This ID is stored with the data object in a key-value store. IDs don’t have to be globally unique, because in Weaviate [classes](/developers/weaviate/current/data-schema/schema-configuration.html#class-object){:target="_blank"} act as namespaces. While each class has a different [HNSW index](/developers/weaviate/current/vector-index-plugins/hnsw.html){:target="_blank"}, including the store around it, which is isolated on disk.

There was however one point in the API where reusing IDs between classes was causing serious issues. Most noticeable this was for the [v1/objects/{id}](/developers/weaviate/current/restful-api-references/objects.html){:target="_blank"} REST endpoints.
If you wanted to retrieve, modify or delete a data object by its ID, you would just need to specify the ID, without specifying the class name. So if the same ID exists for objects in multiple classes (which is fine because of the namespaces per class), Weaviate would not know which object to address and would address all objects with that ID instead. I.e. if you tried to delete an object by ID, this would result in the deletion of all objects with that ID.

### Endpoint changes

This issue is now fixed with a **change to the API endpoints**. To get, modify and delete a data object, you now need to provide both the ID and the class name.

The following object functions are changed: **GET**, **HEAD**, **PUT**, **PATCH** and **DELETE**. 

#### Object change
New
```
/v1/objects/{className}/{id}
```
Deprecated
```
/v1/objects/{id}
```

#### References change
New
```
v1/objects/{className}/{id}/references/{propertyName}
```
Deprecated
```
v1/objects/{id}/references/{propertyName}
```

### Client changes
There are also updates in the language clients, where you now should provide a class name for data object manipulation. Old functions will continue to work, but are considered deprecated, and you will see a deprecation warning message.

## Stronger together

Of course, making Weaviate more reliable would be a lot harder without the great community around Weaviate.
<br/>As it is often the case: *“You can’t fix issues you didn’t know you have”*.

<!-- ![You can’t fix issues you didn’t know you have](/img/blog/weaviate-1.14/you-cant-fix.jpg) -->

### Thank you
Thanks to many active members on Weaviate’s Community Slack and through GitHub issues, we were able to identify, prioritize and fix many more issues than if we had to do it alone.

> Together, we made Weaviate v1.14 <br/>the most stable release yet.

### Help us
If at any point during your journey with Weaviate, you discover any bugs or you have feedback to share, please don’t hesitate to reach out to us via:
* [Weaviate’s Slack](https://join.slack.com/t/weaviate/shared_invite/zt-goaoifjr-o8FuVz9b1HLzhlUfyfddhw){:target="_blank"} – you can join anytime
* [GitHub issues for Weaviate Core](https://github.com/semi-technologies/weaviate/issues/new){:target="_blank"}
* [GitHub issues for Weaviate’s documentation](https://github.com/semi-technologies/weaviate-io/issues/new/choose){:target="_blank"}

We have a little favour to ask. Often reproducing the issue takes a lot more time than it takes to fix it. When you report a new issue, please include steps on how to reproduce the issue, together with some info about your environment. That will help us tremendously to identify the root cause and fix it.

### Guide
Here is a little guide on [how to write great bug reports](/developers/weaviate/current/tutorials/write-great-bug-reports.html){:target="_blank"}.

## Enjoy

We hope you enjoy the most reliable and observable Weaviate release yet!

Please share your feedback with us via [Slack](https://join.slack.com/t/weaviate/shared_invite/zt-goaoifjr-o8FuVz9b1HLzhlUfyfddhw){:target="_blank"}, [Twitter](https://twitter.com/SeMI_tech){:target="_blank"}, or [Github](https://github.com/semi-technologies/weaviate){:target="_blank"}.