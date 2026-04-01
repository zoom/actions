This document explains how to use zm.fetch in Zoom custom actions to call third-party endpoints. It covers script syntax, how to populate headers, data, and parameters using different structures, and includes complete examples with sample endpoints.  

**This information is supplemental to the Zoom developer's documentation for [Actions](https://developers.zoom.us/docs/build-flow/actions/#transformations).**  
## Contents 
1. [Prerequisite](#prerequisite)
2. [Understanding zm.fetch structure](#understanding-zmfetch-structure)
3. [Context objects](#context-objects)
4. [Basic GET requests](#basic-get-requests)
5. [POST requests with data](#post-requests-with-data)
6. [PUT and PATCH requests](#put-and-patch-requests)
7. [Query parameters](#query-parameters)
8. [Response handling](#response-handling)
9. [Best practices](#best-practices)
10. [Quick reference](#quick-reference)

## Prerequisite
Third-party endpoints and URLs are defined in [Connect](https://developers.zoom.us/docs/build-flow/connect/). 

## Understanding zm.fetch structure 

### Key concepts 
1. **Authentication is handled automatically**  
   You do not need to pass auth headers or tokens. Zoom handles authentication in the background based on your app's Connect configuration.
2. **Endpoints must be pre-defined**  
  Every URL you call via `zm.fetch` must already be defined in your app's Connect configuration pages.
3. **All non-GET requests require a `method` param**  
   You must explicitly specify `method: POST`, `method: PUT`, `method: PATCH`, or `method: DELETE`.  
   GET requests do not require a method param.
4. **Pass objects directly to data**  
   JSON.stringify() is not needed.

### zm.fetch Signature  
```JavaScript
const result = await zm.fetch({
    url: '/your-preconfigured-endpoint',    // URL (required) — must be defined in Connect config
    method: 'POST',                          // HTTP method (required for non-GET)
    headers: { },                            // Non-auth headers (optional)
    data: { },                               // Request body as object (optional)
    params: { }                              // URL query params as object (optional)
});
```
## Context objects  
Your action scripts have access to two context objects for retrieving input values.  

### ctx.inputData (Object)  
An object containing your action's input fields. Each field is itself an object. Use `.value` to get the actual value.  

```JavaScript
// Destructure the input fields, then access .value on each
const { name, email, priority } = ctx.inputData;

// Use .value to get the actual string/number values
zm.log.info('Name is: {}', name.value);
zm.log.info('Email is: {}', email.value);
zm.log.info('Priority is: {}', priority.value);
```
### ctx.customField (Array)  
An array of custom field objects defined in your action configuration. Since it's an array, you need to find the field you want by its ID.  
Example:
```plaintext
{
    "action_id": "myFieldName",
    "type": "string",
    "value": "the field value"
}
```

```JavaScript
// Helper: Map the array into an object for convenient access
const customFields = {};
ctx.customField.forEach(field => {
    customFields[field.action_id] = field.value;
});

// Now access fields easily
const projectId = customFields.projectId;
const defaultPriority = customFields.defaultPriority;
```
Or access individual fields directly from the array:  
```JavaScript
// Find a specific custom field by ID
const projectId = ctx.customField.find(f => f.action_id === 'projectId')?.value;

```
## Basic GET requests  
**Note**: GET requests do not require a `method` param.   

### Simple GET request  
```JavaScript
zm.log.info('Fetching user profile');

const { userId } = ctx.inputData;

const result = await zm.fetch({
    url: `/users/${userId.value}`
});

return zm.handleResponse(result,
    body => {
        zm.log.info('User data retrieved: {}', body.name);
        return body;
    },
    (code, message, body) => {
        zm.log.error('Failed to fetch user. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
### GET with query parameters  
```JavaScript
zm.log.info('Fetching filtered products');

const { category, minPrice, maxPrice } = ctx.inputData;

const result = await zm.fetch({
    url: '/products',
    params: {
        category: category.value,
        min_price: minPrice.value,
        max_price: maxPrice.value,
        limit: 20
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Found {} products', body.items.length);
        return { products: body.items };
    },
    (code, message, body) => {
        zm.log.error('Product fetch failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
## POST Requests with data  
**Note:** `method: POST` is required for all `POST` requests.  

### Create new resource

```JavaScript
zm.log.info('Creating new task');

const { taskTitle, taskDescription, assigneeId } = ctx.inputData;

const result = await zm.fetch({
    url: '/tasks',
    method: 'POST',
    data: {
        title: taskTitle.value,
        description: taskDescription.value,
        assignee_id: assigneeId.value,
        priority: 'high',
        status: 'open'
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Task created with ID: {}', body.id);
        return {
            taskId: body.id,
            taskUrl: body.url
        };
    },
    (code, message, body) => {
        zm.log.error('Task creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
### POST with custom fields as defaults  
```JavaScript
zm.log.info('Creating item with default values from custom fields');

const { itemName, itemDescription } = ctx.inputData;

// Map custom field array for convenient access
const customFields = {};
ctx.customField.forEach(field => {
    customFields[field.fieldId] = field.value;
});

const result = await zm.fetch({
    url: `/workspaces/${customFields.workspaceId}/items`,
    method: 'POST',
    data: {
        name: itemName.value,
        description: itemDescription.value,
        category: customFields.defaultCategory
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Item created: {}', body.id);
        return {
            itemId: body.id,
            itemName: body.name
        };
    },
    (code, message, body) => {
        zm.log.error('Item creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
); 
```

## PUT and PATCH requests  
###  Full update with PUT  
```JavaScript
zm.log.info('Updating customer record');

const { customerId, email, phone, status } = ctx.inputData;

const result = await zm.fetch({
    url: `/customers/${customerId.value}`,
    method: 'PUT',
    data: {
        email: email.value,
        phone: phone.value,
        status: status.value,
        updated_at: new Date().toISOString()
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Customer {} updated successfully', customerId.value);
        return body;
    },
    (code, message, body) => {
        zm.log.error('Customer update failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```

### Partial update with PATCH  
```JavaScript
zm.log.info('Updating ticket status');

const { ticketId, newStatus, assignee } = ctx.inputData;

const result = await zm.fetch({
    url: `/tickets/${ticketId.value}`,
    method: 'PATCH',
    data: {
        status: newStatus.value,
        assigned_to: assignee.value
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Ticket {} updated to status: {}', ticketId.value, newStatus.value);
        return body;
    },
    (code, message, body) => {
        zm.log.error('Ticket update failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
### DELETE resource  
```JavaScript
zm.log.info('Deleting record');

const { recordId } = ctx.inputData;

const result = await zm.fetch({
    url: `/records/${recordId.value}`,
    method: 'DELETE'
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Record {} deleted successfully', recordId.value);
        return { deleted: true, recordId: recordId.value };
    },
    (code, message, body) => {
        zm.log.error('Record deletion failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```

## Query parameters  
### Search with multiple parameters  
```JavaScript
zm.log.info('Searching documents');

const { searchQuery, fileType, dateFrom, dateTo } = ctx.inputData;

const result = await zm.fetch({
    url: '/search',
    params: {
        q: searchQuery.value,
        type: fileType.value,
        date_from: dateFrom.value,
        date_to: dateTo.value,
        sort: 'relevance',
        limit: 50
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Found {} documents', body.total);
        return {
            totalResults: body.total,
            documents: body.results
        };
    },
    (code, message, body) => {
        zm.log.error('Search failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
### Pagination parameters  
```JavaScript
zm.log.info('Fetching paginated list');

const { page, perPage } = ctx.inputData;

const result = await zm.fetch({
    url: '/items',
    params: {
        page: page.value || 1,
        per_page: perPage.value || 25,
        sort: 'created_at',
        order: 'desc'
    }
});

return zm.handleResponse(result,
    body => {
        return {
            items: body.items,
            totalPages: body.total_pages,
            currentPage: body.current_page
        };
    },
    (code, message, body) => {
        zm.log.error('List fetch failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
## Response handling  
(This `zm.handleResponse` method is a helper utility. It is not required, but we recommend it as a best practice.)  
The `zm.fetch method` returns a complete response structure with the HTTP status code. The `zm.handleResponse` method checks whether the response's HTTP status code is in the 2xx or 3xx range. If it is, the `success callback` function (the second parameter of the method) is called; otherwise, the `failure callback` function (the third parameter of the method) is called. 

**Note**: Make sure you map your script’s output response to a JSON object that matches your action’s output definition.  

## Best practices  
### Authentication  
**Do not add authentication headers.** Custom actions automatically handles authentication.

### Always specify method for non-GET requests
  ```JavaScript
  // ✅ GET — method not needed
   const result = await zm.fetch({ url: '/items' });

   // ✅ POST — method required
   const result = await zm.fetch({ url: '/items', method: 'POST', data: { name: 'New' } });

   // ✅ PUT — method required
   const result = await zm.fetch({ url: '/items/123', method: 'PUT', data: { name: 'Updated' } });

   // ✅ PATCH — method required
   const result = await zm.fetch({ url: '/items/123', method: 'PATCH', data: { status: 'done' } });

   // ✅ DELETE — method required
   const result = await zm.fetch({ url: '/items/123', method: 'DELETE' });
  ```
### Endpoints must be pre-configured  
   Every URL path used in `zm.fetch` must be defined in your app's [Connect](https://developers.zoom.us/docs/build-flow/connect/) configuration pages first. You cannot call arbitrary URLs.  
   
### Always use zm.handleResponse()
   ```JavaScript
   // ✅ Good — structured success and error handling
   return zm.handleResponse(result,
       body => body,
       (code, message, body) => {
           zm.log.error('Error: {}', message);
           return zm.errorResponse(code, message, body);
       }
   );
   ```
### Use structured logging  
```JavaScript
// ✅ Good — placeholders for structured logging
zm.log.info('Processing user {} with status {}', userId.value, status.value);
zm.log.error('Failed. Code: {}, message: {}', code, message);

// ❌ Bad — string concatenation
zm.log.info('Processing user ' + userId.value + ' with status ' + status.value);
```
### Use params for query parameters  
```JavaScript
// ✅ Good — use params object
const result = await zm.fetch({
    url: '/search',
    params: { q: searchTerm.value, page: 1 }
});

// ❌ Bad — manual URL construction
const result = await zm.fetch({
    url: `/search?q=${searchTerm.value}&page=1`
});
```
### Pass objects directly to data  
```JavaScript
// ✅ Good — plain object
data: { name: userName.value, email: userEmail.value }

// ❌ Bad — unnecessary stringification
data: JSON.stringify({ name: userName.value, email: userEmail.value })
```
### Access input values with .value  
```JavaScript
// ✅ Correct — each inputData field is an object, use .value
const { name, email } = ctx.inputData;
zm.log.info('Name: {}', name.value);
zm.log.info('Email: {}', email.value);

// ❌ Wrong — missing .value
zm.log.info('Name: {}', name);  // This is the field object, not the string
```
### Map custom fields array for easy access
```JavaScript
// ✅ Good — map once at the top, use throughout
const customFields = {};
ctx.customField.forEach(field => {
    customFields[field.fieldId] = field.value;
});
const projectId = customFields.projectId;

// ✅ Also good — direct find for one-off access
const projectId = ctx.customField.find(f => f.fieldId === 'projectId')?.value;
```
   
## Quick reference  

Replace placeholder URLs and field names with your actual pre-configured endpoints and field IDs. Every endpoint used in `zm.fetch` must be defined in your app's [Connect](https://developers.zoom.us/docs/build-flow/connect/) configuration pages.

| Feature | Syntax |
|---------|--------|
| GET request | `zm.fetch({ url: '/endpoint' })` |
| POST request | `zm.fetch({ url: '/endpoint', method: 'POST', data: { } })` |
| PUT request | `zm.fetch({ url: '/endpoint', method: 'PUT', data: { } })` |
| PATCH request | `zm.fetch({ url: '/endpoint', method: 'PATCH', data: { } })` |
| DELETE request | `zm.fetch({ url: '/endpoint', method: 'DELETE' })` |
| Query params | `zm.fetch({ url: '/endpoint', params: { key: 'value' } })` |
| Non-auth headers | `zm.fetch({ url: '/endpoint', headers: { 'Notion-Version': '2022-06-28' } })` |
| Input data value | `ctx.inputData.fieldName.value` |
| Custom field (find) | `ctx.customField.find(f => f.fieldId === 'myField')?.value` |
| Success handler | `zm.handleResponse(result, body => { }, (code, msg, body) => { })` |
| Error response | `zm.errorResponse(code, message, body)` |
| Log info | `zm.log.info('Message {}', variable)` |
| Log error | `zm.log.error('Error {}', variable)` |













