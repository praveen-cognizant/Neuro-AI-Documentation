# Airline Helpdesk Chatbot Documentation

## Overview

The Airline Helpdesk Chatbot is a multi-agent AI system built on the Neuro-SAN framework. It provides customers with comprehensive information about airline policies through a hierarchical structure of specialized AI agents, each with access to specific knowledge documents.

This repository contains detailed documentation about the chatbot's architecture, development process, implementation details, and extension guides.

## Documentation Structure

This repository contains the following documentation files:

1. **[Airline_Demo_Helpdesk_Documentation.md](1%20Airline_Demo_Helpdesk_Documentation.md)** - Overview of the chatbot system, including its development sequence, runtime execution flow, key components, and extension guidelines.

2. **[Agent_Execution_Flow.md](2%20Agent_Execution_Flow.md)** - Detailed diagrams and explanations of the customer interaction flow, agent processing sequence, and tools invocation processes.

3. **[Tools_Implementation.md](3%20Tools_Implementation.md)** - In-depth documentation on the `ExtractPdf` and `URLProvider` tools, their implementation details, and how they integrate with the agent system.

4. **[Development_Guide.md](4%20Development_Guide.md)** - Step-by-step guide for developing the chatbot, including project setup, dependencies installation, knowledge base organization, tool development, and testing strategies.

5. **[Dependencies_Guide.md](5%20Dependencies_Guide.md)** - Explanation of the dependencies required by the system, including details about the proprietary wheel files that form the backbone of the application.

6. **[Hocon_Configuration.md](6%20Hocon_Configuration.md)** - Guide to the HOCON configuration files used to define the agent hierarchy, behavior, and system settings.

7. **[Direct_Pdf_Processing.md](7%20Direct_Pdf_Processing.md)** - Comparison between the direct PDF processing approach used in this system and alternative vector database approaches, with explanations of the tradeoffs and benefits.

8. **[Agent_Framework_L1.rtf](8%20Agent_Framework_L1.rtf)** - Overview of the core architecture of the agent framework, including key components and message flow patterns.

9. **[Agent_Framework_L2.rtf](9%20Agent_Framework_L2.rtf)** - Comprehensive guide to building an agents framework from scratch, including detailed implementation examples.

## System Architecture

The Airline Helpdesk Chatbot uses a hierarchical multi-agent structure:

```
Airline 360 Assistant (Top-level coordinator)
├── Baggage_Handling
│   ├── Carry_On_Baggage
│   ├── Checked_Baggage
│   ├── Bag_Issues
│   ├── Special_Items
│   └── Bag_Fee_Calculator
├── Flights
│   ├── Military_Personnel
│   ├── Basic_Economy_Restrictions
│   └── Mileage_Plus
└── International_Travel
    ├── International_Checked_Baggage
    └── Embargoes
```

Each agent is specialized in a specific domain and has access to relevant policy documents.

## Key Features

- **Hierarchical Agent Structure**: Domain-specific agents collaborate to provide comprehensive answers
- **On-demand PDF Processing**: Direct extraction of content from policy documents at runtime
- **Dynamic URL Provision**: Relevant resource links included in responses
- **Configuration-driven Design**: System behavior defined through HOCON configuration files
- **Extensible Architecture**: Easily add new knowledge domains, agents, or tools

## Development and Deployment

To set up and run the system:

1. Install dependencies specified in `requirements.txt`
2. Organize policy documents in the `knowdocs` directory structure
3. Configure the agent hierarchy in HOCON files
4. Run the system using the provided runner script

For detailed development instructions, refer to the [Development_Guide.md](4%20Development_Guide.md) file.

## Tools and Technologies

The system is built using:
- Python as the core language
- PyPDF2 for PDF document extraction
- Neuro-SAN framework for agent orchestration
- HOCON for configuration management
- LLM integration (supports GPT-4o)

## Extending the System

To extend the chatbot with new capabilities:
1. Add new policy documents to the `knowdocs` directory
2. Update the `ExtractPdf` tool to include paths to new documents
3. Add new specialized agents in the HOCON configuration files
4. Connect new agents to the appropriate parent agents

For detailed extension guidelines, refer to the "Extending the System" section in [Airline_Demo_Helpdesk_Documentation.md](1%20Airline_Demo_Helpdesk_Documentation.md).Documentation related to inner working of the Neuro-AI framework with the airline helpdesk demo as an working example
