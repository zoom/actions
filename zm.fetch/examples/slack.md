This is an example zm.fetch structure to **Post a message to channel** in Slack.  

```JavaScript
zm.log.info('Posting message to Slack');

const { channelId, messageText } = ctx.inputData;

const result = await zm.fetch({
    url: '/api/chat.postMessage',
    method: 'POST',
    data: {
        channel: channelId.value,
        text: messageText.value,
        blocks: [
            {
                type: 'section',
                text: {
                    type: 'mrkdwn',
                    text: `*Meeting Alert*\n${messageText.value}`
                }
            },
            {
                type: 'actions',
                elements: [
                    {
                        type: 'button',
                        text: { type: 'plain_text', text: 'View Details' },
                        url: 'https://example.com/details'
                    }
                ]
            }
        ]
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Slack message posted: {}', body.ts);
        return { messageId: body.ts, channel: body.channel };
    },
    (code, message, body) => {
        zm.log.error('Slack post failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
