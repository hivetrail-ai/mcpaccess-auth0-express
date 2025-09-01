# mcpaccess-auth0-express
Secure your Model Context Protocol (MCP) server in minutes — Auth0-powered Express middleware with JWT validation and role-based access control for tools, resources, and prompts.

## Overview
mcpaccess-auth0-express provides a straightforward way to integrate Auth0 authentication into an Express-based MCP server.
It is designed for developers who need to protect MCP endpoints (tools, resources, and prompts) without adding unnecessary complexity.

This package handles:

- **JWT verification** using Auth0’s public keys.

- **Role-based access control (RBAC)** for fine-grained permissions.

- **Express middleware integration** with minimal configuration.

- **MCP-specific endpoint protection** for tools, resources, and prompts.

## When to Use
Use this package if:

You’re building an MCP server in Node.js with Express.

You want to authenticate and authorize users via Auth0.

You need to restrict access to specific MCP actions based on roles or claims.

You prefer a plug-and-play middleware over custom auth code.

## Features
Plug-in Express middleware for MCP endpoint protection.

Supports Auth0 RS256 token validation.

Role and scope checks for different MCP actions.

Works alongside other Express middleware.

Minimal boilerplate — focus on your MCP server logic, not on token handling.

## Install mcp-auth0-express

### Install the MCP Access package for auth0 and express

```bash
npm i @hivetrail/mcpaccess-auth0-express
```

## Configure mcp-auth0-express

mcp-auth0-express words as an express middleware that validates JWT tokens and applies access restrictions and filtering based on user permissions. 

To create the middleware, we use the `createMcpAccessMiddleware` function, passing an object with the required configuration. The object includes the following properties:

