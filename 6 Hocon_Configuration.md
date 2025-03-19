# HOCON Configuration Files Documentation

This document explains the HOCON configuration files in the Airline Helpdesk Chatbot system - where they are located, how they're structured, and how they're used by the system.

## What is HOCON?

HOCON (Human-Optimized Config Object Notation) is a configuration format that's a superset of JSON. It's designed to be more human-friendly with features like:
- Comments
- Omitting quotes on strings in certain contexts
- Multi-line strings
- Including other files
- Substitutions (variable references)

In this system, HOCON is used to define the agent hierarchy and behavior.

## Location of HOCON Files

The HOCON configuration files are located in the `registries` directory at the project root:

```
/neuro-san-airline-main/registries/
├── airline_policy.hocon   # Main agent configuration file
└── manifest.hocon         # System manifest that references other HOCON files
```

## HOCON File Structure

### 1. manifest.hocon

This is the entry point for the Neuro-SAN framework. It lists all the HOCON configuration files that should be loaded:

```hocon
{
    # List each hocon file to serve as a key with a boolean value
    "airline_policy.hocon": true,
}
```

The manifest allows the system to support multiple agent networks, though this project only uses one.

### 2. airline_policy.hocon

This is the main configuration file that defines:
- The LLM configuration
- Common definitions shared across agents
- The entire agent hierarchy
- Tool connections

It follows this structure:

```hocon
{
    "llm_config": { ... },            # LLM model configuration
    "commondefs": { ... },            # Shared definitions and instructions
    "tools": [                        # List of all agents and tools
        {
            "name": "Airline 360 Assistant",
            "function": { ... },      # Top-level agent definition
            "instructions": "...",
            "tools": ["Baggage_Handling", "Flights", "International_Travel"]
        },
        {
            "name": "Baggage_Handling",
            # Agent definition
            "tools": ["Carry_On_Baggage", "Checked_Baggage", ...]
        },
        # More agent definitions...
        {
            "name": "ExtractPdf",
            # Tool definition with class implementation
            "class": "extract_pdf.ExtractPdf"
        }
    ]
}
```

## How HOCON Files Are Loaded and Used

### Loading Process

The HOCON files are loaded in this sequence:

1. When the system starts, the `run.py` script sets the environment variable `AGENT_MANIFEST_FILE`:
   ```python
   os.environ["AGENT_MANIFEST_FILE"] = "./registries/manifest.hocon"
   ```

2. The Neuro-SAN server reads the manifest file:
   ```python
   # Inside neuro_san.service.agent_main_loop
   manifest_file = os.environ.get("AGENT_MANIFEST_FILE")
   ```

3. The server loads each HOCON file referenced in the manifest:
   ```python
   # Pseudocode from the Neuro-SAN framework
   for hocon_file, enabled in manifest.items():
       if enabled:
           load_hocon_configuration(hocon_file)
   ```

4. The `airline_policy.hocon` configuration is parsed and used to construct the agent hierarchy.

### Code References

The relevant code is spread across multiple files:

1. **run.py**: Sets environment variables pointing to the manifest
   ```python
   def set_environment_variables(self):
       os.environ["AGENT_MANIFEST_FILE"] = "./registries/manifest.hocon"
       os.environ["AGENT_TOOL_PATH"] = "./coded_tools"
   ```

2. **neuro_san.service.agent_main_loop**: Loads the manifest and configurations
   - This is part of the proprietary Neuro-SAN framework in the wheel files.

## How Agents and Tools Are Instantiated

The Neuro-SAN framework uses the HOCON configuration to:

1. Create instances of agents based on their definitions
2. Link agents together according to the hierarchy
3. Connect agents to tool implementations

The process works like this:

1. For each entry in the "tools" array:
   - If it has a "class" field, it's a tool implementation
   - Otherwise, it's an agent definition

2. For tool implementations, the framework:
   - Imports the specified class from the `AGENT_TOOL_PATH`
   - Instantiates the class
   - Registers it with the tool registry

3. For agent definitions, the framework:
   - Creates an agent instance
   - Configures it with the specified instructions
   - Links it to its tools (other agents or tool implementations)

## Visual Representation of HOCON Usage

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│               │     │               │     │               │
│    run.py     │────►│   manifest.   │────►│ airline_policy│
│               │     │    hocon      │     │    .hocon     │
│               │     │               │     │               │
└───────────────┘     └───────────────┘     └───────┬───────┘
       Sets                 Lists                    │
  environment              enabled                   │
   variables            configurations               │
                                                     ▼
┌───────────────┐                             ┌─────────────────┐
│               │                             │                 │
│  CodedTool    │◄───────────────────────────┤  Neuro-SAN      │
│Implementations│      Instantiates tools     │  Framework      │
│               │      based on HOCON         │                 │
└───────────────┘                             └─────────────────┘
                                                     ▲
                                                     │
                                                     │
┌───────────────┐                                    │
│               │                                    │
│  Agent        │◄───────────────────────────────────┘
│  Hierarchy    │       Creates agent hierarchy
│               │       based on HOCON
└───────────────┘
```

## Modifying HOCON Files

When you need to modify the system:

### Adding a New Agent

1. Edit `airline_policy.hocon`
2. Add a new entry to the "tools" array:
   ```hocon
   {
       "name": "New_Agent",
       "function": "aaosa_call",
       "instructions": "Your instructions here...",
       "command": "aaosa_command",
       "tools": ["Tool1", "Tool2"]
   }
   ```
3. Add the new agent to its parent's "tools" array

### Adding a New Tool

1. Create a new Python class implementing `CodedTool`
2. Add it to the `coded_tools` directory
3. Add a new entry to the "tools" array in `airline_policy.hocon`:
   ```hocon
   {
       "name": "NewTool",
       "function": {
           "description": "Tool description",
           "parameters": { ... }
       },
       "class": "module_name.ClassName",
       "command": "Tool execution command"
   }
   ```
4. Add the new tool to the agent's "tools" array that should use it

## Using Environment Variables

The system uses two key environment variables to locate HOCON files:

1. `AGENT_MANIFEST_FILE`: Points to the manifest file
   ```bash
   export AGENT_MANIFEST_FILE="./registries/manifest.hocon"
   ```

2. `AGENT_TOOL_PATH`: Points to the directory containing tool implementations
   ```bash
   export AGENT_TOOL_PATH="./coded_tools"
   ```

These are set automatically by `run.py` but can be overridden for custom configurations.

## HOCON Generation at Runtime

For visualization purposes, the system generates HTML diagrams from HOCON files:

```python
def generate_html_files(self):
    """Generate .html files for all registry files except manifest.hocon."""
    for file in glob.glob("./registries/*"):
        if os.path.basename(file) != "manifest.hocon":
            result = subprocess.run(
                [sys.executable, "-m", "neuro_san_web_client.agents_diagram_builder", "--input_file", file],
                stdout=subprocess.PIPE, stderr=subprocess.PIPE, universal_newlines=True
            )
```

These HTML files provide visual representations of the agent hierarchy defined in the HOCON files.