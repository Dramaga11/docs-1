<!-- markdownlint-disable MD041 MD002 -->

### Make a Call to Your API

To make calls to your API, you need an Access Token. You can get an Access Token for testing purposes from the **Test** view in your <a href="${manage_url}/#/apis" target="_blank" rel="noreferrer">API settings</a>.

![Obtain a JWT](/media/articles/server-apis/aspnet-core-webapi/request-access-token.png)

Provide the Access Token as an `Authorization` header in your requests.

```har
{
  "method": "GET",
  "url": "http://your-domain.com/api_path",
  "headers": [
    { "name": "Authorization", "value": "Bearer YOUR_ACCESS_TOKEN_HERE" }
  ]
}
```
