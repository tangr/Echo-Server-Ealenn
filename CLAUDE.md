# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Echo Server is a Kubernetes testing utility that echoes HTTP requests back to clients. It's designed for testing pods, services, and Kubernetes infrastructure. The server can be customized to return specific HTTP codes, bodies, headers, and response times through query parameters or request headers.

## Architecture

### Core Structure
- **Entry Point**: `src/webserver.js` - Minimal server startup using `src/app.js`
- **Application Core**: `src/app.js` - Express.js setup with middleware chain and catch-all route
- **Configuration**: `src/nconf.js` + `src/global.json` - Hierarchical config (CLI args > ENV vars > JSON defaults)
- **Middlewares**: `src/middlewares/` - Chain of custom middlewares that process echo commands
- **Response Modules**: `src/response/` - Generate different parts of the JSON response

### Request Processing Flow
1. **Custom Response Time**: Add artificial delays via `echo_time` parameter
2. **HTTP Code Control**: Set custom status codes via `echo_code` parameter  
3. **Header Modification**: Add custom headers via `echo_header` parameter
4. **Logging**: Log all requests (can be disabled for ping endpoints)
5. **File Explorer**: Serve directory/file contents via `echo_file` parameter
6. **Environment Body**: Return environment variable values via `echo_env_body` parameter
7. **Custom Body**: Override response body via `echo_body` parameter
8. **Final Response**: JSON containing host, http, request, and environment data

### Key Configuration
Configuration follows hierarchy: CLI args > Environment variables > `src/global.json`

Environment variables use `__` separator (e.g., `LOGS__LEVEL`, `ENABLE__HOST`).

## Development Commands

### Testing
```bash
# Install dependencies
npm ci

# Run tests without coverage  
npm run test

# Run tests with coverage
npm run test-with-coverage
```

### Building
```bash
# Build production bundle with webpack
npm run build
```

### Running
```bash
# Development
PORT=8080 npm run start
# or
node ./src/webserver --port 8080

# Docker
docker run -p 8080:80 ealen/echo-server
```

## Key Features

### Echo Commands
All commands work via query parameters OR request headers:
- `echo_code` / `X-ECHO-CODE`: HTTP status codes (200-599, supports ranges like "200-404")
- `echo_body` / `X-ECHO-BODY`: Custom response body
- `echo_env_body` / `X-ECHO-ENV-BODY`: Return environment variable value
- `echo_header` / `X-ECHO-HEADER`: Add response headers ("Key:Value, Key2:Value2")
- `echo_time` / `X-ECHO-TIME`: Response delay in milliseconds (0-60000)
- `echo_file` / `X-ECHO-FILE`: File/directory browser

### Feature Toggles
Can disable response sections via environment variables:
- `ENABLE__HOST`, `ENABLE__HTTP`, `ENABLE__REQUEST`, `ENABLE__ENVIRONMENT`, etc.

### Logging
- Supports multiple formats: default, line, object (JSON)
- Integration with Seq and ELK stack
- Can ignore ping requests: `LOGS__IGNORE__PING`

## Testing Notes

The application has comprehensive Mocha tests in the `test/` directory covering all middleware functionality. Tests use supertest for HTTP request testing.

## Deployment

Designed primarily for Kubernetes/Docker deployment with multi-architecture support. Includes Helm charts and example Kubernetes manifests in `docs/examples/`.