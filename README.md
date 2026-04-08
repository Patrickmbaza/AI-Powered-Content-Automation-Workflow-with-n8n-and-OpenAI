# AI-Powered Content Automation Workflow with n8n

This project demonstrates an AI content automation workflow built with:

- `n8n` as the no-code workflow orchestrator
- the OpenAI `Responses API` for content generation
- Markdown as the intermediate content format
- `MkDocs` as the static site generator and preview layer

The workflow accepts a product name, generates three versions of product marketing copy, writes the content to a Markdown file, and serves that content through a static site.

This repo also includes:

- a mock workflow for offline/demo-safe testing
- a real OpenAI workflow for live AI generation
- a Docker Compose setup for self-hosted n8n
- a lightweight Node writer script that works inside the n8n container

## What This Project Does

The workflow automates this sequence:

1. A user triggers a workflow manually inside `n8n`.
2. The workflow receives a product name such as `FocusFlow AI`.
3. OpenAI generates three content variants:
   - short ad copy
   - social media copy
   - landing page copy
4. The workflow converts that output into a Markdown page.
5. The Markdown page is saved under `docs/generated/`.
6. `MkDocs` serves the content as a browsable static website.

This is useful as a demo of:

- AI-assisted content generation
- no-code/low-code orchestration
- structured AI output handling
- automated content publishing workflows

## Skills Demonstrated

- n8n workflow design
- OpenAI API integration
- structured JSON output parsing
- automated Markdown generation
- static site publishing with MkDocs
- debugging self-hosted automation infrastructure

## Architecture Overview

The working OpenAI-based flow is:

`Manual Trigger -> Set Product -> OpenAI Responses API -> Parse OpenAI JSON -> Write Markdown File`

### Node responsibilities

- `Manual Trigger`
  Starts the workflow manually from the n8n editor.

- `Set Product`
  Stores the product name input for the run.

- `OpenAI Responses API`
  Sends the product name to OpenAI and requests structured JSON output.

- `Parse OpenAI JSON`
  Extracts the structured fields from the OpenAI response and base64-encodes the payload so it can be safely passed into a shell command.

- `Write Markdown File`
  Calls a Node script inside the container to create or update a Markdown file in `docs/generated/`.

- `MkDocs`
  Serves the `docs/` folder locally as a static preview site.

## Project Structure

- `n8n/ai-powered-content-automation-workflow.json`
  Mock version of the workflow. Uses a local `Code` node instead of a real OpenAI API call.

- `n8n/ai-powered-content-automation-workflow-openai.json`
  Real OpenAI version of the workflow using `POST /v1/responses`.

- `scripts/render_content.py`
  Python writer script kept as an additional reference implementation.

- `scripts/render_content.mjs`
  Node-based Markdown writer used by n8n inside the Docker container.

- `docs/index.md`
  MkDocs homepage.

- `docs/generated/`
  Generated Markdown output pages.

- `mkdocs.yml`
  MkDocs configuration.

- `docker-compose.yml`
  Self-hosted n8n setup with the project mounted to `/files`.

## Requirements

You should have the following installed locally:

- Docker and Docker Compose
- Python 3
- `pip`
- a browser
- an OpenAI API key for the real workflow

## Quick Start

### 1. Start n8n

Run from the project root:

```bash
docker compose up -d
```

Then open:

`http://localhost:5678`

If this is your first time using n8n, complete the initial owner account setup.

### 2. Install MkDocs

Install dependencies:

```bash
python3 -m pip install -r requirements.txt
```

### 3. Start the static site preview

```bash
mkdocs serve
```

Then open:

`http://127.0.0.1:8000`

## Available Workflows

### Option A: Mock workflow

Import:

- `n8n/ai-powered-content-automation-workflow.json`

Use this when:

- you want to demo the workflow without spending API credits
- you want a predictable output every time
- you want to test the Markdown generation flow first

### Option B: Real OpenAI workflow

Import:

- `n8n/ai-powered-content-automation-workflow-openai.json`

Use this when:

- you want live AI-generated output
- you want to demonstrate the OpenAI integration
- you want varied copy across runs

## How To Import a Workflow in n8n

1. Open `http://localhost:5678`.
2. In the n8n editor, open the workflow actions menu.
3. Choose `Import from file`.
4. Select either:
   - `n8n/ai-powered-content-automation-workflow.json`
   - `n8n/ai-powered-content-automation-workflow-openai.json`
5. Wait for the workflow canvas to load.
6. Confirm the nodes appear on the editor canvas.
7. In newer n8n versions, use `Publish` instead of `Save`.

## Running the Mock Workflow

1. Import `n8n/ai-powered-content-automation-workflow.json`.
2. Open `Set Product`.
3. Change `productName` if needed.
4. Make sure the workflow ends with `Write Markdown File`.
5. Click `Publish`.
6. Click `Execute workflow`.
7. Check the generated Markdown file under `docs/generated/`.

## Running the Real OpenAI Workflow

### Step 1: Set your OpenAI API key

You have two ways to do this.

#### Recommended for quick demo use

Paste your API key directly into the `Authorization` header inside the `OpenAI Responses API` node:

```text
Bearer sk-...
```

This is the fastest path for a live demo.

#### Environment variable approach

Export the key before starting Docker:

```bash
export OPENAI_API_KEY='your_actual_openai_key'
docker compose down
docker compose up -d
```

Note:

- some n8n setups block `$env` access inside node expressions
- if you see `access to env vars denied`, use the direct header value approach instead

### Step 2: Import the real workflow

