# 🔌 WebSocket Configuration Guide

This guide explains how to configure WebSocket routes for Django Supportal to enable real-time chat functionality.

## 📋 Prerequisites

Before configuring WebSockets, ensure you have:

- Django Channels installed: `pip install channels`
- Redis server running
- ASGI server (uvicorn, daphne, or hypercorn)

## 🚀 Quick Setup

### 1. Install Required Dependencies

```bash
pip install channels channels-redis uvicorn
```

### 2. Update Your Django Settings

Add the following to your `settings.py`:

```python
INSTALLED_APPS = [
    # ... your other apps
    'channels',
    'django_supportal',
]

# ASGI Application
ASGI_APPLICATION = "your_project.asgi.application"

# Channel Layers Configuration
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels.layers.InMemoryChannelLayer",  # For development
        # For production, use Redis:
        # "BACKEND": "channels_redis.core.RedisChannelLayer",
        # "CONFIG": {
        #     "hosts": [("127.0.0.1", 6379)],
        # },
    },
}
```

### 3. Create ASGI Configuration

Create or update your project's `asgi.py` file:

```python
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack

# Import Supportal WebSocket routes
from django_supportal.websocket.ws_routes import websocket_urlpatterns

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project.settings')

# Get the Django ASGI application
django_asgi_app = get_asgi_application()

application = ProtocolTypeRouter({
    "http": django_asgi_app,
    "websocket": AuthMiddlewareStack(
        URLRouter(
            websocket_urlpatterns
        )
    ),
})
```

### 4. Run with ASGI Server

```bash
# Using uvicorn (recommended)
python -m uvicorn your_project.asgi:application --reload

# Using daphne
daphne your_project.asgi:application

# Using hypercorn
hypercorn your_project.asgi:application --reload
```

## 🔧 Advanced Configuration

### Custom WebSocket Routes

If you have your own WebSocket consumers, combine them with Supportal routes:

```python
# asgi.py
from django.urls import re_path
from django_supportal.websocket.ws_routes import websocket_urlpatterns
from your_app.consumers import YourCustomConsumer

# Your custom WebSocket routes
custom_websocket_urlpatterns = [
    re_path(r"ws/custom/$", YourCustomConsumer.as_asgi()),
]

# Combine all routes
all_websocket_urlpatterns = websocket_urlpatterns + custom_websocket_urlpatterns

application = ProtocolTypeRouter({
    "http": django_asgi_app,
    "websocket": AuthMiddlewareStack(
        URLRouter(
            all_websocket_urlpatterns
        )
    ),
})
```

### Production Configuration

For production environments, use Redis as the channel layer:

```python
# settings.py
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [("127.0.0.1", 6379)],
            # Or use Redis URL:
            # "hosts": [os.environ.get('REDIS_URL', 'redis://localhost:6379')],
        },
    },
}
```

### Authentication Middleware

Add custom authentication middleware if needed:

```python
# asgi.py
from channels.auth import AuthMiddlewareStack
from channels.security.websocket import AllowedHostsOriginValidator

application = ProtocolTypeRouter({
    "http": django_asgi_app,
    "websocket": AllowedHostsOriginValidator(
        AuthMiddlewareStack(
            URLRouter(
                websocket_urlpatterns
            )
        )
    ),
})
```

## 🔌 WebSocket Endpoints

### Main Chat Endpoint

```
ws://your-domain/ws/chat/{business_id}/{session_id}/
```

**Parameters:**
- `business_id` (integer): The business identifier
- `session_id` (string): The chat session identifier

### Example Usage

#### JavaScript Client

```javascript
// Connect to WebSocket
const businessId = 1;
const sessionId = 'session-123';
const socket = new WebSocket(`ws://localhost:8000/ws/chat/${businessId}/${sessionId}/`);

// Connection event handlers
socket.onopen = function(event) {
    console.log('WebSocket connected');
};

