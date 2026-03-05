This is an example zm.fetch structure to **create card on board** for Trello. 

```JavaScript
zm.log.info('Creating Trello card');

const { listId, cardName, description, position } = ctx.inputData;

const result = await zm.fetch({
    url: '/1/cards',
    method: 'POST',
    params: {
        idList: listId.value,
        name: cardName.value,
        desc: description.value,
        pos: position.value || 'top'
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Trello card created: {}', body.id);
        return { cardId: body.id, cardUrl: body.url, shortUrl: body.shortUrl };
    },
    (code, message, body) => {
        zm.log.error('Trello card creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```


