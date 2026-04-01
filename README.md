# Moonshot.Extensions.AI

Provides an implementation of `IChatClient` and `IEmbeddingGenerator` for Moonshot (Kimi) API and OpenAI-compatible endpoints, based on `Microsoft.Extensions.AI`.

## Install the package

From the command-line:

```console
dotnet add package Moonshot.Extensions.AI-Community
```

Or directly in the C# project file:

```xml
<ItemGroup>
  <PackageReference Include="Moonshot.Extensions.AI-Community" Version="10.4.1" />
</ItemGroup>
```

---

# Usage Examples

## Chat

```csharp
using Microsoft.Extensions.AI;

IChatClient client =
    new Moonshot.Chat.ChatClient("kimi-k2.5", Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIChatClient();

Console.WriteLine(await client.GetResponseAsync("What is AI?"));
```

---

## Chat + Conversation History

```csharp
using Microsoft.Extensions.AI;

IChatClient client =
    new Moonshot.Chat.ChatClient("kimi-k2.5", Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIChatClient();

Console.WriteLine(await client.GetResponseAsync(
[
    new ChatMessage(ChatRole.System, "You are a helpful AI assistant"),
    new ChatMessage(ChatRole.User, "What is AI?")
]));
```

---

## Chat Streaming

```csharp
using Microsoft.Extensions.AI;

IChatClient client =
    new Moonshot.Chat.ChatClient("kimi-k2.5", Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIChatClient();

await foreach (var update in client.GetStreamingResponseAsync("What is AI?"))
{
    Console.Write(update);
}
```

---

## Tool Calling

```csharp
using System.ComponentModel;
using Microsoft.Extensions.AI;

IChatClient moonshotClient =
    new Moonshot.Chat.ChatClient("kimi-k2.5", Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIChatClient();

IChatClient client = new ChatClientBuilder(moonshotClient)
    .UseFunctionInvocation()
    .Build();

ChatOptions chatOptions = new()
{
    Tools = [AIFunctionFactory.Create(GetWeather)]
};

await foreach (var message in client.GetStreamingResponseAsync("Do I need an umbrella?", chatOptions))
{
    Console.Write(message);
}

[Description("Gets the weather")]
static string GetWeather() =>
    Random.Shared.NextDouble() > 0.5 ? "It's sunny" : "It's raining";
```

---

## Caching

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Caching.Distributed;
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Extensions.Options;

IDistributedCache cache =
    new MemoryDistributedCache(Options.Create(new MemoryDistributedCacheOptions()));

IChatClient moonshotClient =
    new Moonshot.Chat.ChatClient("kimi-k2.5", Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIChatClient();

IChatClient client = new ChatClientBuilder(moonshotClient)
    .UseDistributedCache(cache)
    .Build();

Console.WriteLine(await client.GetResponseAsync("What is AI?"));
```

---

## Text Embedding Generation

```csharp
using Microsoft.Extensions.AI;

IEmbeddingGenerator<string, Embedding<float>> generator =
    new Moonshot.Embeddings.EmbeddingClient(
        "text-embedding-v1",
        Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIEmbeddingGenerator();

var embeddings = await generator.GenerateAsync("What is AI?");

Console.WriteLine(string.Join(", ", embeddings[0].Vector.ToArray()));
```

---

## Dependency Injection

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var builder = Host.CreateApplicationBuilder();

builder.Services.AddChatClient(services =>
    new Moonshot.Chat.ChatClient(
        "kimi-k2.5",
        Environment.GetEnvironmentVariable("MOONSHOT_API_KEY"))
    .AsIChatClient());

var app = builder.Build();

var chatClient = app.Services.GetRequiredService<IChatClient>();
Console.WriteLine(await chatClient.GetResponseAsync("What is AI?"));
```

---

## Minimal Web API

```csharp
using Microsoft.Extensions.AI;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddChatClient(services =>
    new Moonshot.Chat.ChatClient(
        "kimi-k2.5",
        builder.Configuration["MOONSHOT_API_KEY"])
    .AsIChatClient());

var app = builder.Build();

app.MapPost("/chat", async (IChatClient client, string message) =>
{
    var response = await client.GetResponseAsync(message);
    return response.Message;
});

app.Run();
```

---

## Notes

* Compatible with Moonshot (Kimi) API
* Supports OpenAI-compatible endpoints
* Fully integrates with `Microsoft.Extensions.AI`
* Supports streaming, tool-calling, caching, telemetry, and DI

---

## Environment Variables

```bash
MOONSHOT_API_KEY=your_api_key
```

Optional custom endpoint:

```bash
MOONSHOT_BASE_URL=https://api.moonshot.cn/v1
```

---

## Relationship to Microsoft.Extensions.AI.OpenAI

This package is adapted from `Microsoft.Extensions.AI.OpenAI` and modified to support Moonshot (Kimi) API while preserving the same programming model.

---

## License

Same as upstream project.