socket.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('Message received:', data);
    
    // Handle different message types
    switch(data.sender) {
        case 'assistant':
            displayAssistantMessage(data.message);
            break;
        case 'user':
            displayUserMessage(data.message);
            break;
        case 'system':
            displaySystemMessage(data.message);
            break;
    }
};

socket.onclose = function(event) {
    console.log('WebSocket disconnected');
};

socket.onerror = function(error) {
    console.error('WebSocket error:', error);
};

// Send a message
function sendMessage(message) {
    socket.send(JSON.stringify({
        message: message,
        type: 'user'
    }));
}

// Send system message
function sendSystemMessage(message) {
    socket.send(JSON.stringify({
        message: message,
        type: 'system'
    }));
}
```

#### Python Client (using websockets)

```python
import asyncio
import websockets
import json

async def chat_client():
    uri = "ws://localhost:8000/ws/chat/1/session-123/"
    
    async with websockets.connect(uri) as websocket:
        # Send a message
        message = {
            "message": "Hello, I need help with my order",
            "type": "user"
        }
        await websocket.send(json.dumps(message))
        
        # Listen for responses
        async for response in websocket:
            data = json.loads(response)
            print(f"Received: {data['message']} from {data['sender']}")

# Run the client
asyncio.run(chat_client())
```

## 📨 Message Format

### Outgoing Messages (Client → Server)

```javascript
// User message
{
    "message": "Your question or statement",
    "type": "user"
}

// System message
{
    "message": "System notification or command",
    "type": "system"
}
```

### Incoming Messages (Server → Client)

```javascript
// Assistant response
{
    "message": "AI-generated response content",
    "sender": "assistant",
    "timestamp": "2024-01-01T12:00:00Z"
}

// User message echo
{
    "message": "Original user message",
    "sender": "user",
    "timestamp": "2024-01-01T12:00:00Z"
}

// System message
{
    "message": "System notification",
    "sender": "system",
    "timestamp": "2024-01-01T12:00:00Z"
}

// Error message
{
    "error": "Error description",
    "sender": "system"
}
```

## 🚨 Troubleshooting

### Common Issues

**1. WebSocket connection fails with 404**

- Check that your ASGI configuration is correct
- Verify the WebSocket URL pattern matches your route
- Ensure you're using an ASGI server (not the standard Django development server)

**2. Messages not being received**

- Check that Redis is running and accessible
- Verify the business_id and session_id are valid
- Check the message format is correct JSON

**3. Connection drops frequently**

- Check network stability
- Verify Redis connection
- Monitor server logs for errors

**4. CORS issues in development**

```python
# Add CORS middleware to your ASGI application
from corsheaders.middleware import CorsMiddleware

application = ProtocolTypeRouter({
    "http": CorsMiddleware(django_asgi_app),
    "websocket": AuthMiddlewareStack(
        URLRouter(
            websocket_urlpatterns
        )
    ),
})
```

### Debug Mode

Enable debug logging to troubleshoot issues:

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'channels': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
        'django_supportal': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },
}
```

## 🔒 Security Considerations

### Production Security

1. **Use HTTPS/WSS**: Always use secure WebSocket connections in production
2. **Authentication**: Implement proper authentication for WebSocket connections
3. **Rate Limiting**: Add rate limiting to prevent abuse
4. **Input Validation**: Validate all incoming messages

### Example Secure Configuration

```python
# settings.py
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [os.environ.get('REDIS_URL')],
            "capacity": 1500,  # Maximum number of messages in a channel
            "expiry": 10,      # Message expiry in seconds
        },
    },
}

# Security settings
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

## 📚 Additional Resources

- [Django Channels Documentation](https://channels.readthedocs.io/)
- [WebSocket API Reference](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [Redis Documentation](https://redis.io/documentation)
- [ASGI Specification](https://asgi.readthedocs.io/)

## 🤝 Support

If you encounter issues with WebSocket configuration:

1. Check the troubleshooting section above
2. Review the Django Channels documentation
3. Open an issue on the GitHub repository
4. Check the logs for detailed error messages
