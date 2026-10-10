# Export preparation notes

The published workflows retain the original 49 nodes, connections, prompts, and layout, except for the small configuration changes listed below. The original files were not modified.

- Removed workflow IDs, version IDs, instance metadata, credential binding IDs/names, and cached Google resource names/URLs.
- Replaced private Google file and spreadsheet IDs with descriptive `YOUR_...` placeholders; the spreadsheet tab uses the configurable name `Sheet1`.
- Removed the embedded Adzuna app ID/key from the URL. The HTTP node now uses n8n Custom Auth; the search term is URL-encoded.
- Disabled `hasOutputParser` on the two LLM chains because no output-parser sub-node was attached. The existing downstream Code nodes still parse the returned JSON.
- Gave each workflow a descriptive name and kept exports inactive.

Validation covered JSON parsing, all connection targets and JavaScript node-name references, syntax of each embedded JavaScript program, preservation of node names/counts, and checks that original resource IDs and authentication values are absent. No live workflows, credentials, database resources, or Google documents were created during validation.

The source exports contained an Adzuna API key. Rotate that key before reuse.