- **serverId**: (Required) A string that defines the current server id. The server id is used to identify the correct permissions in a multi-MCP server environment (Read more in the *Custom permission builder* section)
- **mcpPath**: (Required) A string representing the root path for Model Context Protocol requests, which is often “/mcp”. The MCP path is used to distinguish between MCP requests and other network requests.
- **jwtOptions**: (Required) An object containing the JWT validation configuration. It supports all properties from the [AuthOptions](https://auth0.github.io/node-oauth2-jwt-bearer/interfaces/AuthOptions.html) object type in [express-oauth2-jwt-bearer](https://github.com/auth0/node-oauth2-jwt-bearer/tree/main/packages/express-oauth2-jwt-bearer), and requires at least two properties:
    - **issuerBaseURL**
    - **audience**
- **useOutboundFilter**: (Optional) A boolean value for activating and disabling the outbound filtering mechanism where tools,resources and prompts are filtered out of list requests if the user does not have the appropriate permissions. The default is True.
- **loggerOptions**: (Optional) An object containing optional logging calls. See the *Logger object for logging* section for more information.
- **auth0ManagementOptions**: (Optional) An object containing optional Auth0 client configuration that will be used to validate the user, client, and permissions in real-time directly with the Auth0 API instead of relying on the JWT token for the user information. See the *Auth0 management object for client, user and permission verification* section for more information.
- **permissionSeparator**: (Optional) A custom separator string used to create the permission name string, the default is a colon “:”. See the *Custom permission builder* section for more information.
- **permissionBuilderFn**: (Optional) A custom function for creating permission names. The default function will use the following permission syntax: serverId:item1:item2… (for example: myMcpServer:tools:list or myMcpServer:resources:read:schemaType://res1). See the *Custom permission builder* section for more information.

An example of create and using the MCP Access Auth0 Express middleware using minimal configuration:

```jsx
import express from "express";
import {
  createMcpAccessMiddleware
} from "@hivetrail/mcpaccess-auth0-express";

//create the configuration object
const mcpAccessConfig = {
	serverId: "mcp",
  mcpPath: "/mcp",
  jwtOptions: {
	  issuerBaseURL: process.env.AUTH0_ISSUER_URL,
	  audience: process.env.AUTH0_AUDIENCE
  },
};

//create the Auth0 Express middleware
const mcpAuthMiddleware = createMcpAccessMiddleware(mcpAccessConfig);

const app = express();

app.use(express.json());

//set CORS settings as needed
app.use(
  cors({
    origin: "*", // Allow all origins - adjust as needed for production
    exposedHeaders: ["Mcp-Session-Id"],
  })
);

//activate the middleware
app.use(mcpAuthMiddleware);

//... the rest of the MCP server code

```

### JWT token configuration object for token verification

The JWT token configuration object contains the settings used to validate the JWT token sent to the MCP server. To validate the token we use [Auth0 express-oauth2-jwt-bearer](https://github.com/auth0/node-oauth2-jwt-bearer/tree/main/packages/express-oauth2-jwt-bearer) to create a middleware that inspects the token based on the configuration information.

The JWT token configuration object is based on the [***AuthOptions***](https://auth0.github.io/node-oauth2-jwt-bearer/interfaces/AuthOptions.html) type from [Auth0 express-oauth2-jwt-bearer](https://github.com/auth0/node-oauth2-jwt-bearer/tree/main/packages/express-oauth2-jwt-bearer), with two required properties:

- issuerBaseURL
- audience

These properties will be used for:

- Setup and activate the express-oauth2-jwt-bearer’s middleware to validate the JWT token.
- Respond to “/.well-known/oauth-protected-resource” requests with the auth0 issuer url ([RFC 9728](https://datatracker.ietf.org/doc/rfc9728/), [MCP server authorization](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization))
- Respond to “"/.well-known/oauth-authorization-server” requests with the auth0 issuer url ([RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414), [MCP server authorization](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization))

You can add additional optional properties to the JWT token configuration object as described in the [express-oauth2-jwt-bearer](https://github.com/auth0/node-oauth2-jwt-bearer/tree/main/packages/express-oauth2-jwt-bearer) documentation:

- [middleware](https://auth0.github.io/node-oauth2-jwt-bearer/functions/auth.html)
- [AuthOptions](https://auth0.github.io/node-oauth2-jwt-bearer/interfaces/AuthOptions.html)

**configuration example:**

setup an .env file with the audience and base issuer URL.

*.env file:*

```
AUTH0_AUDIENCE=***audience_url***
AUTH0_ISSUER_URL=https://***conn_id***.us.auth0.com/
```

*configuration in main code:*

```jsx
import type {JwtConfig} from '@hivetrail/mcpaccess-auth0-express';

const jwtTokenConfig: JwtConfig = {
  issuerBaseURL: process.env.AUTH0_ISSUER_URL,
  audience: process.env.AUTH0_AUDIENCE,
  tokenSigningAlg: "RS256",
};
```

### Auth0 management object for client, user and permission verification (optional)

In addition to validating the JWT token, we can also verify the information supplied in the JWT token in real-time directly:

- check that the user exists
- retrieve the user permissions
- check that the client exists

We use the [Auth0 client](https://github.com/auth0/node-auth0) to connect to the Auth0 services, query and validate the information from the client and JWT token, and specify which elements to verify by setting the appropriate flags.

The Auth0 management object consist of the following properties:

- **client** - The Auth0 management client settings used for connecting to the Auth0 services. The client config should contain the following properties:
    - **domain**
    - **clientId**
    - **clientSecret**
- **verifyUser** - a flag for specifying weather to query the user information
- **verifyClient** - a flag for specifying weather to query the client information
- **verifyPermissions** - a flag for specifying weather to query the user permission information (bypassed if verifyUser is true, as the permissions are already retrieved with the user information).

**configuration example:**

*.env file:*

```
AUTH0_CLIENT_ID=***client_id***
AUTH0_CLIENT_SECRET=***client_secret***
AUTH0_DOMAIN=***auth0_client_domain***
```

*configuration in main code:*

```jsx
import type {Auth0ManagementConfig} from '@hivetrail/mcpaccess-auth0-express';

const auth0ManagementConfig: Auth0ManagementConfig = {
  client: {
    domain: process.env.AUTH0_DOMAIN,
    clientId: process.env.AUTH0_CLIENT_ID,
    clientSecret: process.env.AUTH0_CLIENT_SECRET,
  },
  verifyUser: true,
  verifyClient: true,
  verifyPermissions: true,
};
```

### Logger object for logging (optional)

You can pass a logger function (winston, pino, etc’) to receive logging information. The logging configuration consists of the following properties:

- **logger** - the logger function that will be called on logging events
- **logPrefix** - an optional string prefix for log messages to help differentiate them from other server log messages
- **logConsoleFallback** - an optional flag that will fallback to using console warning and error in case the logger function fails or throws and error. The default is set to true.

configuration:

```jsx
const loggerConfig: loggerConfig = {
  logger: logger,
  logPrefix: "MyMCP: ",
  logConsoleFallback: false
};
```

### Custom permission builder (optional)

By default, the package expects permissions to follow a syntax starting with the MCP server Id followed by path items separated by a colon (:), or asterisk (*) for any value:

*server_id*:*pathItem[0]*:*pathItem[1]*

for example:

myMcpServer:tools:list

myMcpServer:resources:read:*

You can customize the structure of the permission string in two ways:

1. **Customize the permission separator:** override the default colon separator by setting the optional **“permissionSeparator”** property in the configuration options. For example, setting the permissionSeparator value to “\” will expect the following permission pattern:
myMcpServer\tools\list
myMcpServer\resources\read\*
- **Use a custom function to generate the permission string from path items:** provide a function that accepts an array of strings and returns the permission string to the optional **“permissionBuilderFn”** property in the configuration options. For example:

```jsx
//using the custom function bellow as the permissionBuilderFn:
function customPermissionBuilder(pathItems:string[]){
	return pathItems.join("-");
}

//will create the following permission strings:
// tools-list
// resources-read-*
```

### Creating the middleware

we’ll bring all the objects we’ve create above together to create our MCP auth middleware:

```jsx
import {
  createMcpAccessMiddleware,
  type McpAuthOptions,
  type Auth0ManagementConfig,
  type loggerConfig,
  type JwtConfig,
} from "@hivetrail/mcpaccess-auth0-express";

export const jwtAuthOptions: McpAuthOptions = {
  serverId: "mcp",
  mcpPath: "/mcp",
  jwtOptions: jwtTokenConfig,
  useOutboundFilter: true,
  loggerOptions: loggerConfig,
  auth0ManagementOptions: auth0ManagementConfig
};

export const mcpAuthMiddleware = createMcpAccessMiddleware(jwtAuthOptions);
```

Then we can use the middleware in our main express code:

```jsx
import express, { NextFunction, Request, Response } from "express";
import { mcpAuthMiddleware } from "@services/auth.service";

const app = express();

app.use(express.json());
app.use(
  cors({
    origin: "*",
    exposedHeaders: ["Mcp-Session-Id"],
  })
);
app.use(mcpAuthMiddleware);
```

### complete code example:

```jsx
import {
  createMcpAccessMiddleware,
  type McpAuthOptions,
  type Auth0ManagementConfig,
  type loggerConfig,
  type JwtConfig,
} from "@hivetrail/mcpaccess-auth0-express";

const jwtTokenConfig: JwtConfig = {
  issuerBaseURL: process.env.AUTH0_ISSUER_URL,
  audience: process.env.AUTH0_AUDIENCE,
  tokenSigningAlg: "RS256",
};

const loggerConfig: loggerConfig = {
  logger: logger,
};

const auth0ManagementConfig: Auth0ManagementConfig = {
  client: {
    domain: process.env.AUTH0_DOMAIN || "",
    clientId: process.env.AUTH0_CLIENT_ID || "",
    clientSecret: process.env.AUTH0_CLIENT_SECRET || "",
  },
  verifyUser: true,
  verifyClient: true,
  verifyPermissions: true,
};

export const jwtAuthOptions: McpAuthOptions = {
  serverId: "mcp",
  mcpPath: "/mcp",
  jwtOptions: jwtTokenConfig,
  useOutboundFilter: true,
  loggerOptions: loggerConfig,
  auth0ManagementOptions: auth0ManagementConfig,
};

export const mcpAuthMiddleware = createMcpAccessMiddleware(jwtAuthOptions);
```

Then use the middleware in the main express configuration:

```jsx
import express, { NextFunction, Request, Response } from "express";
import { mcpAuthMiddleware } from "@services/auth.service";

const app = express();
app.use(express.json());
app.use(
  cors({
    origin: "*", // Allow all origins - adjust as needed for production
    exposedHeaders: ["Mcp-Session-Id"],
  })
);
app.use(mcpAuthMiddleware);
```

## License
This package is **free to use for any purpose** (including commercial), but you **may not modify, redistribute, or create derivative works** without prior written permission.  
See the [LICENSE](LICENSE) file for full details.
