# Airline Helpdesk Development Guide

This guide outlines the development process and structure of the Airline Helpdesk Chatbot system.

## Development Sequence

### 1. Project Setup

```
# Create project structure
mkdir -p neuro-san-airline
cd neuro-san-airline
mkdir -p coded_tools/airline_policy
mkdir -p knowdocs/{baggage,flight,international}
mkdir -p registries
```

### 2. Dependencies Installation

```python
# Create requirements.txt
wheels_private/leaf_common-1.2.18-py3-none-any.whl
wheels_private/leaf_server_common-0.1.15-py3-none-any.whl
wheels_private/neuro_san-0.2.7-py3-none-any.whl
wheels_private/neuro_san_web_client-0.1.3-py3-none-any.whl
pypdf2>=3.0.1
```

### 3. Knowledge Base Organization

Structure your documents according to domains:

```
knowdocs/
├── baggage/
│   ├── bag-issues/
│   │   ├── damaged_Issues with your checked bags.pdf
│   │   ├── delayed_Issues with your checked bags.pdf
│   │   ├── lost_Issues with your checked bags.pdf
│   ├── carryon/
│   │   ├── Carry-on Bags.pdf
│   ├── checked/
│   │   ├── Checked bags.pdf
│   ├── special-items/
│   │   ├── Dangerous Items.pdf
│   │   ├── Electronic Device.pdf
│   │   ...
├── flight/
│   ├── basic-econ/
│   ├── mileage-plus/
│   ├── military-personnel/
├── international/
    ├── International Travel Requirements.pdf
    ├── International checked bag limits.pdf
```

### 4. Tool Development

Create the necessary tools for document extraction and URL provision:

#### PDF Extraction Tool

```python
# coded_tools/airline_policy/extract_pdf.py
from typing import Any, Dict, Union
import os
from PyPDF2 import PdfReader
from neuro_san.interfaces.coded_tool import CodedTool

class ExtractPdf(CodedTool):
    def __init__(self):
        self.default_path = ["knowdocs/Help Center _ United Airlines.pdf"]
        self.docs_path = {
            "Bag Issues": "knowdocs/baggage/bag-issues",
            "Carry On Baggage": "knowdocs/baggage/carryon",
            # Add paths for all supported apps
        }
    
    def invoke(self, args: Dict[str, Any], sly_data: Dict[str, Any]) -> Union[Dict[str, Any], str]:
        app_name: str = args.get("app_name", None)
        # Process and extract text from PDFs
        # ...
```

#### URL Provider Tool

```python
# coded_tools/airline_policy/url_provider.py
from typing import Any, Dict, Union
from neuro_san.interfaces.coded_tool import CodedTool

class URLProvider(CodedTool):
    def __init__(self):
        self.airline_policy_urls = {
            "Baggage Tracking": "https://www.united.com/en/us/bagdelivery/start",
            "Damaged Bags Claim": "https://rynnsluggage.com/",
            # Add URLs for all supported apps
        }
    
    def invoke(self, args: Dict[str, Any], sly_data: Dict[str, Any]) -> Union[Dict[str, Any], str]:
        app_name: str = args.get("app_name", None)
        # Return appropriate URL
        # ...
```

### 5. Agent Configuration (HOCON)

Create the configuration file defining agent hierarchy:

```hocon
# registries/airline_policy.hocon
{
    "llm_config": {
        "model_name": "gpt-4o",
        "verbose": true
    },
    "commondefs": {
        # Define shared instructions and functions
    },
    "tools": [
        {
            "name": "Airline 360 Assistant",
            "function": {
                "description": "Your name is Airline 360 Assistant..."
            },
            "instructions": "...",
            "tools": ["Baggage_Handling", "Flights", "International_Travel"]
        },
        {
            "name": "Baggage_Handling",
            "function": "aaosa_call",
            "instructions": "...",
            "command": "aaosa_command",
            "tools": ["Carry_On_Baggage", "Checked_Baggage", "Bag_Issues", "Special_Items", "Bag_Fee_Calculator"]
        },
        # Define all remaining agents
        # ...
    ]
}
```

### 6. System Runner Development

Create a runner script that launches all components:

```python
# run.py
import os
import subprocess
import signal
import argparse

class NeuroSanRunner:
    def __init__(self):
        # Initialize configuration
        # ...
    
    def set_environment_variables(self):
        # Set environment variables
        # ...
    
    def run(self):
        # Start server and client
        # ...

if __name__ == "__main__":
    runner = NeuroSanRunner()
    runner.run()
```

## Development Best Practices

### 1. Agent Design Principles

- **Single Responsibility**: Each agent should focus on a specific domain
- **Clear Instructions**: Write precise instructions that define agent behavior
- **Proper Delegation**: Ensure agents know when to delegate to specialized sub-agents
- **Comprehensive Coverage**: Ensure all policy domains are covered by specialized agents

### 2. Knowledge Management

- **Document Organization**: Group documents logically by topic
- **Content Coverage**: Ensure all common customer questions are addressed in documents
- **Regular Updates**: Keep policy documents up to date as airline policies change

### 3. Tool Development

- **Error Handling**: Implement robust error handling in all tools
- **Performance Optimization**: Optimize PDF extraction for large documents
- **Modularity**: Keep tools focused on single responsibilities

### 4. Testing Strategy

1. **Unit Testing**: Test individual tools and their functions
2. **Integration Testing**: Test agent interactions and communication
3. **End-to-End Testing**: Test complete customer interaction flows
4. **Domain Coverage Testing**: Verify all policy domains can be answered correctly

### 5. Extending the System

When adding new capabilities:

1. **Add Knowledge**: Place new documents in appropriate folders
2. **Update Tools**: Modify tools to access new knowledge sources
3. **Configure Agents**: Add new specialized agents or modify existing ones
4. **Update Registration**: Ensure new agents are properly registered in HOCON files
5. **Test New Capabilities**: Verify new functionality works as expected

## Implementation Notes

### Environment Variables

```bash
# Required environment variables
export AGENT_MANIFEST_FILE="./registries/manifest.hocon"
export AGENT_TOOL_PATH="./coded_tools"
export OPENAI_API_KEY="your-api-key"
```

### Starting Components Manually

```bash
# Start server
python -m neuro_san.service.agent_main_loop --port 30011

# Start web client
python -m neuro_san_web_client.app

# Start CLI client (alternative)
python -m neuro_san.client.agent_cli --connection service --agent airline_policy
```