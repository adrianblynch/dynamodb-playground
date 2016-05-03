# DynamoDB playground

## Notes

Watching this video, [AWS re:Invent 2015 | (DAT401) Amazon DynamoDB Deep Dive](https://www.youtube.com/watch?v=ggDIat_FZtA), I made these notes:

### Limits

- Items have a maximum size of 400KB
- Local Secondary Indexes (LSI) have a limit of 10GB per partition
- Write Capacity Units (WCU) are 1KB/s
- Read Capacity Units (RCU) are 4KB/s

### Costs

- Eventually consistent reads cost 1/2 as much as consistent reads

### Performance

> To get the most out of DynamoDB, the hash key element has a large number of distinct values and those values are requested fairly uniformly, as randomly as possible

- Unused capacity is stored for 5 minutes to help deal with peaks that exceed the threshold
- Replace filters with indexes. A filter happens after the reads, effecting capacity
- To avoid hot keys due to popularity, randomise the key: `"ComposerA_" + RAND(0, 10)`

### Examples

#### Users accessed by ID and email

`user` table - Hash key: `id` - Attributes: `email`, `name`, etc.

`user-email` GSI - Hash key: `email` - Attributes: `id`, `name`, etc.

#### Storing time series data

- Creat tables for time periods
- Create tables ahead of time or dynamically
- De-provision capacity on older tables to save on cost

#### Hot keys or data

- Cache with [ElastiCache](https://aws.amazon.com/elasticache/)
- Use streams with [Lambda](https://aws.amazon.com/lambda/) to update the cache on significant data change

### General

- LSI and GSIs can't on be on document attributes.
- To get around the above, stream to a derived table and query that instead.
- Design the table for access, keeping in mind the W/RCUs for each usecase
- Does a request need all data? E.g. Listing someone's inbox doesn't need the message body. Move to another table and get when needed
- Distribute large items (over tables or partitions?)
- Push attributes into GSIs (and LSIs?) to speed up retrieval
- Sparse indexes don't contain "null" attributes, keeping the size down
- [Regional replication is now possible](https://aws.amazon.com/about-aws/whats-new/2015/07/amazon-dynamodb-available-now-cross-region-replication-triggers-and-streams/). Done with streams

## Issue

- Updating a nested document (map) requires an expression if patch like behaviour is needed.
- If the above is used, the map (or array) needs to be in place before acting on it.
- Defaulting to an empty map or array can mitigate this but isn't ideal.

## Local DynamoDB

Download a [local copy of DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html).

View data locally with [dynamodb-gui](https://www.npmjs.com/package/dynamodb-gui). Requires changes to `/api/common/dynamodb.js` to point to the local endpoint, `http://localhost:8000`.

Start script: `java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb`

Issues with the [local version of DynamoDB](https://github.com/mhart/dynalite#problems-with-amazons-dynamodb-local).

## Tools

[Table copy utils](https://github.com/awslabs/dynamodb-cross-region-library/tree/master/dynamodb-table-copy-utilities)
