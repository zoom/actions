This is an example zm.fetch structure to **create product** for Shopify. 

```JavaScript
zm.log.info('Creating Shopify product');

const { productTitle, productDescription, vendor, price, sku } = ctx.inputData;

const result = await zm.fetch({
    url: '/admin/api/2024-01/products.json',
    method: 'POST',
    data: {
        product: {
            title: productTitle.value,
            body_html: productDescription.value,
            vendor: vendor.value,
            variants: [{
                price: price.value,
                sku: sku.value,
                inventory_management: 'shopify'
            }]
        }
    }
});

return zm.handleResponse(result,
    body => {
        zm.log.info('Shopify product created: {}', body.product.id);
        return {
            productId: body.product.id,
            productTitle: body.product.title,
            productUrl: body.product.admin_graphql_api_id
        };
    },
    (code, message, body) => {
        zm.log.error('Shopify product creation failed. Code: {}, message: {}', code, message);
        return zm.errorResponse(code, message, body);
    }
);
```
