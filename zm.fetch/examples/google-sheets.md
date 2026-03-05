This is an example zm.fetch structure to **append row** for Google Sheets. 

```JavaScript
zm.log.info('Appending row to Google Sheet');

const { spreadsheetId, range, column1, column2, column3 } = ctx.inputData;

const result = await zm.fetch({
    url: `/v4/spreadsheets/${spreadsheetId.value}/values/${range.value}:append`,
    method: 'POST',
    params: {
        valueInputOption: 'USER_ENTERED'
    },
    data: {
        values: [[column1.value, column2.value, column3.value]]
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Row appended: {} rows updated', body.updates.updatedRows);
        return { updatedRows: body.updates.updatedRows, updatedRange: body.updates.updatedRange };
    },
    (code, message, body) => {
        zm.log.error('Google Sheets append failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
