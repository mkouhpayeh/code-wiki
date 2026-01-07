
# AI Foundation

---

## Simple Chat
``` cs
using Azure;
using Azure.AI.OpenAI;
using OpenAI;
using OpenAI.Chat;

var endpoint = new Uri("https://xxxx.azure.com/");
var deploymentName = "gpt-35-turbo";
var apiKey = "xxx";

OpenAIClient client = new OpenAIClient(apiKey); // OpenAI


AzureOpenAIClient azureClient = new(
    endpoint,
    new AzureKeyCredential(apiKey)); // Azure OpenAI

ChatClient chatClient = azureClient.GetChatClient(deploymentName);

var requestOptions = new ChatCompletionOptions()
{
    MaxOutputTokenCount = 4096,
    Temperature = 1.0f,
    TopP = 1.0f,
};

List<ChatMessage> messages = new List<ChatMessage>()
{
    new SystemChatMessage("You are a helpful assistant. Be brief!"),
    new UserChatMessage("I am going to Paris, what should I see?"),
};

var response = chatClient.CompleteChat(messages, requestOptions);

switch(response.Value.FinishReason)
{
    case ChatFinishReason.Stop: // Response is ready
        Console.WriteLine(response.Value.Content[0].Text);
        break;

    case ChatFinishReason.Length: // hit max_tokens or context window
        Console.WriteLine("[truncated] " + response.Value.Content[0].Text);
        // ➜ Mitigate: raise MaxOutputTokenCount, shorten history, summarize older turns.
        break;

    case ChatFinishReason.ContentFilter:
        Console.WriteLine("[blocked] Output filtered by Azure content filters.");
        break;

    default:
        Console.WriteLine("[unknown finish] " + response.Value.Content[0].Text);
        break;
}

Console.ReadKey();
```

---

## Streaming
When we compare our chat application with the official ChatGPT interface, there is a big difference. Even though the generation speed was pretty much the same, our app still felt a lot slower. Why? </br>
Because our app waited until the entire poem was ready before showing it. And ChatGPT showed the text as it was being written word by word. This is called **streaming**. ChatGPT displays the response in chunks, allowing us to start reading the answer much earlier. </br>
The current generation speed of GPT-4 is slightly faster than what a human can read, but the GPT-3.5 is faster than GPT-4. This means that if we stream the response, the user can essentially follow along and doesn't perceive the AI as slow. </br></br>

As cool as streaming is, it has its disadvantages:

- It's more complex to implement. Instead of simply receiving and displaying a complete response

- We have to handle partial responses, keep track of the chunks, handle intermediate errors, and so on

- Requires more bandwidth as every chunk comes with quite a lot of overhead and only one or two useful words worth of content. 
    
Should we use streaming in your project? The answer, as it's pretty much always is, it depends. 

- The model

- The load on the server

- he amount of text you need to generate

- User's expectations or mindset.

``` cs title="Program.cs"
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;
using Azure;
using Azure.AI.OpenAI;

class Program
{
    static async Task Main()
    {
        var endpoint = new Uri("https://xxxx.azure.com/");
        var apiKey = "AZURE_API_KEY";
        var deploymentName = "gpt-35-turbo";

        var azureClient = new AzureOpenAIClient(endpoint, new AzureKeyCredential(apiKey));
        ChatClient chatClient = azureClient.GetChatClient(deploymentName);

        var messages = new List<ChatMessage>
        {
            new SystemChatMessage("You are a helpful assistant. Be brief!"),
            new UserChatMessage("I am going to Paris, what should I see?")
        };

        Console.OutputEncoding = Encoding.UTF8;
        var sb = new StringBuilder();

        Console.Write("Assistant: ");

        // ✅ Stream results asynchronously
        await foreach (StreamingChatCompletionUpdate update in chatClient.CompleteChatStreamingAsync(messages))
        {
            foreach (var part in update.ContentUpdate)
            {
                Console.Write(part.Text);
                sb.Append(part.Text);
            }
        }

        Console.WriteLine("\n\nFull message received:");
        Console.WriteLine(sb.ToString());
    }
}
```

``` cs title="Full Chat Loop"
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;
using Azure;
using Azure.AI.OpenAI;

class Program
{
    static async Task Main()
    {
        var endpoint = new Uri("https://mkou-mhde9425-eastus2.cognitiveservices.azure.com/");
        var apiKey = "AZURE_API_KEY";
        var deploymentName = "gpt-35-turbo";

        var azureClient = new AzureOpenAIClient(endpoint, new AzureKeyCredential(apiKey));
        var chatClient = azureClient.GetChatClient(deploymentName);

        // 🧠 Keeps the whole conversation (system + user + assistant)
        var messages = new List<ChatMessage>
        {
            new SystemChatMessage("You are a helpful and concise assistant.")
        };

        Console.OutputEncoding = Encoding.UTF8;
        Console.WriteLine("=== Chat started (type 'exit' to quit) ===");

        while (true)
        {
            Console.Write("\nYou: ");
            string? userInput = Console.ReadLine();

            if (string.IsNullOrWhiteSpace(userInput))
                continue;
            if (userInput.Equals("exit", StringComparison.OrdinalIgnoreCase))
                break;

            // Add user message to conversation history
            messages.Add(new UserChatMessage(userInput));

            Console.ForegroundColor = ConsoleColor.Yellow;

            Console.Write("Assistant: ");
            var sb = new StringBuilder();

            // 🔁 Stream assistant reply
            await foreach (StreamingChatCompletionUpdate update in chatClient.CompleteChatStreamingAsync(messages))
            {
                foreach (var part in update.ContentUpdate)
                {
                    Console.Write(part.Text);
                    sb.Append(part.Text);
                }
            }

            Console.ForegroundColor = ConsoleColor.White;

            Console.WriteLine();

            // Add assistant message to conversation history
            messages.Add(new AssistantChatMessage(sb.ToString()));
        }

        Console.WriteLine("\nChat ended.");
    }
}

```
