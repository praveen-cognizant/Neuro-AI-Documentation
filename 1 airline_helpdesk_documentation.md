# Airline Helpdesk Chatbot Documentation

## Overview

The Airline Helpdesk Chatbot is built on the Neuro-SAN multi-agent framework. It provides customers with information about airline policies by utilizing a hierarchical structure of specialized AI agents, each with access to specific knowledge documents.

This documentation covers:
1. Development Tasks Sequence
2. Runtime Execution Flow
3. Key Components
4. Extending the System

## 1. Development Tasks Sequence

Building this helpdesk chatbot involves the following sequence of tasks:

### Phase 1: Knowledge Base Preparation
1. **Gather policy documents**: Collect PDFs and text files containing airline policies
2. **Organize knowledge structure**: Arrange documents in appropriate folders by topic (baggage, flights, international)
3. **Prepare document extraction tools**: Create tools that can read and parse PDFs/text files

### Phase 2: Agent Architecture Design
1. **Define agent hierarchy**: Create a tree of specialized agents based on domains
2. **Write agent instructions**: Craft role-specific instructions for each agent
3. **Configure communication flow**: Set up the delegation and response compilation mechanisms

### Phase 3: Tool Development
1. **Create PDF extraction tool**: Develop the `ExtractPdf` class to retrieve document content
2. **Create URL provider tool**: Develop the `URLProvider` class to provide relevant web links
3. **Implement any additional tools**: Develop other required utility functions

### Phase 4: System Configuration
1. **Create HOCON configuration**: Define the complete agent hierarchy in HOCON format
2. **Configure environment variables**: Set up necessary paths and API keys
3. **Implement runner script**: Create the startup script to launch the system

### Phase 5: Testing and Refinement
1. **Test with sample queries**: Verify responses across different policy domains
2. **Refine agent instructions**: Improve agent behavior based on testing results
3. **Expand knowledge base**: Add missing information to fill knowledge gaps

## 2. Runtime Execution Flow

When a customer interacts with the helpdesk chatbot, the following sequence occurs:

### Startup Sequence
1. The `run.py` script initializes the environment variables and configuration
2. The Neuro-SAN server starts, loading the agent configuration from HOCON files
3. The web client launches, providing the user interface

### Query Processing Sequence
1. **User Input**: Customer submits a question through the web interface
2. **Top-Level Agent Processing**:
   - The "Airline 360 Assistant" receives the inquiry
   - It analyzes the question to determine which domain it belongs to
   - It routes the question to appropriate sub-agents (Baggage_Handling, Flights, or International_Travel)

3. **Sub-Agent Processing**:
   - The sub-agent evaluates if it can handle the inquiry
   - It determines which specialized agents should address the question
   - It identifies any additional information needed from the user
   - It delegates to more specialized agents (e.g., Checked_Baggage, Carry_On_Baggage)

4. **Information Retrieval**:
   - Leaf agents use the `ExtractPdf` tool to retrieve relevant policy information
   - They process the content to find answers to the specific question
   - They use the `URLProvider` tool to find relevant web links

5. **Response Compilation**:
   - Each specialized agent creates its response
   - Responses flow back up the agent hierarchy
   - Each level compiles and refines the responses from lower levels
   - The top-level agent creates the final comprehensive response

6. **User Response**: The final answer is presented to the customer through the web interface

## 3. Key Components

### Agent Hierarchy
- **Airline 360 Assistant**: Top-level coordinator
  - **Baggage_Handling**: Handles baggage-related inquiries
    - Carry_On_Baggage, Checked_Baggage, Bag_Issues, Special_Items, Bag_Fee_Calculator
  - **Flights**: Handles flight-related inquiries
    - Military_Personnel, Basic_Economy_Restrictions, Mileage_Plus
  - **International_Travel**: Handles international travel inquiries
    - International_Checked_Baggage, Embargoes

### Tools
- **ExtractPdf**: Extracts and processes text from PDF and text files
- **URLProvider**: Provides relevant URLs to policy pages

### Configuration
- **HOCON Files**: Define agent hierarchy, instructions, and tools
- **Environment Variables**: Configure system paths and settings

### Knowledge Base
- Organized collection of policy documents in PDF and text format
- Structured by topic for efficient retrieval

## 4. Extending the System

### Adding New Knowledge Domains
1. Add new documents to the `knowdocs` directory
2. Update the `ExtractPdf` class to include paths to new documents
3. Create new specialized agents in the HOCON configuration
4. Connect new agents to the appropriate parent agents

### Modifying Agent Behavior
1. Edit agent instructions in the HOCON configuration
2. Refine the communication flow between agents
3. Update tool implementations as needed

### Adding New Tools
1. Create new Python classes in the `coded_tools` directory
2. Implement the `CodedTool` interface
3. Register new tools in the HOCON configuration
4. Connect tools to appropriate agents

### Improving User Experience
1. Enhance the web interface
2. Add feedback mechanisms
3. Implement conversation history and context maintenance