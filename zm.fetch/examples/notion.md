This is an example zm.fetch structure to **create page in database** for Notion. 

```JavaScript
zm.log.info('Creating Notion page');

const { databaseId, title, status, priority } = ctx.inputData;

const result = await zm.fetch({
    url: '/v1/pages',
    method: 'POST',
    headers: {
        'Notion-Version': '2022-06-28'
    },
    data: {
        parent: { database_id: databaseId.value },
        properties: {
            'Name': {
                title: [{ text: { content: title.value } }]
            },
            'Status': {
                select: { name: status.value }
            },
            'Priority': {
                select: { name: priority.value }
            }
        }
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Notion page created: {}', body.id);
        return { pageId: body.id, pageUrl: body.url };
    },
    (code, message, body) => {
        zm.log.error('Notion page creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);

```

