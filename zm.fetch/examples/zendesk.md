This is an example zm.fetch structure to **create support ticket** for Zendesk. 

```JavaScript
zm.log.info('Creating Zendesk ticket');

const { subject, commentBody, priority, requesterName, requesterEmail } = ctx.inputData;

const result = await zm.fetch({
    url: '/api/v2/tickets',
    method: 'POST',
    data: {
        ticket: {
            subject: subject.value,
            comment: { body: commentBody.value },
            priority: priority.value || 'normal',
            status: 'open',
            requester: {
                name: requesterName.value,
                email: requesterEmail.value
            }
        }
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Zendesk ticket created: {}', body.ticket.id);
        return { ticketId: body.ticket.id, ticketUrl: body.ticket.url };
    },
    (code, message, body) => {
        zm.log.error('Zendesk ticket creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```

