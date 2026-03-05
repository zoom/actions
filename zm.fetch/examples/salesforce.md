This is an example zm.fetch structure to **query records** for SalesForce. 

```JavaScript
zm.log.info('Querying Salesforce records');

const { objectType, filterField, filterValue } = ctx.inputData;

const soqlQuery = `SELECT Id, Name FROM ${objectType.value} WHERE ${filterField.value} = '${filterValue.value}'`;

const result = await zm.fetch({
    url: '/services/data/v57.0/query',
    params: {
        q: soqlQuery
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Found {} Salesforce records', body.totalSize);
        return { totalRecords: body.totalSize, records: body.records };
    },
    (code, message, body) => {
        zm.log.error('Salesforce query failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```

