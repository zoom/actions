This is an example zm.fetch structure to **send SMS** for Twilio. 

```JavaScript
zm.log.info('Sending SMS via Twilio');

const { toNumber, fromNumber, messageBody } = ctx.inputData;

// Access account SID from custom fields array
const accountSid = ctx.customField.find(f => f.fieldId === 'twilioAccountSid')?.value;

const result = await zm.fetch({
    url: `/2010-04-01/Accounts/${accountSid}/Messages.json`,
    method: 'POST',
    data: {
        To: toNumber.value,
        From: fromNumber.value,
        Body: messageBody.value
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('SMS sent: {}', body.sid);
        return { messageSid: body.sid, status: body.status };
    },
    (code, message, body) => {
        zm.log.error('Twilio SMS send failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
