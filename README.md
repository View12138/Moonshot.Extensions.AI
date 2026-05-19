# Community.Extensions.AI.CompatibleAI

<div align="center">
<img src="https://github.com/View12138/Community.Extensions.AI.CompatibleAI/blob/main/eng/CompatibleAI-Logo.png?raw=true" height=256px/>
</div>

[![NuGet stable version](https://img.shields.io/nuget/v/Community.Extensions.AI.CompatibleAI.svg)](https://www.nuget.org/packages/Community.Extensions.AI.CompatibleAI)


Provides an implementation of `IChatClient` and `IEmbeddingGenerator` for Kimi/DeepSeek API and OpenAI-compatible endpoints, based on `Microsoft.Extensions.AI`.

## Install the package

From the command-line:

```console
dotnet add package Community.Extensions.AI.CompatibleAI
```

---

# Usage Examples

## Chat

```csharp
using OpenAI;
using Microsoft.Extensions.AI;

IChatClient client = new OpenAIClient("kimi-k2.5", Environment.GetEnvironmentVariable("API_KEY"))
    .AsIChatClient();

Console.WriteLine(await client.GetResponseAsync("What is AI?"));
```

---

## Chat + Conversation History

```csharp
using OpenAI;
using Microsoft.Extensions.AI;

IChatClient client = new OpenAIClient("kimi-k2.5", Environment.GetEnvironmentVariable("API_KEY"))
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
using OpenAI;
using Microsoft.Extensions.AI;

IChatClient client = new OpenAIClient("kimi-k2.5", Environment.GetEnvironmentVariable("API_KEY"))
    .AsIChatClient();

await foreach (var update in client.GetStreamingResponseAsync("What is AI?"))
{
    Console.Write(update);
}
```

---

## Tool Calling

```csharp
using OpenAI;
using System.ComponentModel;
using Microsoft.Extensions.AI;

IChatClient client = new OpenAIClient("deepseek-v4-flash", Environment.GetEnvironmentVariable("API_KEY"))
    .AsIChatClient();

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

## Notes

* Compatible with Kimi/DeepSeek API
* Supports OpenAI-compatible endpoints
* Fully integrates with `Microsoft.Extensions.AI`
* Supports streaming, tool-calling, caching, telemetry, and DI

---

## Environment Variables

```bash
API_KEY=your_api_key
```

Optional custom endpoint:

```bash
BASE_URL=https://api.moonshot.cn/v1
# BASE_URL=https://api.deepseek.com
```

---

## Relationship to Microsoft.Extensions.AI.OpenAI

This package is adapted from `Microsoft.Extensions.AI.OpenAI` and modified to support Kimi/DeepSeek API while preserving the same programming model.

---

## License

Same as upstream project.
