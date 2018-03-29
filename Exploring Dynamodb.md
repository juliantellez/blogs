# Exploring Dynamodb

```
aws dynamodb help
```

Recently I've been looking into incorporating a db for this very blog, while there are tons of options out there, most solutions require an administrative and operating effort that add extra complexity to any project.

Users these days demand low latencies and high availability which makes scalability sort of a hot topic. What they don't tell you is that scaling a distributed database is kind of hard, and the planning, investment and overhead involved is what makes solutions such as dynamodb a desirable commodity.

Dynamodb automatically distributes your traffic and data over a sufficient number of servers to handle your throughput and storage requirements, all of this while maintaining a consistent and fast performance.

```
aws dynamodb create-table
```

<img src="https://cdn-images-1.medium.com/max/2000/1*7EF1S0AxiDPBKNeVdVDsTQ.png" alt="Drawing" style="width: 500px;"/>