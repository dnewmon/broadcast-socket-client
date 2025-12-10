# broadcast-socket-client

A React hook for channel-based WebSocket communication using Socket.IO. This library provides a simple and intuitive way to connect to WebSocket servers, join specific channels, and exchange real-time messages.

## Features

- Simple React hook interface
- Channel-based message broadcasting
- Automatic connection management
- TypeScript support
- Real-time message synchronization
- Connection status tracking

## Installation

```bash
npm install broadcast-socket-client
```

## Requirements

- React 19.1.0 or higher
- Socket.IO server with channel support

## Usage

### Basic Example

```tsx
import { useBroadcastSocket } from 'broadcast-socket-client';

function ChatApp() {
  const { messages, isConnected, sendMessage, connect, disconnect } = useBroadcastSocket();

  // Connect to server and join a channel
  const handleConnect = () => {
    connect('http://localhost:12000', 'my-channel');
  };

  // Send a message
  const handleSend = () => {
    sendMessage({ text: 'Hello, World!' });
  };

  return (
    <div>
      <button onClick={handleConnect}>
        {isConnected ? 'Disconnect' : 'Connect'}
      </button>

      {isConnected && (
        <button onClick={handleSend}>Send Message</button>
      )}

      <div>
        {messages.map((msg, idx) => (
          <div key={idx}>
            <span>{new Date(msg.timestamp).toLocaleTimeString()}</span>
            <pre>{JSON.stringify(msg.data, null, 2)}</pre>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Advanced Example with Channel Switching

```tsx
import { useBroadcastSocket } from 'broadcast-socket-client';
import { useState } from 'react';

function MultiChannelApp() {
  const [serverUrl] = useState('http://localhost:12000');
  const [channel, setChannel] = useState('home');
  const { messages, isConnected, sendMessage, connect, disconnect } = useBroadcastSocket();

  const handleChannelChange = (newChannel: string) => {
    setChannel(newChannel);
    if (isConnected) {
      // Disconnect and reconnect to new channel
      disconnect();
      connect(serverUrl, newChannel);
    }
  };

  const handleSubmit = (text: string) => {
    if (isConnected && text.trim()) {
      sendMessage({
        text,
        userId: 'user-123',
        timestamp: Date.now()
      });
    }
  };

  return (
    <div>
      <select
        value={channel}
        onChange={(e) => handleChannelChange(e.target.value)}
        disabled={isConnected}
      >
        <option value="home">Home</option>
        <option value="sports">Sports</option>
        <option value="news">News</option>
      </select>

      <button onClick={() => isConnected ? disconnect() : connect(serverUrl, channel)}>
        {isConnected ? 'Disconnect' : 'Connect'}
      </button>

      <div>
        <h3>Channel: {channel}</h3>
        <p>Status: {isConnected ? 'Connected' : 'Disconnected'}</p>

        {messages.map((msg, idx) => (
          <div key={idx}>
            <small>{msg.sender || 'Unknown'}</small>
            <p>{msg.data.text}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Real-time Notifications Example

```tsx
import { useBroadcastSocket } from 'broadcast-socket-client';
import { useEffect } from 'react';

function NotificationSystem() {
  const { messages, isConnected, connect } = useBroadcastSocket();

  useEffect(() => {
    // Auto-connect to notifications channel on mount
    connect('http://localhost:12000', 'notifications');
  }, []);

  // Show toast notification for new messages
  useEffect(() => {
    if (messages.length > 0) {
      const latestMessage = messages[messages.length - 1];
      showToast(latestMessage.data);
    }
  }, [messages]);

  const showToast = (data: any) => {
    // Your toast notification logic here
    console.log('New notification:', data);
  };

  return (
    <div>
      <div>Connection: {isConnected ? '🟢 Online' : '🔴 Offline'}</div>
      <div>
        <h3>Recent Notifications ({messages.length})</h3>
        {messages.slice(-5).map((msg, idx) => (
          <div key={idx}>
            <span>{new Date(msg.timestamp).toLocaleString()}</span>
            <p>{msg.data.message}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## API Reference

### `useBroadcastSocket()`

Returns an object with the following properties and methods:

#### Properties

- **`messages`**: `BroadcastMessage[]` - Array of received messages
- **`isConnected`**: `boolean` - Current connection status

#### Methods

- **`connect(serverUrl: string, channel: string): void`** - Connect to a WebSocket server and join a channel
- **`disconnect(): void`** - Disconnect from the current connection
- **`sendMessage(data: any): void`** - Send a message to the current channel

### Types

```typescript
interface BroadcastMessage {
  data: any;
  timestamp: number;
  sender?: string;
}
```

## Server Requirements

Your Socket.IO server should support the following:

1. Accept a `channel` query parameter during connection
2. Emit `message` events to clients
3. Listen for `message` events from clients

Example server setup:

```javascript
io.on('connection', (socket) => {
  const channel = socket.handshake.query.channel;
  socket.join(channel);

  socket.on('message', (data) => {
    io.to(channel).emit('message', {
      data,
      timestamp: Date.now(),
      sender: socket.id
    });
  });
});
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
