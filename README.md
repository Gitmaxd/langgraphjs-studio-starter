# LangGraph Studio TypeScript Starter (Beta)

This is a sample project to help you get started with developing [LangGraph.js](https://github.com/langchain-ai/langgraphjs) projects in [LangGraph Studio](https://github.com/langchain-ai/langgraph-studio).

![](/static/studio.png)

It contains a simple example graph exported from `src/agent.ts` that implements a basic ReAct pattern where the model can use tools for more information before responding to a user query, as well as the required `langgraph.json` for opening the graph in LangGraph Studio.

## Getting Started

This demo requires an [OpenAI API key](https://openai.com/) and a [Tavily API key](https://tavily.com/) for search results.

1. Clone this repository. (git clone https://github.com/langchain-ai/langgraphjs-studio-starter.git)
2. Rename the existing `.env.example` file `.env` and fill in your `OPENAI_API_KEY` and `TAVILY_API_KEY`.
3. Download the latest release of LangGraph Studio [from here](https://github.com/langchain-ai/langgraph-studio/releases).
4. Log in/sign up for [LangSmith](https://smith.langchain.com/) if you haven't already.
5. Open the enclosing folder in LangGraph Studio.
6. Start testing your app!

The graph has access to a web search tool powered by Tavily - you can try asking it about current events like `"What is the current conservation status of the Great Barrier Reef?"` and see it use the tool.

Note that the `Deploy` button is currently not supported, but will be soon!

You will also need the latest versions of `@langchain/langgraph` and `@langchain/core`. See these instructions for help upgrading an [existing project](https://langchain-ai.github.io/langgraphjs/how-tos/manage-ecosystem-dependencies/).

You can also [click here](https://www.loom.com/share/81cafa32d57f4933bd5d9b08c70f460c?sid=4ebcb366-f27a-4c49-854d-169106b4f6fe) to see a (rough) video tour of Studio.

## Development

You must export your graph, or a function that returns a created graph, from a specified file. See [this page for more information](https://langchain-ai.github.io/langgraph/cloud/reference/cli/#configuration-file).

While iterating on your graph, you can edit past state and rerun your app from past states to debug specific nodes. Local changes will be automatically applied via hot reload. Try adding an interrupt before the agent calls tools, updating the default system message in `src/utils/state.ts` to take on a persona, or adding additional nodes and edges!

Follow up requests will be appended to the same thread. You can create an entirely new thread, clearing previous history, using the `+` button in the top right.

You can find the latest (under construction) docs on [LangGraph.js](https://langchain-ai.github.io/langgraphjs/) here, including examples and other references.

### Defining state

The sample graph's state uses a prebuilt annotation called `MessagesAnnotation` to declare its state define how it handles return values from nodes. This annotation defines a state that is an object with a single key called `messages`. When a node in your graph returns messages, these returned messages are accumulated under the `messages` key in the state.

A sample pattern might look like this:

1. HumanMessage - initial user input
2. AIMessage with .tool_calls - agent picking tool(s) to use to collect information
3. ToolMessage(s) - the responses (or errors) from the executed tools
    (... repeat steps 2 and 3 as needed ...)
4. AIMessage without .tool_calls - agent responding in unstructured format to the user.
5. HumanMessage - user responds with the next conversational turn.
    (... repeat steps 2-5 as needed ... )

The graph's state will merge lists of messages or returned single messages, updating existing messages by ID.

By default, this ensures the state is "append-only", unless the new message has the same ID as an existing message.

For further reading, see [this page](https://langchain-ai.github.io/langgraphjs/how-tos/define-state/#getting-started).

## Notes

Currently in order for Studio to draw conditional edges properly, you will need to add a third parameter that manually lists the possible nodes the edge can route between. Here's an example:

```ts
.addConditionalEdges(
  // First, we define the edges' source node. We use `callModel`.
  // This means these are the edges taken after the `callModel` node is called.
  "callModel",
  // Next, we pass in the function that will determine the sink node(s), which
  // will be called after the source node is called.
  routeModelOutput,
  // List of the possible destinations the conditional edge can route to.
  // Required for conditional edges to properly render the graph in Studio
  [
    "tools",
    "__end__"
  ],
)
```

We are working to lift this requirement in the future.

LangGraph Studio also integrates with [LangSmith](https://smith.langchain.com/) for more in-depth tracing and collaboration with teammates.

You can swap in other models if you'd like by using [the appropriate LangChain.js integration package](https://js.langchain.com/v0.2/docs/integrations/chat/) or the appropriate SDK directly.
