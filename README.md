# Salesforce Apex Documentation for LLMs

This repository contains LLM-friendly documentation for Salesforce Apex, designed to be easily consumed by AI coding agents and language models. The documentation is structured in a way that makes it ideal for context retrieval and code assistance tasks.

## Purpose

This documentation serves multiple purposes:

1. Provide context to coding or code review agents
2. Serve as a resource for Model Context Protocol (MCP) servers
3. Direct learning resource for LLMs about Salesforce Apex

## Features

- Comprehensive coverage of Apex core concepts
- Examples with proper context and explanations
- Structured in markdown format for easy parsing
- Organized by topics for better context retrieval

## Usage

### With MCP Server

This repository is designed to work with the [SETA MCP Server](https://github.com/techformist/seta-mcp).

To use this documentation with the MCP server:

1. Clone this repository:

   ```bash
   git clone https://github.com/prashanth1k/seta-apex-docs-example
   ```

2. Note the path where you cloned the repository (e.g., `c:\mcp\salesforce-apex`)

3. Configure the folder path as a local library using the environment variable. See [SETA MCP documentation](https://github.com/techformist/seta-mcp) for detailed setup instructions.

### Directory Structure

```
apex/
├── manifest.json        # Documentation metadata and topic organization
└── topics/
    ├── apex_core_concepts.md    # Fundamental Apex concepts and features
    ├── apex_data_types.md       # Apex data types and type system
    ├── apex_dml_operations.md   # Database operations and DML statements
    ├── apex_soql_basics.md      # SOQL query basics and best practices
    └── batch.md                 # Batch Apex implementation and patterns
```

## Contributing

Contributions are welcome! Feel free to submit pull requests to improve the documentation or add new topics.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
