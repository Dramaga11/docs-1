---
title: Troubleshooting
name: Shows how to troubleshoot the JWT middleware configuration
description: This document will help you troubleshoot your configuration if you get a 401 (Unauthorized) response from your API.
budicon: 500
topics:
    - quickstart
    - backend
    - webapi-owin
contentType: tutorial
useCase: quickstart
---

If you configured the JWT middleware correctly, you will be able to get proper responses from your API when you make requests. However, in the case where you get a 401 (Unauthorized) response from your API, it is because the configuration of your JWT middleware does not match with the JWT which was passed.

This document will help you troubleshoot your JWT middleware configuration.

## How does a token get validated?

In terms of validating a JWT, there are various things to consider:

1. **Is the token well-formed?** In other words, is this token conforming to the structure of a JSON Web Token (JWT)? To get more information on the structure of a JWT, please refer to <a href="/jwt#what-is-the-json-web-token-structure-" target="_blank" rel="noreferrer">this section on the structure of a JWT</a>

2. **Has the token been tampered with?** The last part of a JWT is the signature. The signature is used to verify that the token was in fact signed by the sender and not altered in any way.

3. **Has the token been received in its validity period?** JWTs are only valid for a specified time period (as expressed in the `exp` claim).

4. **Is the token coming from the intended Authority?** This consists of 2 parts

    * **Signature Verification**: Can we confirm that the JWT is correctly signed using the key issued by the issuing authority?

    * **Issuer Value**: The Issuer is defined in the `iss` claim. Once again does this claim match up with what your application expects?

5. **Is the token intended for the current application?** So does the `aud` claim of the JWT match with what your application is expecting?

## Inspecting a token

A quick way to inspect a JWT is by using the <a href="https://jwt.io/" target="_blank" rel="noreferrer">JWT.io</a> website. It has a handy debugger which allows you to quickly check that a JWT is well-formed, and also inspect the values of the various claims.

![Debugging a JWT on JWT.io](/media/articles/server-apis/aspnet-core-webapi/jwt-io-debugger-rs256.png)

In the screenshot above you can see that the token was signed using the **RS256** algorithm. The **Issuer** of the token is **https://jerrie.auth0.com/**, and the **Audience** is **https://quickstarts/api**.

So in other words these values in your JWT middleware registration must match **exactly** - including the trailing slash for the Issuer, such as

```csharp
var keyResolver = new OpenIdConnectSigningKeyResolver("https://jerrie.auth0.com/");
app.UseJwtBearerAuthentication(
    new JwtBearerAuthenticationOptions
    {
        AuthenticationMode = AuthenticationMode.Active,
        TokenValidationParameters = new TokenValidationParameters()
        {
            ValidAudience = "https://quickstarts/api",
            ValidIssuer = "https://jerrie.auth0.com/",
            IssuerSigningKeyResolver = (token, securityToken, kid, parameters) => keyResolver.GetSigningKey(kid)
        }
    });
```

For a token signed using HS256, the debugger view will look a little different:

![Debugging a JWT on JWT.io](/media/articles/server-apis/aspnet-core-webapi/jwt-io-debugger-hs256.png)

In the screenshot above you can see that the token was signed using the **HS256** algorithm. The **Issuer** of the token is **https://jerrie.auth0.com/**, and the **Audience** is **https://quickstarts/api/hs256**.

In this case, the middleware needs to be configured as follows:

```csharp
app.UseJwtBearerAuthentication(
    new JwtBearerAuthenticationOptions
    {
        AuthenticationMode = AuthenticationMode.Active,
        AllowedAudiences = new[] { "https://quickstarts/api/hs256" },
        IssuerSecurityKeyProviders = new IIssuerSecurityKeyProvider[]
        {
            new SymmetricKeyIssuerSecurityKeyProvider("https://jerrie.auth0.com/",
              Encoding.UTF8.GetBytes("your api secret"))
        }
    });
```

## Using the log files to debug configuration issues

In order to inspect the log message which are generated by the OWIN middleware, you need to set the logging level to Verbose. So in your `web.config` file, add the following:

