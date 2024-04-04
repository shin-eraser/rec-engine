<p><a target="_blank" href="https://app.eraser.io/workspace/pgkd7DLmFDpGJKYB2HSC" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

Imagine you work at **Makers and Markers**, an online marketplace for artisanal whiteboards and whiteboard-related products. The marketplace has really taken off, and users are asking for a richer search experience to help them find what they're looking for. You definitely need the raw product information to be searchable, and also may want to add other data, like product reviews and ratings.

![Kubernetes Diagram](undefined "Kubernetes Diagram")

You've decided that it's time to employ a specialized search database or service such as Elasticsearch. This leaves you with an important question: **how to get all of our data indexed and available for search?**

This article covers three approaches, exploring the trade-offs for each:

1. Dual writes
2. Batch jobs
3. Database change streaming
# Dual Writes
### What it is
Every time you make a change (create, update, or delete) to the primary datastore, the same service also writes the relevant changes directly to the search DB.

![Dual Write](undefined "Dual Write")

### Prerequisites
For dual write to work, writes to the ProductsDB should be centralized within a single service, whether it is a true microservice or a module within a monolith. If you are writing a lot of raw DB queries or accessing low level ORMs across your codebase, it will be a nightmare to ensure that all writes are coupled.

### The Good
The primary advantage here is **simplicity**. Because it requires no additional architecture (aside from the SearchDB itself, which we'll need to set up in all cases), we can:

- Quickly build a product to test (be it internally or with your users)
- Avoid trapdoor decisions that would be harder to undo
- Use existing logging and monitoring tools to support your roll out
- Use existing tests and test infrastructure
**Real-time** updates also come for free with this approach. As soon as a product description changes, our users can search for it (*_subject to _[﻿eventual consistency](https://www.scylladb.com/glossary/eventual-consistency/)  ).
### The Bad and the Ugly
One downside of having no additional architecture is that we have tough choices with **error handling. **What do we do if the write to the SearchDB fails? What if we're inside a transaction? Handling those can be complex and testing them is non-trivial. Any **durability** comes with **increased user-facing latency. **If we retry on failures and the SearchDB is slow or overwhelmed, the product experience suffers whenever a vendor tries to update a product or a user tries to leave a review because the primary service needs to wait.

**Scaling your code **can also be hard.** **Let's say we want to add more data to search: user reviews, average star ratings, vendor metadata. We'll need to add dual write code in all those services. And if we want to add search to other items, it can add up. 

Harder still is supporting more complicated writes. Let's say we want to support bulk updates for our vendors. We might end up with some code that needs to execute a query like this:

```
UPDATE products
SET is_scented = TRUE
WHERE vendor_id = 'vendor123'
AND product_category = 'marker'
```
Figuring out how to covert that into a bulk write to our SearchDB introduces new complexity and surface area for bugs (which products where changed? what exactly was set?)

---

# Batch Jobs
### What it is
A separate process that queries for all recently changed records and updates them:

![Batch job](undefined "Batch job")

### Prerequisites
In order for this to work, we'll just need to make sure our database records have an `updatedAt` field that is consistently changed. 

![Scaling our batch service](undefined "Scaling our batch service")

### The Good
It is **modular. **We can add a CSV upload feature to our vendor portal for bulk creates or updates of products, and not have to touch this part of the architecture. We could decide that we want our products to be in multiple categories and split `product.category` into a separate `category` table with a `product_category` join table, and we would only need to change a single query for fetching the new data.

We also have the **flexibility** to add more advanced logic over time. Some of the products listed on Makers and Markers have thousands of reviews! Instead of adding all of the raw review text, our product team wants to run some expensive processing before we update the indexed record.

We can make this service **durable **without sacrificing latency. If a write to the SearchDB fails, we can retry it without affecting the user experience in the vendor portal.

Lastly, this approach has room to **scale. **If we run into compute or memory constraints, we can Get a Bigger Box. Scaling horizontally is less trivial - we may need to adapt our code or infrastructure:

### The Bad and the Ugly
One downside is that this is not **real-time.** If your cron job runs every hour, a product or review might need an hour to become searchable.

**Testing **and **debugging** are also harder. Let's say we hear from one of our top marker makers that none of their new products are showing up in search. The problem might be we changed our cron job but forgot to change the query. Or there's a bug in the new queue we implemented. Or maybe we just forgot to set `updatedAt` when we introduced our new CSV upload feature to the vendor portal. Multiple teams might need to get involved to trace down the source.

If your activity is very spiky, this approach can also introduce heavy **load** on our primary datastore during peak.

Lastly, if one of our Makers chooses to delete their Marker from our product and we actually delete a record in our DB, those won't show up in queries! Fixing that can be a major overhaul.

---

# Streaming Database Changes
### What it is
We subscribe to changes at the database level and either process them directly or put them onto a message queue such as RabbitMQ or Kafka. (To learn more, watch this [﻿excellent talk](https://www.youtube.com/watch?v=fU9hR3kiOK0))

### Prerequisites
The major limitation is that we need a database that supports this. Luckily, most popular DBs include a mechanism for this. For example:

- Postgres, (which we should [﻿probably be using](https://www.amazingcto.com/postgres-for-everything/)  ) via [﻿l﻿ogical replication](https://www.postgresql.org/docs/current/logical-replication.html)  
- MySQL via the [﻿bin log](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html)  
- MongoDB via [﻿change streams ](https://www.mongodb.com/docs/current/changeStreams/)  There are also many databases and services that provide real-time reactivity out of the box. If you're already using a tool like that, you should probably use that here!
### The Good
This is the best approach for **performance** and **scale **(fun fact: CDC is how databases implement replication, so you know it's built for speed!).

We can get **near** **real time **indexing - the only limitation is the latency in our pipeline, and not the frequency of our batch job.

This architecture is also **adaptable**. You can use it to update your caches, maintain materialize views, and more!

### The Bad and the Ugly
The major downside to this approach is that it requires a lot of work to deploy, configure and maintain the new services.

Depending on our **dev environment set-up**, it may be prohibitively expensive to run all of these services locally, mostly measured in the time spent trying to make sure they work on all machines. We may find ourselves needing to build tools to only selectively run parts of our stack.

These same concerns also makes **testing **and **debugging** these systems very difficult.

---

# TL;DR
Dual writes may not be the sexiest approach, but don't sleep on it as an initial approach that can last a while and avoid trapdoor decisions.

This is the best approach if:

- We want to quickly build a feature that supports real-time search
- We aren't sure what the long term needs are from a product and / or scale perspective
- We are willing to tolerate some failure cases or occasional latency issues in our primary service
- Our updates are mostly single-item updates 
- We don't want to deploy additional infrastructure
For more mature products or to support more complex features, batch jobs that periodically query and index data might be a good middle ground. This is the best approach if:
- We need to combine data from different sources that may be updated in many ways
- We are willing to tolerate some amount of latency
- We aren't concerned about spiky activity and our primary datastore doesn't suffer from performance issues 
- We don't hard delete data from our primary datastore!
And once Makers and Markers moves into the mainstream, it may be time to look into tapping directly into our database's change log. This approach is best if:
- We want to support a number of other complex querying and reporting use cases with a single architectural approach
- We need to support high-throughput scale while still maintaining low latency
![Batch job](/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---figure---CwWD1lcjJ-5RjvxWZFZT8---figure---BCN2n_Ozx69KwSvEnzo8NA.png "Batch job")




<!-- eraser-additional-content -->
## Diagrams
<!-- eraser-additional-files -->
<a href="/Search Indexing-cloud-architecture-1.eraserdiagram" data-element-id="CM5mBMuwy_d6MWmwQ-gX-"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----8367fa6eca43af7f9e0c0cf86b1dd4e1.png" alt="" data-element-id="CM5mBMuwy_d6MWmwQ-gX-" /></a>
<a href="/Search Indexing-cloud-architecture-2.eraserdiagram" data-element-id="wN4BjVLkZf3ZcxUbqFrnp"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----d43561297f49f7223d0e9ab95f025c6d.png" alt="" data-element-id="wN4BjVLkZf3ZcxUbqFrnp" /></a>
<a href="/Search Indexing-cloud-architecture-3.eraserdiagram" data-element-id="LTr9lvaJTQwi4LPWmhqlx"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----1251132c191aeea76cecd4fa597ea1c2.png" alt="" data-element-id="LTr9lvaJTQwi4LPWmhqlx" /></a>
<a href="/Search Indexing-cloud-architecture-4.eraserdiagram" data-element-id="2A-JoOdemLazssQUWYccu"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----8f6822d300a66cd9d2d94772a61009f0.png" alt="" data-element-id="2A-JoOdemLazssQUWYccu" /></a>
<a href="/Search Indexing-cloud-architecture-5.eraserdiagram" data-element-id="j71SW5eVzOyPgjcjo_iDd"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----41ec3c1f81787a682aed2277c01c9cac.png" alt="" data-element-id="j71SW5eVzOyPgjcjo_iDd" /></a>
<a href="/Search Indexing-cloud-architecture-6.eraserdiagram" data-element-id="EH0kVLp_Xmnt21fEAAt_z"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----5f3d3cb8ae9b42eb0d2667391175c0e8.png" alt="" data-element-id="EH0kVLp_Xmnt21fEAAt_z" /></a>
<a href="/Search Indexing-cloud-architecture-7.eraserdiagram" data-element-id="uNBRwYv_4Fdza0zQM7BRk"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----021390a4e149defb8956774d6cd5431d.png" alt="" data-element-id="uNBRwYv_4Fdza0zQM7BRk" /></a>
<a href="/Search Indexing-sequence-diagram-8.eraserdiagram" data-element-id="J-Btd_85RcFD-aU6odkYc"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----dc7d5d76c79b54dcb909849d85263fd6.png" alt="" data-element-id="J-Btd_85RcFD-aU6odkYc" /></a>
<a href="/Search Indexing-entity-relationship-9.eraserdiagram" data-element-id="VxKmuux6eWvrpW0v-IiUR"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----96579b00c6de1dcea7933f206b281a90.png" alt="" data-element-id="VxKmuux6eWvrpW0v-IiUR" /></a>
<a href="/Search Indexing-Auth0 Authorization Code Flow-10.eraserdiagram" data-element-id="vO3MmJiO-bCzmZiBV3Gyx"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----866b653dfd87ddd425f7ed3e572cae95-Auth0-Authorization-Code-Flow.png" alt="" data-element-id="vO3MmJiO-bCzmZiBV3Gyx" /></a>
<a href="/Search Indexing-cloud-architecture-11.eraserdiagram" data-element-id="rKIfNL5JdjN4BLzSplRcm"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----097217e93dc3be010f7686558aa2c887.png" alt="" data-element-id="rKIfNL5JdjN4BLzSplRcm" /></a>
<a href="/Search Indexing-Music Database Schema-12.eraserdiagram" data-element-id="3-9kj-vfbwrJTb2Dbq7W6"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----f0800f2940e081b53e2f4c4ee1a877a8-Music-Database-Schema.png" alt="" data-element-id="3-9kj-vfbwrJTb2Dbq7W6" /></a>
<a href="/Search Indexing-cloud-architecture-13.eraserdiagram" data-element-id="owdHA1luBSX5h9Y2Q-7D5"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----a4b34b73ba89e4959d45ed5e14d48f5e.png" alt="" data-element-id="owdHA1luBSX5h9Y2Q-7D5" /></a>
<a href="/Search Indexing-User Authentication Flow-14.eraserdiagram" data-element-id="tvvA3QaerwmZQdgOEbhWD"><img src="/.eraser/pgkd7DLmFDpGJKYB2HSC___reS6fUv66LcKWYn8yV2OvCPvwSm2___---diagram----11efe0831296fda6040b89516cada12a-User-Authentication-Flow.png" alt="" data-element-id="tvvA3QaerwmZQdgOEbhWD" /></a>
<!-- end-eraser-additional-files -->
<!-- end-eraser-additional-content -->
<!--- Eraser file: https://app.eraser.io/workspace/pgkd7DLmFDpGJKYB2HSC --->