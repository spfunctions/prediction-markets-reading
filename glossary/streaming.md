# Streaming Responses

**Streaming responses deliver data incrementally as it becomes available, rather than waiting for the complete response. SimpleFunctions uses streaming for real-time thesis evaluations and agent conversations.**


## Explanation

## Streaming in SimpleFunctions

Some operations take time — a thesis evaluation might need to process multiple signals, query market data, and run LLM inference. Streaming lets you see results as they're generated rather than waiting for the complete response.

### Where Streaming is Used

1. **Thesis evaluation**: As the agent processes each signal and updates each node, you see results incrementally
2. **Chat mode**: When conversing with the agent (`sf chat`), responses stream token-by-token
3. **Market scans**: Results appear as each market is processed, rather than all at once
4. **What-if analysis**: Cascade effects are shown as they're computed

### Technical Implementation

SimpleFunctions uses Server-Sent Events (SSE) for streaming:
- The client opens a persistent HTTP connection
- The server sends events as they become available
- Each event contains a chunk of the response
- The connection closes when the response is complete

### Benefits of Streaming

- **Faster perceived latency**: You see the first results in <1 second even if the full response takes 10 seconds
- **Interruptible**: You can cancel mid-stream if you see what you need
- **Progress visibility**: Know that the system is working, not hung

### CLI Streaming

The CLI streams by default for interactive commands. You can disable it for scripting:

```bash
sf thesis evaluate --no-stream  # Waits for complete response
sf thesis evaluate              # Streams results as they arrive
```


## Example

sf thesis evaluate recession-2026

Streaming output:
[0.2s] Processing 3 new signals...
[0.5s] Signal 1/3: BLS unemployment data → node n1.1
[0.8s]   n1.1 updated: 35% → 42%
[1.2s] Signal 2/3: Consumer confidence report → node n2.2
[1.5s]   n2.2 updated: 48% → 52%
[1.8s] Signal 3/3: Market price change (no node mapping)
[2.0s]   Filtered as noise
[2.5s] Recomputing root probability...
[2.8s]   Root: 38% → 42% (+4pt)
[3.0s] Checking strategies...
[3.2s]   S-001: Entry condition MET ✓
[3.5s] Evaluation complete.
       Confidence: 72% (+4pt)
       Summary: Labor market weakening accelerates thesis.


## CLI

```bash
sf chat
```


## Related

[cli](cli.md), [evaluation-cycle](evaluation-cycle.md), [mcp-server](mcp-server.md), [delta-api](delta-api.md)
