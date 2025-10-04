# Server-Sent Events (SSE) with .NET 10 - Educational POC

This project demonstrates how to implement **Server-Sent Events (SSE)** using the new ## 📁 Project Structure

```
server-sent-events/
├── API/
│   ├── API.csproj              # .NET 10 project file
│   ├── Program.cs              # Main application with SSE endpoint
│   ├── API.http               # HTTP test requests
│   └── Properties/
│       └── launchSettings.json # Development settings
├── global.json                # .NET SDK version pinning
├── server-sent-events.sln     # Solution file
└── README.md                  # This file
```s introduced in **.NET 10**. It's a simple educational proof-of-concept that shows real-time heart rate monitoring using minimal APIs.

## 🎯 What is Server-Sent Events (SSE)?

Server-Sent Events is a web standard that allows a server to push data to a web page in real-time. Unlike WebSockets, SSE is unidirectional (server-to-client only) and simpler to implement for scenarios where you only need to send data from server to client.

> **Note:** This project uses "server-sent-events" (with hyphens) as the standard naming convention for SSE, which is the correct terminology according to the W3C specification.

## 🆕 What's New in .NET 10

.NET 10 introduces native support for Server-Sent Events with:

- **`SseItem<T>`** - Strongly-typed SSE item representation
- **`TypedResults.ServerSentEvents()`** - Built-in minimal API support
- **`System.Net.ServerSentEvents`** namespace - Complete SSE framework

## 🚀 Features

- ✅ **Pure API** - No UI, just the SSE endpoint
- ✅ Real-time heart rate simulation (60-100 BPM)
- ✅ Native .NET 10 SSE implementation
- ✅ Minimal API with OpenAPI documentation
- ✅ Proper cancellation token handling
- ✅ Automatic reconnection support
- ✅ Type-safe event streaming
- ✅ Ready for integration with any frontend

## 📋 Prerequisites

