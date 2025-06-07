# DatoCMS SDKs Documentation

Welcome to the comprehensive documentation for all DatoCMS SDKs, refactored for clarity, discoverability, and optimal LLM-based retrieval.

## 🚀 Getting Started

To begin working with the DatoCMS SDKs, the first step is to set up authentication. This is a core concept that applies to all clients.

- **[Authentication Guide](./01-cma-client/00-core-concepts/authentication.md)** - Learn how to get API tokens and initialize the clients.

## 📚 Documentation Structure

This documentation is organized by client, reflecting the primary use cases for the SDKs.

### [01. Content Management API Client](./01-cma-client/)
Programmatically manage every aspect of your DatoCMS projects, from content and media to schemas and configuration. This is the most commonly used client.
- **[Core Concepts](./01-cma-client/00-core-concepts/)**: Foundational knowledge for using the client.
- **[Resource Guides](./01-cma-client/01-content-management/)**: In-depth documentation for every API resource.
- **[Advanced Topics](./01-cma-client/08-advanced-topics/)**: Deep dives into complex patterns and performance optimization.
- **[Reference](./01-cma-client/09-reference/)**: Quick look-up guides, matrices, and best practices.

### [02. Dashboard API Client](./02-dashboard-client/)
Manage account-level entities such as organizations, projects, users, and billing information.
- **[Account Management](./02-dashboard-client/account.md)**
- **[Organization Management](./02-dashboard-client/organization.md)**
- **[Site Management](./02-dashboard-client/site.md)**
- **[Billing & Subscriptions](./02-dashboard-client/billing-profile.md)**

### [03. Plugin SDK](./03-plugin-sdk/)
Extend the DatoCMS user interface by creating custom plugins, field editors, and sidebar widgets.
- **[Core Concepts](./03-plugin-sdk/01-core-concepts/)**: Plugin manifest and `connect` function.
- **[Hooks](./03-plugin-sdk/02-hooks/)**: Detailed documentation for all 44 available hooks.
- **[UI Components](./03-plugin-sdk/03-ui-components/)**: Reusable React components for building plugin interfaces.

## 📦 Package Reference

| Package | Description | Documentation |
|---------|-------------|---------------|
| `@datocms/cma-client` | Content Management API base client | [CMA Client](./01-cma-client/) |
| `@datocms/cma-client-node` | Node.js CMA client with file uploads | [CMA Client](./01-cma-client/02-media-management/upload.md) |
| `@datocms/cma-client-browser` | Browser CMA client | [CMA Client](./01-cma-client/02-media-management/upload.md) |
| `@datocms/dashboard-client` | Dashboard/Account API client | [Dashboard Client](./02-dashboard-client/) |
| `datocms-plugin-sdk` | Plugin development SDK | [Plugin SDK](./03-plugin-sdk/) |
| `datocms-react-ui` | React UI components for plugins | [UI Components](./03-plugin-sdk/03-ui-components/) |
| `@datocms/rest-api-events` | Real-time event subscriptions | [REST API Events](./01-cma-client/08-advanced-topics/rest-api-events.md) |
| `@datocms/rest-client-utils` | Shared utilities | [REST Client Utils](./01-cma-client/08-advanced-topics/rest-client-utils.md) |

## 🤝 Contributing

Found an issue or want to contribute?
- [GitHub Issues](https://github.com/datocms/js-datocms-client/issues)
- [Community Forum](https://community.datocms.com)