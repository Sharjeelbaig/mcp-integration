# MCP in Microsoft Foundry

## Foundry MCP cheat sheet

**MCP** lets a Foundry agent use a tool from another program.

### 1. Connect to Foundry

Create a credential and connect to the Foundry project:

```python
credential = DefaultAzureCredential()
project_client = AIProjectClient(
    endpoint=project_endpoint,
    credential=credential,
)
```

### 2. Initialize the MCP tool

Tell Foundry where the MCP server is:

```python
mcp_tool = MCPTool(
    server_label="api-specs",
    server_url="https://learn.microsoft.com/api/mcp",
    require_approval="always",
)
```

- `server_label` is the tool's name.
- `server_url` is the MCP server address.
- `require_approval` asks before the tool is used.

### 3. Create an agent with the MCP tool

Add the tool to the agent:

```python
agent = project_client.agents.create_version(
    agent_name="MyAgent",
    definition=PromptAgentDefinition(
        model=model_deployment,
        instructions="Use MCP tools when helpful.",
        tools=[mcp_tool],
    ),
)
```

### 4. Ask the agent a question

Create a conversation and send a request:

```python
conversation = openai_client.conversations.create()
response = openai_client.responses.create(
    conversation=conversation.id,
    input="Give me Azure CLI commands for a Container App.",
    extra_body={
        "agent_reference": {
            "name": agent.name,
            "type": "agent_reference",
        }
    },
)
```

### 5. Approve and read the answer

If Foundry asks to use the MCP tool, approve the request. Then read:

```python
response.output_text
```

### 6. Clean up

Delete the temporary agent version when finished:

```python
project_client.agents.delete_version(
    agent_name=agent.name,
    agent_version=agent.version,
)
```

**Flow:** Connect → Add MCP tool → Create agent → Ask → Approve → Read answer.

### Run it

1. Install the packages:

   ```bash
   pip install -r requirements.txt
   ```

2. Set `PROJECT_ENDPOINT` and `MODEL_DEPLOYMENT_NAME` in a `.env` file.
3. Sign in to Azure so `DefaultAzureCredential` can work.
4. Start the example:

   ```bash
   python client.py
   ```

5. Ask an inventory question. Type `quit` to stop.
