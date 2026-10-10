# Credential mapping

Create credentials inside n8n, then select them on the nodes below. No credential exports are included.

| Workflow | Node | n8n credential type |
| --- | --- | --- |
| 1 | NVIDIA Nemotron Embeddings | `nvidiaApi` |
| 1 | adzuna api | `httpCustomAuth` |
| 1 | job embedder | `supabaseApi` |
| 2 | NVIDIA Nemotron Embeddings | `nvidiaApi` |
| 2 | get user skill list | `googleDriveOAuth2Api` |
| 2 | job retrieval | `supabaseApi` |
| 2 | skills getter | `supabaseApi` |
| 2 | NVIDIA Nemotron Embeddings1 | `nvidiaApi` |
| 2 | skill embedder | `supabaseApi` |
| 2 | NVIDIA Nemotron Chat Model | `nvidiaApi` |
| 2 | Get many rows | `supabaseApi` |
| 2 | Append row in sheet | `googleSheetsOAuth2Api` |
| 3 | Get row(s) in sheet | `googleSheetsOAuth2Api` |
| 3 | Download file | `googleDriveOAuth2Api` |
| 3 | NVIDIA Nemotron Chat Model | `nvidiaApi` |
| 3 | Copy file | `googleDriveOAuth2Api` |
| 3 | HTTP Request | `googleDocsOAuth2Api` |
| 3 | Append or update row in sheet | `googleSheetsOAuth2Api` |
