This is an example zm.fetch structure to **create a task in project** for Asana. 

```JavaScript
zm.log.info('Creating Asana task');

const { projectId, taskName, notes, dueDate, assigneeId } = ctx.inputData;

const result = await zm.fetch({
    url: '/api/1.0/tasks',
    method: 'POST',
    data: {
        data: {
            name: taskName.value,
            notes: notes.value,
            projects: [projectId.value],
            due_on: dueDate.value,
            assignee: assigneeId.value
        }
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Asana task created: {}', body.data.gid);
        return { taskId: body.data.gid, taskName: body.data.name, permalink: body.data.permalink_url };
    },
    (code, message, body) => {
        zm.log.error('Asana task creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
