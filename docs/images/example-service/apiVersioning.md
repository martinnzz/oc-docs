### API Versioning best practices

URL-based versioning for recommended for all API endpoints. All API routes are prefixed with a version identifier (e.g., `/v1/health`, `/v1/data/:name`).
This approach allows the introduction of new features or breaking changes in future versions (such as `/v2/`) without affecting existing clients that rely on the current API structure.

Backwards Compatibility:
Older API versions will be maintained for a reasonable period to ensure existing integrations continue to function. When a new version is released,
clients can migrate at their own pace.

Example Endpoints:

- `/v1/health`
- `/v1/data`
- `/v1/cpu`

Always specify the desired API version in your requests to ensure consistent behavior.

### API Versioning Example

To support multiple API versions, the following example demonstrates both a non-breaking change (adding a new version
with extra fields) and a breaking change (changing the response type) using the welcome endpoint:

```typescript
// v1 endpoint
app.get('/v1/welcome', (req, res) => {
    res.type('text/plain').send('Hello user');
});

// v2 endpoint (Non-breaking change: updated response with JSON and extra info)
app.get('/v2/welcome', (req, res) => {
    res.json({ message: 'Hello user', info: 'Welcome to API v2!' });
});

// Breaking change: now returns JSON instead of plain text
app.get('/v1/welcome', (req, res) => {
    res.json({ message: 'Hello user' });
});
```

### API Data Structures vs Internal Data Structures

To maintain backwards compatibility and support API evolution, data structures used for API responses are separated from
internal data models.

- API Data Structs: Each API version defines its own data struct for responses (e.g., `UserV1`, `UserV2`).
  These structs remain stable for each version, ensuring clients receive consistent data even as the API evolves.
- Internal Data Structs: The internal data model (e.g., `User`) can change as needed to support new features or fields.

Example:

- `/v1/user` returns a `UserV1` struct.
- `/v2/user` returns a `UserV2` struct with additional fields.
- The internal user model (`User`) may include all fields required by any version.

This approach allows you to add new fields or change internal representations without breaking existing API clients.
Each API version only exposes the fields defined in its corresponding data struct.