```xml
<system.diagnostics>
  <switches>
    <add name="Microsoft.Owin" value="Verbose" />
  </switches>
</system.diagnostics>
```

This will log all OWIN log messages to the Output window in Visual Studio when running your application in Debug Mode.

### 1. Are you actually passing the JWT in the Authorization header?

The first thing is to ensure that you actually pass along the JWT as a Bearer token in the Authorization header of the request. If you get a 401 (Unauthorized) response from your Web API, but you do not see any other error messages being logged to the Output Window, then most likely you are simply not sending the JWT in the Authorization header of the request.

To resolve this issue, ensure that you send the JWT as a bearer token in the Authorization header.

## 2. Did you configure the JWT middleware for the correct signing algorithm?

Another common mistake is that your tokens are signed using the HS256 <a href="/tokens/concepts/signing-algorithms" target="_blank" rel="noreferrer">signing algorithm</a>, but your middleware is configured for RS256 - or vice versa.

In the following screenshot, you can see that we get an error message that the signature validation failed. This is because the JWT middleware was configured to handle tokens signed using RS256, but instead, a token was sent which was signed using HS256.

![Wrong Signature Configured](/media/articles/server-apis/webapi-owin/troubleshoot-wrong-signature-rs256.png)

The relevant error message to look for is the following:

```text
IDX10503: Signature validation failed. Keys tried: 'null'.
```

In the case where you configured your middleware for HS256, but you are sending an RS256 signed token the error message will be different:

![Wrong Signature Configured](/media/articles/server-apis/webapi-owin/troubleshoot-wrong-signature-hs256.png)

In this case, the relevant error message to look for is the following:

```text
IDX10500: Signature validation failed. Unable to resolve SecurityKeyIdentifier...
```

To resolve this issue, be sure that the signature algorithm with which the JWT was signed matches how your middleware is configured.

## 3. Has your token expired?

Each JSON Web Token is only valid until the time specified in the `exp` claim. If you and send a token which has expired, the token will be rejected:

![Token Expired](/media/articles/server-apis/webapi-owin/troubleshoot-token-expired.png)

The error message to look for is the following:

```text
Expired bearer token received
```

The resolve this issue, be sure to send a token which has not expired.

::: panel exp
  The value of the `exp` claim is a numeric value representing the number of seconds from 1970-01-01T00:00:00Z UTC until the specified UTC date/time. If you want to see the actual date/time for the value, you can visit <a href="http://www.epochconverter.com/">EpochConverter</a>.
:::

## 4. Did you configure the correct Issuer?

The Issuer specified in your token must match exactly with what is configured in your JWT middleware.

![Issuer Validation Failed](/media/articles/server-apis/webapi-owin/troubleshoot-issuer-validation-failed.png)

The error message to look for is the following:

```text
IDX10205: Issuer validation failed.
```

The resolve this issue, ensure that you specify the correct issuer for your JWT middleware. If your middleware is configured for RS256 signed tokens, this means ensuring that you have specified the correct value for the `TokenValidationParameters.ValidIssuer` property of the `JwtBearerAuthenticationOptions` parameter passed when calling `app.UseJwtBearerAuthentication(...)`.

If your middleware is configured for HS256 signed tokens, this means ensuring that you have passed the correct value to the constructor of `SymmetricKeyIssuerSecurityTokenProvider` which was configured as one of the `IssuerSecurityTokenProviders` for the `JwtBearerAuthenticationOptions`.

## 5. Does the audience match?

The audience specified in your token must match exactly with what is configured in your JWT middleware.

![Audience Validation Failed](/media/articles/server-apis/webapi-owin/troubleshoot-audience-validation-failed.png)

The error message to look for is the following:

```text
IDX10214: Audience validation failed.
```

The resolve this issue, ensure that you specify the correct audience for your JWT middleware.  If your middleware is configured for RS256 signed tokens, this means ensuring that you have specified the correct value for the `ValidAudience` property of the `TokenValidationParameters`

If your middleware is configured for HS256 signed tokens, this means ensuring that you have added the correct audience to the list of `AllowedAudiences` for the `JwtBearerAuthenticationOptions`.
