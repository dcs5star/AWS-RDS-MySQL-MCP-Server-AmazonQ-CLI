# Amazon Q CLI with MySQL MCP Server Integration

A comprehensive guide for integrating AWS RDS MySQL MCP server with Amazon Q CLI, enabling natural language database interactions through Amazon Q's conversational interface.

## Overview

This project demonstrates how to configure Amazon Q CLI to work with a FastMCP MySQL server, allowing you to interact with your AWS RDS MySQL database using natural language commands through Amazon Q's chat interface.

## Prerequisites

- Python 3.8 or higher
- Amazon Q CLI installed and configured
- AWS RDS MySQL instance or local MySQL server
- AWS CLI configured with appropriate permissions

## Installation & Setup

### 1. Install Amazon Q CLI

Follow the official installation guide:
- **Reference**: [Installing Amazon Q Developer CLI](https://dev.to/aws/the-essential-guide-to-installing-amazon-q-developer-cli-on-windows-lmh)

### 2. Create Virtual Environment

```bash
# Create isolated virtual environment
python3 -m venv venv-mcp

# Activate virtual environment
# Linux/macOS:
source venv-mcp/bin/activate
```

### 3. Install Dependencies

```bash
pip3 install fastmcp pymysql boto3 python-dotenv
```

### 4. Environment Configuration

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your actual configuration
```

Configure your `.env` file:

```bash
# AWS Configuration
AWS_REGION=us-east-1
BEDROCK_MODEL_ID=anthropic.claude-3-haiku-20240307-v1:0

# MySQL Database Configuration
RDS_HOST=your-mysql-host.amazonaws.com
RDS_PORT=3306
RDS_USER=your-username
RDS_PASS=your-password
RDS_DB=your-database-name
```

**Security Best Practices:**
- Never commit `.env` files to version control
- Add `.env` to your `.gitignore` file
- Use AWS IAM roles in production environments
- Regularly rotate database credentials

## MCP Configuration Options

Amazon Q CLI supports two MCP configuration modes: (We will be implementing Global Configuration)

### Workspace Configuration
- **Scope**: Directory-specific
- **Location**: `/.amazonq/mcp.json` in working directory
- **Usage**: Must run Q CLI commands from the configured directory

### Global Configuration (Recommended) 
- **Scope**: System-wide access
- **Location**: `~/.aws/amazonq/mcp.json`
- **Usage**: Run Q CLI commands from any directory
- **Benefit**: Persistent across system reboots

## Global MCP Setup (Recommended)

### 1. Create Configuration Directory

```bash
# Create Amazon Q configuration directory
mkdir -p ~/.aws/amazonq
```

### 2. Create MCP Configuration File

Create `~/.aws/amazonq/mcp.json` with the following content:

```json
{
  "mcpServers": {
    "rds_mysql_server": {
      "command": "/absolute/path/to/your/venv-mcp/bin/python3",
      "args": [
        "/absolute/path/to/your/fastmcp_server.py"
      ],
      "env": {},
      "timeout": 120000,
      "disabled": false
    }
  }
}
```

**Important**: Replace paths with your actual absolute paths:
- `/absolute/path/to/your/venv-mcp/bin/python3`
- `/absolute/path/to/your/fastmcp_server.py`

### 3. Verify Configuration

```bash
q mcp list
```

**Expected Output:**
```
workspace:
  /.amazonq/mcp.json
    (empty)

global:
  /home/username/.aws/amazonq/mcp.json
    • rds_mysql_server /path/to/venv-mcp/bin/python3
```

## Usage

### Start Amazon Q Chat

```bash
q chat
```

### Example Database Interactions

```
# Natural language queries
"Show me all tables in the database"
"Create a users table with id, name, and email columns"
"Find all records in the users table"
"Insert a new user with name 'John Doe' and email 'john@example.com' in users table"
"Show the structure of the users table"
```

## Security Features

- **Input Validation**: Prevents malicious SQL injection
- **Operation Restrictions**: Blocks dangerous operations (DROP DATABASE, TRUNCATE)
- **Connection Timeout**: 120-second timeout prevents hanging connections
- **Environment Isolation**: Secure credential management through `.env` files
- **Error Handling**: Graceful error responses without exposing sensitive information

## Supported Operations

| Operation | Description | Example |
|-----------|-------------|----------|
| `SELECT` | Query data | "Show all users" |
| `INSERT` | Add records | "Add a new user" |
| `UPDATE` | Modify data | "Update user email" |
| `DELETE` | Remove records | "Delete user" |
| `CREATE` | Create tables | "Create users table" |
| `SHOW` | Display info | "Show all tables" |
| `DESCRIBE` | Table structure | "Describe users table" |

## Troubleshooting

### Common Issues

1. **MCP Server Not Found**
   - Verify absolute paths in `mcp.json`
   - Ensure virtual environment is properly created
   - Check file permissions

2. **Database Connection Failed**
   - Validate `.env` configuration
   - Test database connectivity
   - Check firewall settings

3. **Permission Denied**
   - Verify AWS credentials
   - Check database user permissions
   - Ensure proper IAM roles

### Debug Commands

```bash
# Test MCP server directly
python3 fastmcp_server.py

# Verify environment variables
echo $AWS_REGION

# Check Q CLI configuration
q mcp list
q chat --debug
```

## Configuration Management

### Update MCP Server

1. Stop any running Q chat sessions
2. Update `fastmcp_server.py` or configuration
3. Restart Q chat: `q chat`

### Disable MCP Server

```json
{
  "mcpServers": {
    "rds_mysql_server": {
      "disabled": true
    }
  }
}
```

## Performance Optimization

- **Connection Pooling**: Reuse database connections
- **Query Caching**: Cache frequently accessed data
- **Timeout Management**: Optimize timeout values based on query complexity
- **Resource Monitoring**: Monitor CPU and memory usage

## Next Steps

1. **Extend Functionality**: Add custom tools for specific business logic
2. **Multi-Database Support**: Configure multiple database connections
3. **Advanced Security**: Implement role-based access control
4. **Monitoring**: Set up logging and performance monitoring
5. **Automation**: Create deployment scripts for team environments

---

**Note**: This configuration provides persistent, system-wide access to your MySQL database through Amazon Q CLI's natural language interface. Always follow security best practices when handling database credentials and connections.

