# TaskTakeout

An open standard, OpenAPI-driven spec and toolkit that frees tasks from proprietary lock-ins.

## Overview

TaskTakeout defines a shared API for task data. Any provider or client can implement it, which gives users true data freedom. Users can switch apps, sync accounts, or export and import their data without pain. A common spec means clients do not need one-off adapters and providers can focus on features instead of bespoke integrations.

## The Problem

Task management tools today trap your data. Each app has its own format, its own API, and its own export quirks. Moving between tools means:
- Losing your task history and timestamps
- Breaking parent-child relationships
- Losing custom fields and metadata
- Writing one-off migration scripts
- Vendor lock-in

## The Solution

TaskTakeout provides a **standard API specification** that any task management provider can implement. When providers adopt TaskTakeout:
- Users can export complete data in a universal format
- Users can import that data into any other TaskTakeout-compliant provider
- Apps can sync across multiple providers
- Developers build once, work everywhere
- Competition focuses on features, not data lock-in

## Key Features

- **Complete data portability** - IDs, timestamps, relationships, and metadata preserved
- **RESTful API** with comprehensive filtering, search, and sorting
- **Hierarchical tasks** with parent-child relationships
- **Rich task model** - priorities, tags, due dates, descriptions, custom metadata
- **Atomic import/export** - transactional bulk operations
- **OpenAPI 3.1 specification** - machine-readable, tooling-friendly

## Available Versions

### [Version 1.0.0](./v1/) - Current
**Status:** Stable

The initial release of the TaskTakeout specification.

**Features:**
- Core CRUD operations for tasks
- Hierarchical task support (parent-child relationships)
- Filtering, searching, and sorting
- Priority levels (0-99)
- Tags for organization
- Custom metadata
- Bulk export/import with ID preservation
- Optimistic concurrency control

**Specification:** [`v1/openapi.yaml`](./v1/openapi.yaml)

**Documentation:** [`v1/README.md`](./v1/README.md)

## Quick Start

### For API Providers

Implement the TaskTakeout API in your task management service:

1. Review the specification: [`v1/openapi.yaml`](./v1/openapi.yaml)
2. Implement the core endpoints according to the spec
3. Support export/import for data portability
4. Test against the specification

### For API Clients

Integrate with TaskTakeout-compliant providers:

1. Review the API documentation: [`v1/README.md`](./v1/README.md)
2. Obtain an API token from your provider
3. Use the standard endpoints at `/task/v1/`
4. Leverage export/import for backups and migrations

### For Users

Look for task management apps that support TaskTakeout:
- Check if your current provider offers TaskTakeout export
- Choose new tools based on features, knowing your data isn't locked in
- Use export/import to switch tools or maintain backups

## Versioning Strategy

This repository uses directory-based versioning:
- Each major version lives in its own directory (`v1/`, `v2/`, etc.)
- Each version contains its own OpenAPI spec and documentation
- Versions are maintained independently
- Breaking changes result in a new version directory

## Implementation Status

TaskTakeout is an open specification. We're looking for:
- **API Providers** to implement the spec in their services
- **Client Developers** to build TaskTakeout-compatible apps
- **Contributors** to improve the specification

## Contributing

We welcome contributions to improve the TaskTakeout specification:

1. Fork the repository
2. Create a feature branch
3. Make your changes (update the spec or documentation)
4. Submit a pull request with a clear description

For breaking changes, propose a new version with rationale.

## License

[Add your license information here]

## Support

For questions, issues, or discussions about TaskTakeout:
- Open an issue in this repository
- Review existing issues and discussions
- Check the version-specific documentation

## Project Status

TaskTakeout is currently in **initial release**. We're seeking:
- Early adopters to implement the specification
- Feedback on the API design
- Real-world usage to validate the approach
- Community growth and adoption
