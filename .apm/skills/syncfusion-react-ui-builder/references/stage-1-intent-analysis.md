# Stage 1: Intent Analysis

**Purpose:** Parse and validate the user's natural language request, identify component type and modifiers, resolve ambiguities.

**AI Should:**
- Read the user's raw query carefully
- Identify primary intent: `generate_component`, `generate_page`, or `modify_existing`
- Extract component type (e.g., "login form" → form/login, "data table" → table/data)
- Extract modifiers (e.g., "dark theme" → styling:dark, "with OAuth" → feature:oauth)
- Identify target directory if specified (e.g., "in the auth folder" → targetDir:auth/)

**Ambiguity Resolution:**
If the request is unclear, ask ONE clarifying question. Examples:

| Ambiguous Input | Clarifying Question |
|---|---|
| "Build me a form" | "What kind of form? (login, registration, contact, multi-step)" |
| "Add a component" | "What component would you like? (data table, navigation, modal, etc.)" |
| "Make it better" | "Which component and what aspect? (accessibility, styling, layout)" |

## ⚠️ MANDATORY: Service URL Detection & Adaptor Gate Flag

**During Stage 1, the agent MUST scan the user prompt for any service URL pattern.**

### Detection Rule

Apply this check to every incoming user prompt:

```
IF prompt contains a URL pattern (http:// or https://)
AND prompt does NOT contain any of these exact adaptor keywords:
  ["OData v4", "ODataV4", "odata/v4", "OData v3", "ODataAdaptor",
   "Web API", "WebAPI", "ASP.NET API", "Items and Count",
   "URL adaptor", "UrlAdaptor", "result and count",
   "GraphQL", "GraphQL endpoint", "mutations",
   "custom adaptor", "CustomAdaptor", "in-memory", "Entity Framework direct"]
THEN:
  → Set dataBindingStatus = "AMBIGUOUS_URL_DETECTED"
  → Record detectedUrl = {extracted URL}
  → Append warning to Stage 1 output (see template below)
  → Stage 3 MUST present the human gate before advancing to Stage 4
```

### Stage 1 Output Addition (When URL Detected Without Adaptor)

Append this block at the end of the Stage 1 confirmation message:

```
⚠️ Service URL Detected: {detectedUrl}
   Adaptor type: UNKNOWN — Human gate approval required at Stage 3.
   No DataManager code will be generated until the adaptor is confirmed.
```

### Examples

| Prompt | URL Detected | Adaptor Keyword | Action |
|--------|-------------|-----------------|--------|
| `"Build employee app using http://myapi.com/employees"` | ✅ Yes | ❌ None | Flag → gate at Stage 3 |
| `"Create a grid with OData v4 at https://myserver/odata/orders"` | ✅ Yes | ✅ "OData v4" | No gate — ODataV4Adaptor resolved |
| `"Make a dashboard with local list data"` | ❌ No | ❌ N/A | No gate — local binding |
| `"Connect grid to https://myserver/api/employees"` | ✅ Yes | ❌ None | Flag → gate at Stage 3 |
| `"Use my REST API at http://custom-url/data"` | ✅ Yes | ❌ "REST" is ambiguous | Flag → gate at Stage 3 |

---

**Output to User:**
One-line confirmation:
```
✓ Understood: Generating a dark-themed login form with "Remember Me" support.
Starting project detection...
```

**Status:** This stage requires NO user interaction for confirmation. AI decides intent based on pure reasoning.
