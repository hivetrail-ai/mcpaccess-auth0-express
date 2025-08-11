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

## License
This package is **free to use for any purpose** (including commercial), but you **may not modify, redistribute, or create derivative works** without prior written permission.  
See the [LICENSE](LICENSE) file for full details.