- **.NET 10 SDK** (RC or later) - [Download here](https://dotnet.microsoft.com/download/dotnet/10.0)
- **Visual Studio 2025** or **VS Code** (optional)

## 🛠️ Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd server-sent-events
```

### 2. Verify .NET 10 Installation

```bash
dotnet --version
# Should show: 10.0.100-rc.1.25451.107 or later
```

### 3. Restore Dependencies

```bash
dotnet restore
```

### 4. Build the Project

```bash
dotnet build
```

### 5. Run the Application

```bash
cd API
dotnet run
```

The application will start at `http://localhost:5039`

## 🔌 API Endpoints

| Endpoint | Method | Description | Response Type |
|----------|--------|-------------|---------------|
| `/sse-item` | GET | Server-Sent Events stream of heart rate data | `text/event-stream` |
| `/openapi/v1.json` | GET | OpenAPI specification | `application/json` |

## 🧪 Testing the SSE Endpoint

### Option 1: Using VS Code REST Client

Open `API/API.http` and click "Send Request" on the SSE endpoint:

```http
GET http://localhost:5039/sse-item
Accept: text/event-stream
Cache-Control: no-cache
```

### Option 2: Using Browser

Navigate to: `http://localhost:5039/sse-item`

You'll see a continuous stream of heart rate data:

```
event: heartRate
data: 75

event: heartRate
data: 82

event: heartRate
data: 68
```

### Option 3: Using curl

```bash
curl -N -H "Accept: text/event-stream" http://localhost:5039/sse-item
```

### Option 4: Using JavaScript Client

Create your own HTML file to test the SSE endpoint:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Heart Rate Monitor</title>
</head>
<body>
    <h1>Real-time Heart Rate</h1>
    <div id="heartRate">Connecting...</div>

    <script>
        const eventSource = new EventSource('http://localhost:5039/sse-item');
        
        eventSource.addEventListener('heartRate', function(event) {
            document.getElementById('heartRate').innerHTML = 
                `❤️ Heart Rate: ${event.data} BPM`;
        });

        eventSource.onerror = function(event) {
            console.error('SSE error:', event);
        };
    </script>
</body>
</html>
```

### Option 4: Using curl

```bash
curl -N -H "Accept: text/event-stream" http://localhost:5039/sse-item
```

## 📁 Project Structure

```
server-side-events/
├── API/
│   ├── API.csproj              # .NET 10 project file
│   ├── Program.cs              # Main application with SSE endpoint
│   ├── API.http               # HTTP test requests
│   └── Properties/
│       └── launchSettings.json # Development settings
├── global.json                # .NET SDK version pinning
├── server-side-events.sln     # Solution file
└── README.md                  # This file
```

## 🔍 Code Explanation

### Core SSE Implementation

```csharp
app.MapGet("sse-item", (CancellationToken cancellationToken) =>
{
    async IAsyncEnumerable<SseItem<int>> GetHeartRate(
        [EnumeratorCancellation] CancellationToken cancellationToken)
    {
        while (!cancellationToken.IsCancellationRequested)
        {
            var heartRate = Random.Shared.Next(60, 100);
            yield return new SseItem<int>(heartRate, eventType: "heartRate")
            {
                ReconnectionInterval = TimeSpan.FromMinutes(1)
            };
            await Task.Delay(2000, cancellationToken);
        }
    }

    return TypedResults.ServerSentEvents(GetHeartRate(cancellationToken));
});
```

### Key Components

1. **`IAsyncEnumerable<SseItem<int>>`** - Returns a stream of SSE items
2. **`SseItem<int>`** - Strongly-typed SSE item with data and metadata
3. **`TypedResults.ServerSentEvents()`** - Converts the stream to SSE response
4. **`[EnumeratorCancellation]`** - Proper cancellation token propagation
5. **`eventType: "heartRate"`** - Custom event type for client filtering

## ⚙️ Configuration

### global.json

Ensures the project uses .NET 10 SDK:

```json
{
  "sdk": {
    "version": "10.0.100-rc.1.25451.107",
    "rollForward": "latestPatch"
  }
}
```

### Project File (API.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="10.0.0-rc.1.25451.107" />
  </ItemGroup>
</Project>
```

## 🌟 SSE vs WebSockets

| Feature | Server-Sent Events | WebSockets |
|---------|-------------------|------------|
| **Direction** | Server → Client only | Bidirectional |
| **Protocol** | HTTP | WebSocket Protocol |
| **Complexity** | Simple | More complex |
| **Auto-reconnect** | Built-in | Manual implementation |
| **Firewall-friendly** | Yes (HTTP) | Sometimes blocked |
| **Use cases** | Live feeds, notifications | Chat, gaming, collaboration |

## 🎓 Educational Objectives

This POC demonstrates:

1. **Modern .NET 10 SSE APIs** - Latest framework features
2. **Minimal APIs** - Simplified web API development
3. **Async Streaming** - `IAsyncEnumerable` patterns
4. **Real-time Communication** - Server-to-client data push
5. **Cancellation Patterns** - Proper resource cleanup
6. **Type Safety** - Strongly-typed event streaming

## 🚀 Next Steps & Extensions

### Beginner Extensions
- Add different event types (temperature, pressure, etc.)
- Implement event filtering on the client side
- Add basic error handling and retry logic

### Intermediate Extensions
- Add authentication and authorization
- Implement multiple SSE endpoints
- Add persistent data storage (Entity Framework)
- Create a web dashboard with charts

### Advanced Extensions
- Add horizontal scaling with Redis
- Implement SSE with SignalR fallback
- Add monitoring and logging
- Performance testing and optimization

## 📚 Additional Resources

- [Microsoft Docs - SSE in Minimal APIs](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-10.0#server-sent-events-sse)
- [MDN - Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [.NET 10 Preview Features](https://devblogs.microsoft.com/dotnet/)
- [Minimal APIs Documentation](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)

## 🤝 Contributing

This is an educational project. Feel free to:

1. Fork the repository
2. Create feature branches
3. Add more SSE examples
4. Improve documentation
5. Submit pull requests

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## ⚠️ Note

This project uses **.NET 10 RC** which is a preview version. For production applications, wait for the stable release or use .NET 8/9 with manual SSE implementation.

---

**Happy Learning! 🎉**

If you find this helpful, please ⭐ star the repository and share it with others learning about Server-Sent Events and .NET 10!