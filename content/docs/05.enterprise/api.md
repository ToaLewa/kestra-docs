---
title: Kestra EE API
icon: /docs/icons/admin.svg
---

This page describes how you can interact with Kestra Enterprise Edition using the API.

## Authentication

To authenticate with the Kestra API, you need to first create an [API token](./api-tokens.md). You can create it directly from the Kestra UI.

Once you have your API token, you can use it to authenticate with the API. You can use the `Authorization` header with the `Bearer` token to authenticate with the API.

```bash
curl -X POST http://localhost:8080/api/v1/executions/dev/hello-world \
-H "Authorization: Bearer YOUR_API_TOKEN"
```

## Browse the API Reference

For a full list of available API endpoints, check the [Enterprise Edition API Reference](https://kestra.io/docs/api-reference/enterprise).