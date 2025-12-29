# AI Coding Agent Instructions

## Project Overview
Educational proof-of-concept demonstrating .NET 10's native Server-Sent Events (SSE) implementation using minimal APIs. Single endpoint streams simulated heart rate data (60-100 BPM) every 2 seconds.

## Key Architecture
- **Single minimal API** in [Program.cs](../API/Program.cs) - entire application logic in ~35 lines
- **No UI layer** - pure API for SSE streaming, designed for frontend integration
- Uses .NET 10's native `System.Net.ServerSentEvents` namespace with `SseItem<T>` and `TypedResults.ServerSentEvents()`
- Single endpoint pattern: `/sse-item` returns `text/event-stream` with `heartRate` events

## Critical .NET 10 Patterns

### SSE Implementation Pattern
```csharp
// Always use IAsyncEnumerable<SseItem<T>> with [EnumeratorCancellation]
async IAsyncEnumerable<SseItem<int>> GetHeartRate(
    [EnumeratorCancellation] CancellationToken cancellationToken)
{
    yield return new SseItem<int>(data, eventType: "heartRate")
    {
        ReconnectionInterval = TimeSpan.FromMinutes(1)
    };
}
```

**Why**: `[EnumeratorCancellation]` properly propagates cancellation tokens through async enumerables. The `eventType` parameter enables client-side event filtering via `addEventListener('heartRate', handler)`.

## Development Workflows

### Build & Run
```bash
dotnet restore              # Restore .NET 10 packages
dotnet build                # Compile (outputs to bin/Debug/net10.0/)
cd API && dotnet run        # Launch on https://localhost:7144
```

### Testing SSE Endpoint
- **VS Code**: Use [API.http](../API/API.http) with REST Client extension
- **Browser**: Navigate to `https://localhost:7144/sse-item` (streams continuously)
- **curl**: `curl -N -H "Accept: text/event-stream" https://localhost:7144/sse-item`

**Note**: The `-N` flag in curl disables buffering, critical for real-time SSE streaming.

## Project Conventions

### SDK Version Pinning
[global.json](../global.json) locks to .NET 10 SDK with `rollForward: latestPatch` - ensures consistent builds across environments. Always update both `global.json` and package references together when upgrading.

### Port Configuration
Fixed ports in [launchSettings.json](../API/Properties/launchSettings.json):
- HTTPS: `7144`
- HTTP: `5039`

Referenced in [API.http](../API/API.http) variable: `@API_HostAddress = https://localhost:7144`

### Naming Convention
Use "server-sent-events" (hyphens) per W3C specification, not "server sent events" or "ServerSentEvents".

## When Adding Features

### New SSE Endpoints
Follow the same pattern in Program.cs:
1. Create `IAsyncEnumerable<SseItem<T>>` method with `[EnumeratorCancellation]`
2. Use `yield return` for streaming
3. Set meaningful `eventType` for client filtering
4. Return via `TypedResults.ServerSentEvents()`
5. Add test case to [API.http](../API/API.http)

### Multiple Event Types
Return different `eventType` values in the same stream. Clients filter with `eventSource.addEventListener('eventName', handler)`.

### Error Handling in SSE
Don't throw exceptions in the enumerator - they close the stream. Instead, yield error events:
```csharp
yield return new SseItem<string>("Error occurred", eventType: "error");
```

## External Dependencies
- `Microsoft.AspNetCore.OpenApi` (version matches .NET 10 SDK) - provides `/openapi/v1.json` endpoint
- No database, external APIs, or authentication - intentionally kept minimal for educational purposes

## What This Project Is NOT
- Not production-ready (preview SDK, no auth, no persistence)
- Not a full-stack application (API only, no UI)
- Not demonstrating SSE with scaling (single instance, no Redis/SignalR)

Focus on understanding .NET 10's SSE APIs - see README's "Next Steps & Extensions" for production considerations.
