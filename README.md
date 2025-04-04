# Notion MCP Server (Read-only Version)

MCP Server for the Notion API, enabling Claude to read content from Notion workspaces. This is a read-only version that only supports retrieval operations.

## Setup

Here is a detailed explanation of the steps mentioned above in the following articles:

- English Version: https://dev.to/suekou/operating-notion-via-claude-desktop-using-mcp-c0h
- Japanese Version: https://qiita.com/suekou/items/44c864583f5e3e6325d9

1. **Create a Notion Integration**:

   - Visit the [Notion Your Integrations page](https://www.notion.so/profile/integrations).
   - Click "New Integration".
   - Name your integration and select "Read content" permissions only (this is a read-only version).

2. **Retrieve the Secret Key**:

   - Copy the "Internal Integration Token" from your integration.
   - This token will be used for authentication.

3. **Add the Integration to Your Workspace**:

   - Open the page or database you want the integration to access in Notion.
   - Click the "···" button in the top right corner.
   - Click the "Connections" button, and select the the integration you created in step 1 above.

4. **Configure Claude Desktop**:
   Add the following to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@suekou/mcp-notion-server"],
      "env": {
        "NOTION_API_TOKEN": "your-integration-token"
      }
    }
  }
}
```

or

```json
{
  "mcpServers": {
    "notion": {
      "command": "node",
      "args": ["your-built-file-path"],
      "env": {
        "NOTION_API_TOKEN": "your-integration-token"
      }
    }
  }
}
```

## Environment Variables

- `NOTION_API_TOKEN` (required): Your Notion API integration token.
- `NOTION_MARKDOWN_CONVERSION`: Set to "true" to enable experimental Markdown conversion. This can significantly reduce token consumption when viewing content, but may cause issues when trying to edit page content.

## Advanced Configuration

### Markdown Conversion

By default, all responses are returned in JSON format. You can enable experimental Markdown conversion to reduce token consumption:

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@suekou/mcp-notion-server"],
      "env": {
        "NOTION_API_TOKEN": "your-integration-token",
        "NOTION_MARKDOWN_CONVERSION": "true"
      }
    }
  }
}
```

or

```json
{
  "mcpServers": {
    "notion": {
      "command": "node",
      "args": ["your-built-file-path"],
      "env": {
        "NOTION_API_TOKEN": "your-integration-token",
        "NOTION_MARKDOWN_CONVERSION": "true"
      }
    }
  }
}
```

When `NOTION_MARKDOWN_CONVERSION` is set to `"true"`, responses will be converted to Markdown format (when `format` parameter is set to `"markdown"`), making them more human-readable and significantly reducing token consumption. However, since this feature is experimental, it may cause issues when trying to edit page content as the original structure is lost in conversion.

You can control the format on a per-request basis by setting the `format` parameter to either `"json"` or `"markdown"` in your tool calls:

- Use `"markdown"` for better readability when only viewing content
- Use `"json"` when you need to modify the returned content

## Troubleshooting

If you encounter permission errors:

1. Ensure the integration has "Read content" permissions (this is a read-only version)
2. Verify that the integration is invited to the relevant pages or databases
3. Confirm the token and configuration are correctly set in `claude_desktop_config.json`
4. Note: Write operations will not work as this is a read-only version

## Available Read-only Tools

This version only supports read-only operations. All tools support the following optional parameter:

- `format` (string, "json" or "markdown", default: "markdown"): Controls the response format. Use "markdown" for human-readable output, "json" for programmatic access to the original data structure.

1. `notion_retrieve_block`

   - Retrieve information about a specific block.
   - Required inputs:
     - `block_id` (string): The ID of the block to retrieve.
   - Returns: Detailed information about the block.

2. `notion_retrieve_block_children`

   - Retrieve the children of a specific block.
   - Required inputs:
     - `block_id` (string): The ID of the parent block.
   - Optional inputs:
     - `start_cursor` (string): Cursor for the next page of results.
     - `page_size` (number, default: 100, max: 100): Number of blocks to retrieve.
   - Returns: List of child blocks.

3. `notion_retrieve_page`

   - Retrieve information about a specific page.
   - Required inputs:
     - `page_id` (string): The ID of the page to retrieve.
   - Returns: Detailed information about the page.

4. `notion_query_database`

   - Query a database.
   - Required inputs:
     - `database_id` (string): The ID of the database to query.
   - Optional inputs:
     - `filter` (object): Filter conditions.
     - `sorts` (array): Sorting conditions.
     - `start_cursor` (string): Cursor for the next page of results.
     - `page_size` (number, default: 100, max: 100): Number of results to retrieve.
   - Returns: List of results from the query.

5. `notion_retrieve_database`

   - Retrieve information about a specific database.
   - Required inputs:
     - `database_id` (string): The ID of the database to retrieve.
   - Returns: Detailed information about the database.

6. `notion_search`

   - Search pages or databases by title.
   - Optional inputs:
     - `query` (string): Text to search for in page or database titles.
     - `filter` (object): Criteria to limit results to either only pages or only databases.
     - `sort` (object): Criteria to sort the results
     - `start_cursor` (string): Pagination start cursor.
     - `page_size` (number, default: 100, max: 100): Number of results to retrieve.
   - Returns: List of matching pages or databases.

7. `notion_list_all_users`

   - List all users in the Notion workspace.
   - Note: This function requires upgrading to the Notion Enterprise plan and using an Organization API key to avoid permission errors.
   - Optional inputs:
     - start_cursor (string): Pagination start cursor for listing users.
     - page_size (number, max: 100): Number of users to retrieve.
   - Returns: A paginated list of all users in the workspace.

8. `notion_retrieve_user`

   - Retrieve a specific user by user_id in Notion.
   - Note: This function requires upgrading to the Notion Enterprise plan and using an Organization API key to avoid permission errors.
   - Required inputs:
     - user_id (string): The ID of the user to retrieve.
   - Returns: Detailed information about the specified user.

9. `notion_retrieve_bot_user`

   - Retrieve the bot user associated with the current token in Notion.
   - Returns: Information about the bot user, including details of the person who authorized the integration.

10. `notion_retrieve_comments`
    - Retrieve a list of unresolved comments from a Notion page or block.
    - Requires the integration to have 'read comment' capabilities.
    - Required inputs:
      - `block_id` (string): The ID of the block or page whose comments you want to retrieve.
    - Optional inputs:
      - `start_cursor` (string): Pagination start cursor.
      - `page_size` (number, max: 100): Number of comments to retrieve.
    - Returns: A paginated list of comments associated with the specified block or page.

## License

This MCP server is licensed under the MIT License. This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.
