# MSAL token refresh example with Nuxt

As described in the comment of acquireTokenSilent in MSAL's UserAgentApplication.ts,
MSAL users should call acquireTokenSilent before each API call.

This function checks if the token is exprired (or within the expiration offset) and
retrieves a new token if needed.

```
L697: Use this function to obtain a token before every call to the API / resource provider
``` 

https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core/src/UserAgentApplication.ts#L697

## MSAL

https://github.com/AzureAD/microsoft-authentication-library-for-js

## Prepare dotenv

CLIENT_ID and TENANT_ID of Azure AD are required in the .env file.

```
$ cat .env
CLIENT_ID=xxx
TENANT_ID=xxx
```

## Build Setup

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```

# License

MIT
