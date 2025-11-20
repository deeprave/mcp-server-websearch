# Debugging search-mcp

## Checking Logs

The server now writes debug logs to stderr. Check where Amazon Q logs its output to see what's happening.

### For Amazon Q CLI

Check the log location:
```bash
# Usually in:
tail -f $TMPDIR/qlog/*
# or
tail -f ~/.config/amazon-q/logs/*
# or
tail -f /tmp/qlog/*
```

Look for lines like:
- `Listing available tools` - confirms tools/list is being called
- `Tool call: name='...'` - shows what tool Amazon Q is trying to call
- `Search request: query='...'` - shows what search queries are being sent
- `Returning X results` - shows if results are being found

## Common Issues

### 1. Amazon Q calling wrong tool name

If you see:
```
Error: Unknown tool 'search'
```

Then Amazon Q is calling `search` instead of `web_search`. Check the config to see if the tool name is being overridden.

### 2. Empty queries

If you see:
```
Search request: query='', max_results=...
```

Then Amazon Q is sending empty search queries. This is a client-side issue.

### 3. No results found

If you see:
```
Returning 0 results for query 'xyz'
```

The search is returning no results from DuckDuckGo. Try the query manually:
```bash
python3 << 'EOF'
from search_mcp import search_duckduckgo
result = search_duckduckgo("your query here")
print(result)
EOF
```

## Direct Testing

Run the server in test mode:
```bash
python3 /Users/davidn/Code/search-mcp/search_mcp.py <<EOF
{"jsonrpc": "2.0", "method": "initialize", "params": {"protocolVersion": "2024-11-05", "capabilities": {}, "clientInfo": {"name": "test", "version": "1.0"}}, "id": 1}
{"jsonrpc": "2.0", "method": "tools/call", "params": {"name": "web_search", "arguments": {"query": "your query here"}}, "id": 2}
EOF
```

Watch stderr for debug output.
