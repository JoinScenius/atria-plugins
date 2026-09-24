# Security

## Reporting a vulnerability

Email [engineer-support@tryatria.com](mailto:engineer-support@tryatria.com) with the details and a
way to reproduce. Please do not open a public issue for a security report. We will acknowledge and
keep you updated while we work on a fix.

## Scope

This repository holds the Atria plugin, its skills and its documentation. The MCP server itself is a
hosted service at `https://api.tryatria.com/mcp` and its source is not in this repository; report
issues with the service to the same address.

## How the plugin authenticates

The plugin declares one remote MCP server and no credentials. Sign-in is OAuth 2.1 with automatic
discovery, performed by your MCP client; tokens are held by the client, never by this repository.
The plugin ships no hooks and runs no code on your machine.
