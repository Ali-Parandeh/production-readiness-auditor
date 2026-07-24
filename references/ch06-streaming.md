# Chapter 6 — Real-time streaming (SSE and WebSocket)

Grounding for the **streaming half of Area 3**. Read before grading streaming. Source: *Building Generative AI Services with FastAPI*, Ch 6. (Model loading and serving, Ch 3, is grounded separately when that reference exists.)

## Why stream

LLMs are autoregressive: they predict the next token from previous inputs, appending each output token and passing it through again until a stop token ends the loop. Rather than waiting for the loop to finish, forward tokens to the client as they are generated. Providers expose this with `stream=True`, returning a data generator instead of the final output, which you pass to FastAPI for streaming.

## Communication mechanisms

The chapter compares HTTP request-response, short/regular polling, long polling, SSE, and WebSocket for AI workflows. SSE is a one-way server-to-client stream over HTTP. WebSocket is full-duplex. Choose the mechanism by the interaction, not by habit.

## SSE endpoints

Set the provider's `stream=True`, take the returned token generator, and stream it through FastAPI. Suits one-way generation (server streams tokens to the client).

## WebSocket endpoints

FastAPI supports WebSocket via Starlette's WebSocket interface. Connections must be managed, so implement a **connection manager** that tracks active connections and their state in an `active_connections` list. The manager can also **broadcast** to all clients (group chat, collaborative editing, system alerts).

Endpoint pattern (for example `ws://host/generate/text/stream`):
- Accept and open the connection.
- While open, keep sending/receiving messages.
- On the first message, pass it as the prompt to the model API.
- Asynchronously iterate over the generated stream and send each chunk to the client.
- Wait a small amount between chunks to reduce race conditions and give the client time to process.
- Log important events and errors inside the controller to trace root causes.
- `WebSocketDisconnect` is raised when the client closes the connection.
- On a server-side error during an open connection, log it and identify the client.
- Break the loop and gracefully close when the stream finishes, on internal error, or on client close, then remove the connection from `active_connections`.

## WebSocket exception handling

Different from HTTP. After acceptance you maintain an open connection, so you no longer return responses with status codes or `HTTPException`. WebSocket does not support HTTP 4xx/5xx codes. On an exception, gracefully close the connection and/or send the client an error **message** in place of an HTTP response, notifying them before closing. Use WebSocket CLOSE-frame status codes to signal the closure reason and drive custom closure behaviour on server or client.

## Designing streaming APIs (key architectural opinion)

Pitfall: exposing too many streaming endpoints, for example one preconfigured endpoint per message type in a conversation. This forces the client to switch between endpoints and manage conversation state on both sides, adding backend and frontend complexity and inviting race-condition and networking issues.

Preferred: a **single entry point** to initiate a stream, using headers, request body, or query parameters to trigger the relevant backend logic. The backend owns routing and business logic, manages state, and switches prompts or models, with access to databases and services for CRUD. This simplifies frontend state management. One endpoint acts as the single entry point for switching logic, managing state, and generating custom responses.

## Applying this when grading (Area 3, streaming)

- Are tokens streamed as generated (`stream=True` to a generator, through FastAPI), or buffered until the full completion is ready? Buffered output for an interactive UX is a fail.
- Is the mechanism (SSE vs WebSocket) chosen with a rationale fit to the workflow? One-way generation suits SSE; interactive/bidirectional suits WebSocket.
- WebSocket: is there a connection manager? Is graceful close and `WebSocketDisconnect` handled? Are errors sent as WebSocket messages and CLOSE codes rather than HTTP exceptions? Is there inter-chunk pacing?
- API design: a single streaming entry point, or a proliferation of streaming endpoints for one conversation (flag the latter)?
