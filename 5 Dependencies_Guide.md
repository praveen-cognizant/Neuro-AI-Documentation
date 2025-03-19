# Dependencies and Wheel Files Documentation

This document explains the dependencies required by the Airline Helpdesk Chatbot system and details what's contained in the wheel (.whl) files.

## Dependencies Overview

The Airline Helpdesk Chatbot relies on several key dependencies:

1. **Neuro-SAN Framework**: The core multi-agent framework implemented in the private wheel files
2. **PyPDF2**: For PDF document extraction and processing
3. **Python Standard Library**: For file operations, process management, etc.

## Wheel Files Explained

The `wheels_private` directory contains proprietary wheel files that form the backbone of the application:

### 1. leaf_common-1.2.18-py3-none-any.whl

**Purpose**: Provides fundamental utilities and common code for the Leaf AI platform.

**Key Components**:
- Base utility classes for data handling
- Configuration management tools
- Common exception handling
- JSON/HOCON parsing utilities
- Logging infrastructure

### 2. leaf_server_common-0.1.15-py3-none-any.whl

**Purpose**: Server-side components for Leaf AI systems.

**Key Components**:
- Server configuration management
- API endpoint definitions
- Request/response handling
- Authentication mechanisms
- Connection pooling
- Communication protocols

### 3. neuro_san-0.2.7-py3-none-any.whl

**Purpose**: Core multi-agent orchestration framework.

**Key Components**:
- Agent definition system
- Agent communication protocols
- Tool interface definitions
- Agent lifecycle management
- Task delegation mechanisms
- HOCON configuration parsers
- LLM integration components
- Agent state management

**Key Classes**:
- `CodedTool` interface: Base class for all tools like ExtractPdf
- `agent_main_loop`: Server implementation for agent execution
- `agent_cli`: Command-line interface to interact with agents

### 4. neuro_san_web_client-0.1.3-py3-none-any.whl

**Purpose**: Web-based client interface for the Neuro-SAN framework.

**Key Components**:
- Web server implementation
- User interface components
- WebSocket communication
- Client-side state management
- Agent visualization tools (`agents_diagram_builder`)
- User session handling
- Chat history management

## Installation Process

The dependencies are installed through pip using the requirements.txt file:

```bash
pip install -r requirements.txt
```

The requirements.txt file lists both the private wheel files (installed from local paths) and public dependencies like PyPDF2.

## Dependency Flow

```
┌─────────────────────┐
│                     │
│ neuro_san_web_client│◄───────────┐
│                     │            │
└─────────┬───────────┘            │
          │                        │
          │ depends on             │
          ▼                        │
┌─────────────────────┐            │
│                     │            │
│     neuro_san       │────────────┘
│                     │              
└─────────┬───────────┘            
          │                        
          │ depends on             
          ▼                        
┌─────────────────────┐            
│                     │            
│ leaf_server_common  │            
│                     │            
└─────────┬───────────┘            
          │                        
          │ depends on             
          ▼                        
┌─────────────────────┐
│                     │
│    leaf_common      │
│                     │
└─────────────────────┘
```

## Integration with Custom Code

The Neuro-SAN framework provides extension points through:

1. **CodedTool Interface**: Custom tools like ExtractPdf and URLProvider implement this interface
2. **HOCON Configuration**: Agent definitions in HOCON format that reference custom tools
3. **Environment Variables**: Configuration of paths and settings

## Using the Dependencies

### Setting Up the Environment

```python
# Import the coded tool interface
from neuro_san.interfaces.coded_tool import CodedTool

# Create a custom tool
class MyCustomTool(CodedTool):
    def invoke(self, args, sly_data):
        # Custom implementation
        return {"result": "processed data"}
```

### Working with HOCON Files

```python
# The Neuro-SAN framework loads HOCON configuration
# registries/airline_policy.hocon defines agents that use:
{
    "name": "MyAgent",
    "function": "aaosa_call",
    "tools": ["MyCustomTool"]
}

# The class MyCustomTool is loaded by the framework through:
{
    "name": "MyCustomTool",
    "class": "my_module.MyCustomTool",
    "command": "Tool execution command"
}
```

## Advanced Usage

### LLM Configuration

The system uses OpenAI's models by default:

```hocon
"llm_config": {
    "model_name": "gpt-4o",
    "verbose": true
}
```

### Agent Communication

Agents communicate through a standardized protocol defined in the Neuro-SAN framework:

```hocon
"aaosa_call": {
    "description": "Depending on the mode, returns a natural language string in response.",
    "parameters": {
        "type": "object",
        "properties": {
            "inquiry": {
                "type": "string",
                "description": "The inquiry"
            },
            "mode": {
                "type": "string",
                "description": "indicates whether the agent is being asked to determine if the inquiry belongs to it..."
            }
        }
    }
}
```

## Security Considerations

1. **API Keys**: Required for LLM services (OpenAI)
2. **Private Wheels**: Contain proprietary code and should not be distributed
3. **Knowledge Management**: PDF documents may contain sensitive information