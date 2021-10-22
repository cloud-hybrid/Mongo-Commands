# Mongo-Commands #

## Configuration ##

**Checking Database**

```js
db.runCommand( { getParameter : 1, "internalQueryExecMaxBlockingSortBytes" : 1 } )
```
