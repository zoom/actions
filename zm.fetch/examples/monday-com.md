This is an example zm.fetch structure to **create item (GraphQL)** for Monday.com.

```JavaScript
zm.log.info('Creating Monday.com item');

const { boardId, groupId, itemName, columnValues } = ctx.inputData;

const query = `
    mutation {
        create_item (
            board_id: ${boardId.value},
            group_id: "${groupId.value}",
            item_name: "${itemName.value}",
            column_values: "${columnValues.value}"
        ) {
            id
            name
        }
    }
`;

const result = await zm.fetch({
    url: '/v2',
    method: 'POST',
    data: {
        query: query
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Monday.com item created: {}', body.data.create_item.id);
        return { itemId: body.data.create_item.id, itemName: body.data.create_item.name };
    },
    (code, message, body) => {
        zm.log.error('Monday.com item creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
