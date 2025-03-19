# PDF Extraction and URL Provider Tools Documentation

This document explains how the `ExtractPdf` and `URLProvider` tools work within the Airline Helpdesk Chatbot system.

## Overview

The system uses two primary tools to access information:

1. **ExtractPdf**: Reads and extracts text content from PDF and text files
2. **URLProvider**: Returns URLs to relevant airline policy webpages

Both tools follow an on-demand processing model rather than preprocessing. This means they retrieve information at runtime when called by agents.

## ExtractPdf Tool

### Processing Model: On-Demand Text Extraction

The `ExtractPdf` tool performs **runtime extraction** of document content. When an agent needs information from a policy document, it calls this tool which:

1. Locates the appropriate documents based on the agent's domain
2. Extracts text from those documents at runtime
3. Returns the extracted content to the requesting agent

This approach has several advantages:
- No need for a separate preprocessing step
- Content is always up-to-date with the latest documents
- Reduced storage requirements
- Simplified implementation

### Implementation Details

```python
class ExtractPdf(CodedTool):
    def __init__(self):
        # Path mapping: agent domains to document directories
        self.default_path = ["knowdocs/Help Center _ United Airlines.pdf"]
        self.docs_path = {
            "Bag Issues": "knowdocs/baggage/bag-issues",
            "Carry On Baggage": "knowdocs/baggage/carryon",
            "Checked Baggage": "knowdocs/baggage/checked",
            # Additional mappings...
        }
    
    def invoke(self, args: Dict[str, Any], sly_data: Dict[str, Any]):
        # Get the app name (agent domain)
        app_name: str = args.get("app_name", None)
        
        # Look up the directory containing relevant documents
        directory = self.docs_path.get(app_name, self.default_path)
        
        # Dictionary to store extracted document content
        docs = {}
        
        # Traverse directory and process files
        for root, dirs, files in os.walk(directory):
            for file in files:
                file_path = os.path.join(root, file)
                
                # Extract PDF content
                if file.lower().endswith(".pdf"):
                    content = self.extract_pdf_content(file_path)
                    rel_path = os.path.relpath(file_path, directory)
                    docs[rel_path] = content
                    
                # Extract text file content
                elif file.lower().endswith(".txt"):
                    content = self.extract_txt_content(file_path)
                    rel_path = os.path.relpath(file_path, directory)
                    docs[rel_path] = content
        
        # Return results to the agent
        return {"files": docs}
```

### Processing Flow

When an agent calls the `ExtractPdf` tool:

1. The agent provides its domain name (e.g., "Checked Baggage")
2. The tool looks up the corresponding document directory
3. The tool processes all PDFs and text files in that directory:
   - For PDFs: Uses PyPDF2 to extract text while preserving pagination
   - For text files: Reads the raw content
4. The extracted content is returned to the agent as a dictionary
5. The agent uses this information to answer user queries

### Runtime Processing Benefits

- **Always Current**: Documents are read at runtime, ensuring agents always use the latest content
- **Memory Efficient**: Content is only loaded when needed, not stored in memory permanently
- **Flexible**: New documents can be added without modifying code or rebuilding indexes

## URLProvider Tool

### Processing Model: Predefined URL Mapping

The `URLProvider` tool uses a **static mapping** approach:

- URLs are predefined in the tool's initialization
- At runtime, the tool simply looks up the requested URL by name
- No processing is performed beyond this simple lookup

### Implementation Details

```python
class URLProvider(CodedTool):
    def __init__(self):
        # Static mapping of domains to URLs
        self.airline_policy_urls = {
            "Baggage Tracking": "https://www.united.com/en/us/bagdelivery/start",
            "Damaged Bags Claim": "https://rynnsluggage.com/",
            "Missing Items": "https://www.united.com/en/US/fly/help/lost-and-found.html",
            # Additional URL mappings...
        }
    
    def invoke(self, args: Dict[str, Any], sly_data: Dict[str, Any]):
        # Get the app name
        app_name: str = args.get("app_name", None)
        
        # Look up and return the URL
        app_url = self.airline_policy_urls.get(app_name)
        return app_url
```

### Processing Flow

When an agent calls the `URLProvider` tool:

1. The agent provides the name of the policy or feature (e.g., "Checked Baggage")
2. The tool looks up the corresponding URL in its predefined dictionary
3. The URL is returned to the agent
4. The agent includes this URL in its response to the user

### Static Mapping Benefits

- **Fast Execution**: Simple dictionary lookup with no processing overhead
- **Predictable Results**: URLs are consistent across all interactions
- **Centralized Management**: All URLs are defined in one location for easy updates

## Comparison of Approaches

| Feature | ExtractPdf | URLProvider |
|---------|------------|-------------|
| Processing Timing | Runtime (on-demand) | None (static mapping) |
| Data Source | PDF and text files | Hardcoded dictionary |
| Storage | Reads directly from filesystem | In-memory dictionary |
| Update Method | Replace PDF/text files | Modify code and restart |
| Execution Speed | Slower (file I/O and processing) | Very fast (dictionary lookup) |
| Memory Usage | Low (temporary during extraction) | Minimal (small dictionary) |

## Integration with Agent System

Agents interact with these tools through a standardized interface:

```hocon
{
    "name": "Checked_Baggage",
    "function": "aaosa_call",
    "instructions": "...",
    "command": "aaosa_command",
    "tools": ["ExtractPdf", "URLProvider"]
}
```

When an agent like `Checked_Baggage` needs information:

1. It first calls `ExtractPdf` with its domain name to get document content
2. It processes this content to find answers to the user's question
3. It then calls `URLProvider` to get relevant URLs to include in its response
4. It combines the extracted information and URLs into a comprehensive answer

## Extending the Tools

### Adding New Documents

To add new policy documents:

1. Place the PDFs or text files in the appropriate directory under `knowdocs/`
2. Update the `docs_path` dictionary in `ExtractPdf` if adding a new domain

No preprocessing is required as documents are read at runtime.

### Adding New URLs

To add new policy URLs:

1. Update the `airline_policy_urls` dictionary in `URLProvider`
2. Restart the application to load the updated URLs

## Performance Considerations

### ExtractPdf Optimization

For large document collections:
- Consider implementing caching for frequently accessed documents
- Add document filtering to only extract text from relevant files
- Implement text indexing for faster search within documents

### URLProvider Optimization

The current implementation is already highly optimized as a simple dictionary lookup.