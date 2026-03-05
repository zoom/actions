This is an example zm.fetch structure to **create an issue** in Jira.  

```JavaScript
zm.log.info('Creating Jira issue');

const { projectKey, summary, description, issueType } = ctx.inputData;

const result = await zm.fetch({
    url: '/rest/api/3/issue',
    method: 'POST',
    data: {
        fields: {
            project: { key: projectKey.value },
            summary: summary.value,
            description: {
                type: 'doc',
                version: 1,
                content: [{
                    type: 'paragraph',
                    content: [{ type: 'text', text: description.value }]
                }]
            },
            issuetype: { name: issueType.value }
        }
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Jira issue created: {}', body.key);
        return { issueKey: body.key, issueId: body.id, issueUrl: body.self };
    },
    (code, message, body) => {
        zm.log.error('Jira issue creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
