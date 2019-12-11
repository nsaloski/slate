### Basic Authentication

As of Caymland 2.3.0, support for basic authentication can be enabled through Caymland's Configuration -> API Settings. As with OAuth2, it is only recommended to use basic authentication over HTTPS.
 
To authorize a request for basic authentication, set an `Authorization` header. 

1. Combine the username and password of a Caymland user with a colon `:`. For example, `user:password`.
2. Base64 encode the string from above. `dXNlcjpwYXNzd29yZA==`.
3. Add an Authorization header to each API request as `Authorization: Basic dXNlcjpwYXNzd29yZA==`
