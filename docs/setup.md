# Setup guide

## 1. Connect credentials

Use the [credential map](credentials.md) to reconnect every service node after import. Keep secrets in n8n's credential store.

For workflow 1's `adzuna api` node, choose **Generic Credential Type → Custom Auth** and create a credential with this structure, entering your own values in the credential editor:

```json
{
  "qs": {
    "app_id": "YOUR_ADZUNA_APP_ID",
    "app_key": "YOUR_ADZUNA_APP_KEY"
  }
}
```

The public workflow contains neither value. n8n's [HTTP Request credential documentation](https://docs.n8n.io/integrations/builtin/credentials/httprequest/#using-custom-auth) describes Custom Auth query parameters.

NVIDIA embedding and chat nodes need your NVIDIA API credential. Supabase nodes need your project connection credential. Google Drive, Sheets, and the Google Docs HTTP request require their respective OAuth credentials and access to the selected resources. Enable the required Google APIs in your own project when configuring OAuth.

## 2. Prepare Supabase

Create a vector-enabled project with `jobs` and `user_profiles` tables. Both must expose `id`, `content`, `metadata`, and `embedding` columns compatible with the n8n Supabase Vector Store node. Use JSONB for `metadata`, and configure the vector dimension to match the output of `nvidia/nv-embedqa-e5-v5` in your deployment.

Configure a similarity matching function that searches **jobs**, returns the document fields plus similarity, and accepts the arguments expected by the vector-store integration. Set **Options → Query Name** in workflow 2's `job retrieval` node to your matching function. Selecting `jobs` as the table does not automatically rewrite a function that queries another table.

Start from the [Supabase LangChain setup](https://supabase.com/docs/guides/ai/langchain) and [n8n Supabase Vector Store documentation](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoresupabase/). Adapt table names and embedding dimensions instead of copying defaults unchanged.

Workflow 1 stores several metadata keys with trailing spaces, including `source `, `title `, `company `, `location `, `url `, and `description  `. Workflow 2 reads those same exact keys. Keep the names consistent if you rename them.

## 3. Prepare the profile and resume template

Upload a UTF-8 candidate-profile `.txt` file to Google Drive. It should describe skills, education, actual experience, and three projects, using only supported facts. The same file is used for profile indexing and as the master resume source. A fictional example is in [candidate-profile.txt](../examples/candidate-profile.txt).

Set `YOUR_CANDIDATE_PROFILE_TEXT_FILE_ID` in:

- Workflow 2: `get user skill list`.
- Workflow 3: `Download file`.

Create a Google Docs resume template and replace `YOUR_RESUME_TEMPLATE_DOCUMENT_ID` in workflow 3's `Copy file` node. Its exact replacement tokens are:

```text
{{summary}}
{{project_1_title}}
{{project_1_description}}
{{project_2_title}}
{{project_2_description}}
{{project_3_title}}
{{project_3_description}}
{{skills_list}}
```

See [resume-template.md](../examples/resume-template.md). Put fixed contact, education, and experience sections in your own Docs template as needed; only the eight tokens above are replaced.

## 4. Prepare the shared Google Sheet

Create a spreadsheet with a tab named `Sheet1` and these exact headers:

```text
source id,fit_score,recommended_action,title,company,location,source,url,explanation,red_flags,Tailored Resume
```

An importable header CSV is in [job-tracker-headers.csv](../examples/job-tracker-headers.csv). Replace `YOUR_JOB_TRACKER_SPREADSHEET_ID` in workflow 2's `Append row in sheet`, and workflow 3's `Get row(s) in sheet` and `Append or update row in sheet`. Change the tab name in all three nodes if you use another name.

The final node matches on `source id`, not a physical row number. Use distinct IDs and verify that the returned document URL is written to the intended row.

## 5. Index the candidate profile once

In workflow 2, `get user skill list → skill embedder` is an unconnected setup branch, with its data loader and embedding sub-nodes attached. Add a separate Manual Trigger feeding `get user skill list`, then execute that branch alone before the main matching run. Alternatively, populate `user_profiles` through another compatible indexing workflow.

Do not connect indexing and matching as parallel branches of one trigger without ordering them: matching may read the profile table before indexing finishes. Avoid re-indexing the same profile repeatedly without clearing or updating its previous entries.

## 6. Run and inspect

1. Run workflow 1. Check that `jobs` contains document content and complete metadata. If Remote OK arrives as a single array item rather than separate job items in your n8n version, add a Split Out step before the normalizer.
2. Run workflow 2's main trigger after candidate indexing. Check that `user_profiles` rows are retrieved, vector matches refer to jobs, and the sheet contains ranked scores above 50.
3. Start workflow 3 with one test row. Check that the template copy exists, all eight tokens are replaced, and the correct sheet row receives the Docs URL.
4. Review every generated resume before using it. These workflows create documents and recommendations; they do not submit job applications.

The Sheets read node currently retrieves every returned row. Configure a filter for an empty `Tailored Resume` cell before repeat runs to prevent duplicate document creation.
