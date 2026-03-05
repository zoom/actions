This is an example zm.fetch structure to **create a repository issue** in GitHub.  

```JavaScript
zm.log.info('Creating GitHub issue');

const { owner, repo, title, issueBody, labels } = ctx.inputData;

const result = await zm.fetch({
    url: `/repos/${owner.value}/${repo.value}/issues`,
    method: 'POST',
    headers: {
        'Accept': 'application/vnd.github.v3+json'
    },
    data: {
        title: title.value,
        body: issueBody.value,
        labels: labels.value ? labels.value.split(',').map(l => l.trim()) : []
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('GitHub issue created: #{}', body.number);
        return { issueNumber: body.number, issueUrl: body.html_url, state: body.state };
    },
    (code, message, body) => {
        zm.log.error('GitHub issue creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