Import:

- `n8n/ai-powered-content-automation-workflow-openai.json`

### Step 3: Configure the OpenAI node

Open `OpenAI Responses API` and confirm:

- Method: `POST`
- URL: `https://api.openai.com/v1/responses`
- Header `Content-Type`: `application/json`
- Header `Authorization`: `Bearer sk-...` or your approved env expression if enabled

### Step 4: Confirm the parsing node

The `Parse OpenAI JSON` node should contain JavaScript that:

- reads the response payload
- checks for model refusal
- parses the JSON text if needed
- base64-encodes the final content object

### Step 5: Confirm the writer node

The `Write Markdown File` node should execute this command:

```text
node /files/scripts/render_content.mjs --base64 '{{$json.payloadBase64}}'
```

Important:

- the field should evaluate `{{$json.payloadBase64}}`
- the final executed command should start with `node`, not `=node`

### Step 6: Execute the workflow

1. Open `Set Product`
2. Set a product name such as `FocusFlow AI`
3. Click `Publish`
4. Click `Execute workflow`

If successful, the workflow will create or update a page like:

- `docs/generated/focusflow-ai.md`

## Expected Output Format

The generated Markdown page contains:

- page title using the product name
- generation timestamp
- `Short Ad Copy`
- `Social Post`
- `Landing Page Blurb`

Example generated file:

- [focusflow-ai.md](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/docs/generated/focusflow-ai.md)

## How To Check Generated Files

List generated pages:

```bash
ls docs/generated
```

Read a generated page:

```bash
sed -n '1,220p' docs/generated/focusflow-ai.md
```

## How To Preview the Site

Start the local server:

```bash
mkdocs serve
```

Open:

`http://127.0.0.1:8000`

You can then navigate to the generated page from the homepage.

## Example Demo Walkthrough

Use this flow during a live presentation:

1. Open the n8n workflow.
2. Show the `Set Product` node and change the product name.
3. Point to the `OpenAI Responses API` node and explain that it generates structured JSON output.
4. Run the workflow.
5. Switch to the terminal and show the updated file:

```bash
ls docs/generated
sed -n '1,220p' docs/generated/focusflow-ai.md
```

6. Switch to the MkDocs browser preview and show the rendered page.

### Suggested narration

“Here I’m using n8n as the automation layer. I pass in a product name, call the OpenAI Responses API to generate three forms of marketing copy, parse the result as structured JSON, and write that output into Markdown. MkDocs then serves the content immediately as a static site page.”

## Troubleshooting

### `Write Markdown File` shows `python3: not found`

Cause:

- the n8n container does not include Python

Fix:

- use `scripts/render_content.mjs`
- make sure the command starts with:

```text
node /files/scripts/render_content.mjs ...
```

### `Execute Command` node is missing

Cause:

- the node is disabled in the container

Fix:

- ensure `docker-compose.yml` includes:

```yaml
- NODES_EXCLUDE=[]
```

- then recreate the container:

```bash
docker compose down
docker compose up -d
```

### `access to env vars denied`

Cause:

- n8n blocks `$env` inside node expressions

Fix:

- paste the API key directly into the `Authorization` header for demo purposes
- or reconfigure n8n admin settings to allow env access

### `syntax error: unterminated quoted string`

Cause:

- raw JSON was passed into a shell command and the generated content contained quotes or apostrophes

Fix:

- use the base64 flow
- ensure `Parse OpenAI JSON` creates `payloadBase64`
- ensure `Write Markdown File` uses:

```text
node /files/scripts/render_content.mjs --base64 '{{$json.payloadBase64}}'
```

### `/bin/sh: =node: not found`

Cause:

- the command field contained a literal leading `=`

Fix:

- remove the leading `=`
- the actual command must begin with `node`

### JSON parsing errors in the writer script

Cause:

- the command field is treating expressions as literal text
- or the wrong data is being passed into the writer node

Fix:

- confirm the parsing node returns `payloadBase64`
- confirm the writer node uses only the base64 command

## Files You Are Most Likely To Touch

- [docker-compose.yml](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/docker-compose.yml)
- [mkdocs.yml](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/mkdocs.yml)
- [docs/index.md](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/docs/index.md)
- [n8n/ai-powered-content-automation-workflow.json](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/n8n/ai-powered-content-automation-workflow.json)
- [n8n/ai-powered-content-automation-workflow-openai.json](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/n8n/ai-powered-content-automation-workflow-openai.json)
- [scripts/render_content.mjs](/mnt/c/Users/patri/Desktop/AI-PROJECTS/AI-CONTENT-GENERATOR/scripts/render_content.mjs)

## OpenAI API Notes

This project uses the OpenAI `Responses API` and structured outputs.

Relevant references:

- Responses API endpoint: `https://api.openai.com/v1/responses`
- Structured outputs guide: `https://developers.openai.com/api/docs/guides/structured-outputs`
- API keys: `https://platform.openai.com/settings/organization/api-keys`

Why structured output is used here:

- it returns predictable fields
- it reduces parsing ambiguity
- it fits automation workflows better than free-form text

## Submission/Presentation Checklist

Before presenting, confirm:

- `n8n` is running at `http://localhost:5678`
- `MkDocs` is running at `http://127.0.0.1:8000`
- the imported workflow executes successfully
- a file exists under `docs/generated/`
- the generated page loads in the browser
- you can explain each workflow node in one sentence

## License / Usage

This project is intended as a demo and portfolio-style workflow example. Adapt the prompts, page structure, and static site theme as needed for your own use case.
