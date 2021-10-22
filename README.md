# Mongo-Commands #

- [General Settings](https://docs.mongodb.com/manual/reference/config-database/#mongodb-data-config.settings)
- [Operational Checklist](https://docs.mongodb.com/manual/administration/production-checklist-operations/)
- [Development Checklist](https://docs.mongodb.com/manual/administration/production-checklist-development/)
- [Configuration & Maintenence](https://docs.mongodb.com/manual/administration/configuration-and-maintenance/)
- [Limits and Thresholds](https://docs.mongodb.com/manual/reference/limits/)

## Aggregation Method(s) & Reference ##

- [MongoDB Documentation](https://docs.mongodb.com/manual/reference/aggregation/)

## Configuration ##

### Memory-related Areas of Concern ###

- Reference: https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/

While memory errors may not be related to sorting specifically, the following excerpt still holds true:

> If MongoDB requires using more than 100 megabytes of system memory for the blocking sort operation, MongoDB returns an error unless the query specifies `cursor.allowDiskUse()` (New in MongoDB 4.4). `allowDiskUse()` allows MongoDB to use temporary files on disk to store data exceeding the 100 megabyte system memory limit while processing a blocking sort operation.

The following command will show the current DB configuration that's used for controlling the amount of allocated RAM for blocking queries:

```js
db.runCommand( { getParameter : 1, "internalQueryExecMaxBlockingSortBytes" : 1 } )
```

Additionally, if memory errors are occuring, an appropriate solution would be to index commonly used queries:

The following documents introduce indexing strategies:

[Create Indexes to Support Your Queries](https://docs.mongodb.com/manual/tutorial/create-indexes-to-support-queries/)
- An index supports a query when the index contains all the fields scanned by the query. Creating indexes that support queries results in greatly increased query performance.
[Use Indexes to Sort Query Results](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/)
- To support efficient queries, use the strategies here when you specify the sequential order and sort order of index fields.
[Ensure Indexes Fit in RAM](https://docs.mongodb.com/manual/tutorial/ensure-indexes-fit-ram/)
- When your index fits in RAM, the system can avoid reading the index from disk and you get the fastest processing.
[Create Queries that Ensure Selectivity](https://docs.mongodb.com/manual/tutorial/create-queries-that-ensure-selectivity/)
- Selectivity is the ability of a query to narrow results using the index. Selectivity allows MongoDB to use the index for a larger portion of the work associated with fulfilling the query.

Sharding may also be of benefit depending on use case and cluster vs. singleton DB setup: https://docs.mongodb.com/manual/sharding/

#### Workarounds, Temporary Solutions ####

**Increasing Internal Query Buffer**

Conversion: `335544320B :== 320MB`

```js
db.adminCommand({setParameter: 1, internalQueryExecMaxBlockingSortBytes: 335544320})
```

**Enabling Temporary Writes to SWAP or Local File-System during Aggregation(s)**

```js
db.getCollection("[Collection-Name-Here]").aggregate( [
      { $sort : { [filter]: 1} }
   ],
   { allowDiskUse: true }
)
```

However, the more appropriate & performant solution:

```js
// 1. Create the Index
db.[Collection].createIndex( { "[Filter]": 1 } );

// 2. Test the Aggregate or otherwise Indexed Query
db.getCollection("[Collection]").find({}).sort({"[Filter]": 1});
```
