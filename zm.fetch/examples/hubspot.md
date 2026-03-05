This is an example zm.fetch structure to **create contact** for HubSpot. 

```JavaScript
zm.log.info('Creating HubSpot contact');

const { email, firstName, lastName, phone, company } = ctx.inputData;

const result = await zm.fetch({
    url: '/crm/v3/objects/contacts',
    method: 'POST',
    data: {
        properties: {
            email: email.value,
            firstname: firstName.value,
            lastname: lastName.value,
            phone: phone.value,
            company: company.value,
            lifecyclestage: 'lead'
        }
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('HubSpot contact created: {}', body.id);
        return { contactId: body.id, email: body.properties.email };
    },
    (code, message, body) => {
        zm.log.error('HubSpot contact creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
