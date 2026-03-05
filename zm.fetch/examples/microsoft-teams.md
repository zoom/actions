This is an example zm.fetch structure to **post to channel webhook** in Microsoft Teams. 

```JavaScript
zm.log.info('Posting to Microsoft Teams');

const { title, messageText } = ctx.inputData;

const result = await zm.fetch({
    url: '/webhook-endpoint',
    method: 'POST',
    data: {
        '@type': 'MessageCard',
        '@context': 'https://schema.org/extensions',
        summary: title.value,
        themeColor: '0078D7',
        title: title.value,
        sections: [{
            activityTitle: messageText.value,
            facts: []
        }]
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Teams message posted successfully');
        return { success: true };
    },
    (code, message, body) => {
        zm.log.error('Teams post failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
