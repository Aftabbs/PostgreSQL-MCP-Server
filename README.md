# PostgreSQL MCP Server

![image](https://github.com/user-attachments/assets/ad6d595a-271d-4d5e-82eb-c25da0d3ef50)


A Model Context Protocol (MCP) server that provides PostgreSQL database exploration and querying capabilities through Claude. This server enables direct database interaction, schema exploration, and intelligent relationship discovery through natural language queries.

## Features

- **Database Querying**: Execute SQL queries directly through Claude
- **Schema Exploration**: List databases, tables, and get detailed table descriptions
- **Relationship Discovery**: Find both explicit foreign key relationships and implied relationships
- **Intelligent Analysis**: Automatically detect patterns and suggest relationships between tables
- **Safe Query Execution**: Parameterized queries with proper escaping and error handling
- **Comprehensive Logging**: Detailed logging for debugging and monitoring

## Prerequisites

- Python 3.8+
- Docker (for PostgreSQL database)
- Claude Desktop application
- UV package manager

## Installation

### 1. Set up the Project

```bash
# Clone or create your project directory
mkdir postgres-mcp-server
cd postgres-mcp-server

# Initialize with UV
uv init
```

### 2. Install Dependencies

```bash
# Install required packages
uv add psycopg2-binary fastmcp pandas
```

### 3. Set up PostgreSQL with Docker

```bash
# Pull and run PostgreSQL container
docker run --name postgres-mcp \
  -e POSTGRES_DB=testdb \
  -e POSTGRES_USER=testuser \
  -e POSTGRES_PASSWORD=testpass \
  -p 5432:5432 \
  -d postgres:latest

# Wait for PostgreSQL to start (about 10-15 seconds)
# You can check if it's ready with:
docker logs postgres-mcp
```

### 4. Configure Claude Desktop

Add the following configuration to your `claude_desktop_config.json` file:

**On macOS:**
```bash
# Location: ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**On Windows:**
```bash
# Location: %APPDATA%/Claude/claude_desktop_config.json
```

**Configuration:**
```json
{
  "mcpServers": {
    "postgres-explorer": {
      "command": "uv",
      "args": [
        "run",
        "python",
        "/path/to/your/postgres-mcp-server/main.py",
        "--conn",
        "postgresql://testuser:testpass@localhost:5432/testdb"
      ],
      "cwd": "/path/to/your/postgres-mcp-server"
    }
  }
}
```

> **Note**: Replace `/path/to/your/postgres-mcp-server` with the actual path to your project directory.

### 5. Alternative Environment Variable Setup

Instead of passing the connection string in the config, you can set it as an environment variable:

```bash
# Set environment variable
export POSTGRES_CONNECTION_STRING="postgresql://testuser:testpass@localhost:5432/testdb"
```

Then use this simpler configuration:

```json
{
  "mcpServers": {
    "postgres-explorer": {
      "command": "uv",
      "args": [
        "run",
        "python",
        "/path/to/your/postgres-mcp-server/main.py"
      ],
      "cwd": "/path/to/your/postgres-mcp-server",
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://testuser:testpass@localhost:5432/testdb"
      }
    }
  }
}
```

## Usage

### Starting the Server

1. **Start PostgreSQL container** (if not already running):
   ```bash
   docker start postgres-mcp
   ```

2. **Restart Claude Desktop** to load the MCP server configuration.

3. **Verify connection** in Claude by asking:
   ```
   Can you list the available schemas in the database?
   ```

### Example Queries

Once connected, you can interact with your PostgreSQL database through Claude using natural language:

#### Schema Exploration
```
- "Show me all the tables in the database"
- "What's the structure of the users table?"
- "List all schemas available"
```

#### Data Querying
```
- "Show me the first 10 records from the products table"
- "Find all users created in the last 30 days"
- "What are the unique categories in the products table?"
```

#### Relationship Discovery
```
- "What are the relationships for the orders table?"
- "Show me foreign key constraints for the customers table"
- "Find implied relationships in the inventory table"
```

#### Advanced Analysis
```
- "Create a summary of sales by month"
- "Find duplicate records in the users table"
- "Show me tables that might be related to the orders table"
```

### Available Tools

The MCP server provides the following tools that Claude can use:

1. **`query(sql, parameters)`** - Execute SQL queries
2. **`list_schemas()`** - List all database schemas
3. **`list_tables(schema)`** - List tables in a specific schema
4. **`describe_table(table_name, schema)`** - Get table structure
5. **`get_foreign_keys(table_name, schema)`** - Get foreign key relationships
6. **`find_relationships(table_name, schema)`** - Find explicit and implied relationships

## Configuration Options

### Connection String Formats

The server supports various PostgreSQL connection string formats:

```bash
# Basic format
postgresql://username:password@host:port/database

# With additional parameters
postgresql://username:password@host:port/database?sslmode=require

# Using environment variables
postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}
```

### Environment Variables

- `POSTGRES_CONNECTION_STRING` - Database connection string
- `LOG_LEVEL` - Logging level (DEBUG, INFO, WARNING, ERROR)

## Troubleshooting

### Common Issues

1. **"Connection refused" errors**
   - Ensure PostgreSQL container is running: `docker ps`
   - Check if port 5432 is available: `netstat -an | grep 5432`

2. **"Authentication failed" errors**
   - Verify username and password in connection string
   - Check PostgreSQL container logs: `docker logs postgres-mcp`

3. **MCP server not loading**
   - Verify the path in `claude_desktop_config.json` is correct
   - Check Claude Desktop logs for error messages
   - Ensure UV is installed and accessible

4. **Permission errors**
   - Ensure the user has appropriate database permissions
   - Check that the database exists and is accessible

### Testing the Setup

You can test the server independently:

```bash
# Test database connection
uv run python main.py --conn "postgresql://testuser:testpass@localhost:5432/testdb"
```

### Logs and Debugging

The server provides comprehensive logging. Check the logs if you encounter issues:

- Server logs will appear in Claude Desktop's console
- PostgreSQL logs: `docker logs postgres-mcp`
- Enable debug logging by setting `LOG_LEVEL=DEBUG`

## Sample Data Setup

To test the server with sample data:

```sql
-- Connect to your PostgreSQL instance
docker exec -it postgres-mcp psql -U testuser -d testdb

-- Create sample tables
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    total_amount DECIMAL(10,2),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO customers (name, email) VALUES 
('John Doe', 'john@example.com'),
('Jane Smith', 'jane@example.com');

INSERT INTO orders (customer_id, total_amount) VALUES 
(1, 99.99),
(2, 149.50);
```

## Security Considerations

- Use environment variables for connection strings in production
- Implement proper database user permissions
- Consider using connection pooling for high-traffic scenarios
- Enable SSL connections in production environments
- Regularly update dependencies for security patches

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is open source and available under the [MIT License](LICENSE).

## Support

For issues and questions:
- Check the troubleshooting section above
- Review Claude Desktop MCP documentation
- Open an issue in the project repository

