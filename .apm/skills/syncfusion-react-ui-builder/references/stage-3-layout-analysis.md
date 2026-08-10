# Stage 3: Layout Analysis & Component Mapping (Combined)

**Purpose:** Analyze user requirements, create optimal component-mapping.json, and map to Syncfusion components automatically. **FULLY AUTOMATED — NO user interaction (unless component validation required OR data binding adaptor is ambiguous).**

---

## 🛑 MANDATORY DATA BINDING GATE (Runs Before Component Mapping Output)

**This gate MUST be evaluated before completing Stage 3 output.**

### Gate Evaluation Logic

```
IF Stage 1 flagged dataBindingStatus = "AMBIGUOUS_URL_DETECTED"
OR the component mapping includes any data-bound component
   (GridComponent, DropDownListComponent, ComboBoxComponent, ListViewComponent, TreeViewComponent, GanttComponent, PivotViewComponent, etc.)
   AND a service URL was provided in the user prompt
   AND no explicit adaptor keyword was identified in Stage 1
THEN:
  → Add to Component Mapping JSON:
      "dataBinding": {
        "status": "PENDING_HUMAN_GATE",
        "serviceUrl": "{detectedUrl}",
        "adaptorDecision": null,
        "gateRequired": true,
        "reason": "Service URL present but adaptor type unknown"
      }
  → STOP Stage 3 output after the Component Mapping JSON
  → PRESENT the Human Gate using the template in:
       skills/syncfusion-react-data-manager/references/adaptor-decision-gate.md
  → WAIT for user adaptor selection
  → DO NOT advance to Stage 4 until adaptorDecision is confirmed
  → Update "dataBinding.adaptorDecision" with confirmed adaptor
  → Update "dataBinding.status" to "CONFIRMED"
  → Then resume Stage 3 → Stage 4 flow normally
```

### Component Mapping JSON — Data Binding Flag Example

When the gate is pending, the component-mapping.json MUST include this flag:

```json
{
  "component_type": "Employee Dashboard",
  "variant": "Standard",
  "dataBinding": {
    "status": "PENDING_HUMAN_GATE",
    "serviceUrl": "http://customurl/api/employees",
    "adaptorDecision": null,
    "gateRequired": true,
    "reason": "Service URL provided but adaptor type (ODataV4 / WebAPI / UrlAdaptor / GraphQL / Custom) cannot be determined without user confirmation"
  },
  "mapped_components": [...]
}
```

After user confirms:

```json
{
  "dataBinding": {
    "status": "CONFIRMED",
    "serviceUrl": "http://customurl/api/employees",
    "adaptorDecision": "WebApiAdaptor",
    "referenceFile": "web-api-adaptor.md",
    "gateRequired": false
  }
}
```

### Gate Presentation

When presenting the gate to the user, use the **exact template** from:
📄 `skills/syncfusion-react-data-manager/references/adaptor-decision-gate.md`
→ Section: **"Gate Presentation Template"**

After user confirms → proceed to Stage 4 with the confirmed adaptor recorded in the component-mapping JSON.

---

---


## Stage 3: Layout Analysis

### AI Should:

1. **Read component type** from Stage 1 intent analysis
2. **Analyze user query** for specific requirements and context
3. **Determine optimal layout variant** (no user choice needed)
4. **Create structured component-mapping.json** with all elements and **run the script** with component-mapping.json
5. **Output JSON** for Stage 3-4 combined processing

### Decision Framework

Based on user requirements, AI selects the **best** layout (not multiple options):

| Component Type | Decision Criteria | Best Variant |
|---|---|---|
| **Login Form** | Enterprise? 2FA needed? Social login? | Choose: Minimal/Standard/Advanced |
| **Data Table** | Read-only or editable? Export needed? | Choose: Simple/Interactive/Full-featured |
| **Dashboard** | Internal or customer-facing? Complexity? | Choose: Focused/Standard/Enterprise |
| **Navigation** | Mobile traffic high? Multi-level needed? | Choose: Simple/Sidebar/Progressive |
| **Form** | Single-step or multi-step? Validation level? | Choose: Basic/Standard/Advanced |

**Key Principle:** AI makes the optimal choice based on the user query context and best practices. No variant selection UI needed.

### Output: Structured JSON (for Stage 3-4 Component Mapping)

```json
{
  "component_type": "Login Form",
  "variant": "Standard",
  "elements": [
    {
      "id": "email_input",
      "name": "Email Address",
      "description": "Email field with validation",
      "type_hint": "text input email form validation",
      "icon_hint": "email envelope mail user input"
    },
    {
      "id": "password_input",
      "name": "Password",
      "description": "Password field, masked",
      "type_hint": "text input password form masked",
      "icon_hint": "lock password secure"
    },
    {
      "id": "remember_me",
      "name": "Remember Me",
      "description": "Keep me logged in",
      "type_hint": "checkbox form input",
      "icon_hint": "check checkbox remember"
    },
    {
      "id": "submit_button",
      "name": "Submit",
      "description": "Login button",
      "type_hint": "button primary action submit cta",
      "icon_hint": "arrow right send proceed forward"
    }
  ],
  "icon_elements": [
    {
      "id": "forgot_password_link",
      "name": "Forgot Password",
      "description": "Password reset link",
      "type": "link",
      "icon_hint": "help question info reset"
    },
    {
      "id": "error_message",
      "name": "Error",
      "description": "Error indicator",
      "type": "icon",
      "icon_hint": "error warning alert"
    }
  ]
}
```

**JSON Structure Details:**

- `component_type`: The UI component being built (e.g., "Login Form", "Data Table", "Dashboard")
- `variant`: Chosen variant based on user requirements (e.g., "Standard", "Advanced", "Minimal")
- `sections` (optional): For complex layouts, group elements into logical sections
  - `section_id`: Unique identifier (e.g., "header_section")
  - `section_name`: Display name (e.g., "Header")
  - `elements`: Array of elements within section
- `elements`: Array of component elements
  - `id`: Unique identifier (snake_case)
  - `name`: Display name for UI
  - `description`: What this element does
  - `type_hint`: UI element type for BM25 search (e.g., "text input", "button", "dropdown", "table", "chart")
  - `icon_hint`: Icon keywords for EJ2 icon mapping (e.g., "email envelope mail" for email input)
- `icon_elements` (optional): Array of icon-only elements without Syncfusion components
  - `id`: Unique identifier (snake_case)
  - `name`: Display name
  - `description`: What this element does
  - `type`: Element type (e.g., "link", "icon", "spinner", "badge")
  - `icon_hint`: Icon keywords for EJ2 icon mapping

**Important:**
- `type_hint` is critical for Stage 4 ComponentMapper BM25 search accuracy
- Keep descriptions concise and functional
- Use lowercase for `id` and `type_hint`
- Create `component-mapping.json` in project root (reused in Stage 4 & 5)
- Output minimal summary table in chat for visibility

### Type Hint Best Practices

**For Header/AppBar Elements:**
- Always include `appbar` or `header` keyword in type_hint
- Examples:
  - Logo: `"image logo appbar header branding"` (not just `"image logo"`)
  - Notifications: `"icon button notification appbar header"`
  - User Menu: `"dropdown button menu user profile appbar header"`

**General Guidelines:**
- Use **compound keywords** - `"icon button notification"` scores better than `"notification"`
- Include **context keywords** - appbar/header/sidebar context improves matching
- Add **modifiers** - sortable, filterable, collapsible, paginated
- Reference **component keywords from CSV** - Use exact component keywords for best BM25 matching

**Scoring Guide:**
- Score **40+** = Excellent match (multiple keywords + context)
- Score **20-40** = Good match (several keywords)
- Score **<20** = Weak match (may fall back to NATIVE_HTML)
- Score **0** = No match (unrelated keywords)

### Icon Hint Best Practices

**Icon Hint Purpose:** Map elements to EJ2 icons (separate from component mapping)

**For Component Elements (with `icon_hint`):**
- Add semantic keywords related to the element's function
- Examples:
  - Email input: `"email envelope mail user input"`
  - Password input: `"lock password secure"`
  - Submit button: `"arrow right send proceed forward"`
  - Delete button: `"trash delete remove discard"`

**For Icon-Only Elements (in `icon_elements`):**
- Use keywords matching the icon's semantic meaning
- Examples:
  - Help link: `"help question info support"`
  - Error indicator: `"error warning alert critical"`
  - Loading spinner: `"refresh loading spinner sync"`

**Icon Scoring:**
- Score **10+** = Excellent icon match
- Score **5-10** = Good match
- Score **<5** = Weak match (may use emoji fallback)
- Score **0** = No match (no EJ2 icon available)

---

## Complex Layouts (Multi-Section)

For dashboards, admin panels, or multi-section layouts:

```json
{
  "component_type": "Admin Dashboard",
  "variant": "Classic Admin Dashboard",
  "layout_grid": "2-column",
  "sections": [
    {
      "section_id": "header",
      "section_name": "Header",
      "description": "Fixed top navigation",
      "responsive": "fixed",
      "elements": [
        {
          "id": "logo",
          "name": "Company Logo",
          "description": "Brand logo in app bar",
          "type_hint": "image logo appbar header branding"
        },
        {
          "id": "notification_bell",
          "name": "Notifications",
          "description": "Bell icon with count in header",
          "type_hint": "icon button notification appbar header"
        },
        {
          "id": "user_menu",
          "name": "User Profile",
          "description": "User avatar dropdown menu",
          "type_hint": "dropdown button menu user profile appbar header"
        }
      ]
    },
    {
      "section_id": "sidebar",
      "section_name": "Sidebar",
      "description": "Left navigation",
      "responsive": "collapsible",
      "elements": [
        {
          "id": "nav_menu",
          "name": "Navigation Menu",
          "description": "Main navigation",
          "type_hint": "sidebar navigation collapsible"
        }
      ]
    },
    {
      "section_id": "main_content",
      "section_name": "Main Content",
      "responsive": "flexible",
      "elements": [
        {
          "id": "kpi_cards",
          "name": "KPI Cards",
          "description": "Metric cards displaying statistics",
          "type_hint": "card grid statistics dashboard metrics kpi"
        },
        {
          "id": "data_grid",
          "name": "Users Table",
          "description": "Data table with sorting and filtering",
          "type_hint": "grid data grid table sortable filterable paging"
        }
      ]
    }
  ]
}
```

---

## Stage 3-4 File-Based Workflow (TOKEN OPTIMIZATION)

### ⚠️ MANDATORY: Create component-mapping.json in Project Root

**Step 1: Create `component-mapping.json`** in project root (NOT in scripts folder)

Before running the script, confirm the file exists:
```bash
# macOS/Linux
ls <project-root>/component-mapping.json

# Windows
dir <project-root>\component-mapping.json
```
If the file is missing, re-run the JSON creation step above before proceeding. **Do not run the script without this file present.**

**Step 2: Verify Node.js is installed**
```bash
node --version
```
- Required: Node.js 14 or higher (18+ recommended)
- If missing or outdated: **Hard stop** — tell the user:
  ```
  ⚠ Node.js 18+ is required to run the component mapper script.
  Re-run Stage 3 after installing.
  ```
  Do not proceed or use any manual fallback. The script is required for accurate component mapping.

**Step 3: Run ComponentMapper Script**
```bash
cd <project-root>/<skills-dir>/syncfusion-react-ui-builder/scripts
node components-search.cjs <project-root>/component-mapping.json
```

**Step 4: Capture Output in Chat Context**
- ✅ Script outputs component + icon mapping JSON
- ✅ Keep mapping results in conversation context ONLY (no file)
- ✅ Do NOT save script output to file
- ✅ Reference mapping in chat for Stages 4-5

**Script Error Recovery:**

| Error | Fix |
|-------|-----|
| `ENOENT: component-mapping.json not found` | Verify file exists at project root (Step 1), correct the path, retry |
| `node: command not found` | Hard stop — Node.js not installed (Step 2) |
| Empty output `{}` or `[]` | Validate JSON structure — ensure all elements have `type_hint` and `id` in snake_case, recreate file and retry |
| 0 components mapped | Hard stop — do not advance to Stage 4. Share error output for diagnosis |

### Workflow Benefits
| Aspect | Benefit |
|--------|---------|
| **Token Efficiency** | Create component-mapping.json once, reuse in script without re-passing JSON in chat |
| **Persistence** | `component-mapping.json` stays in project for version control + auditing |
| **Scriptability** | Script reads `component-mapping.json` directly from filesystem (faster) |
| **Clean Context** | Only component mapping output in chat (not redundant component-mapping.json) |
| **Reusability** | Stage 5 code generation can reference `component-mapping.json` if needed |

---

## Stage 3-4 Part 2: Component & Icon Picking (Script-Based)

### Input: Component-Mapping JSON from Stage 3

Receive the component-mapping.json created above with `icon_hint` fields for component elements and `icon_elements` for icon-only elements.

### Processing: ComponentMapper + IconMapper Script

**⚠️ MANDATORY STEPS (DO NOT SKIP):**

Follow the 4-step process above: verify file exists → check Node.js → run script → capture output.

**Path examples by skills directory:**

```bash
# .codestudio
cd /my-project/.codestudio/skills/syncfusion-react-ui-builder/scripts
node components-search.cjs /my-project/component-mapping.json

# .agents
cd /my-project/.agents/skills/syncfusion-react-ui-builder/scripts
node components-search.cjs /my-project/component-mapping.json

# visible skills folder
cd /my-project/skills/syncfusion-react-ui-builder/scripts
node components-search.cjs /my-project/component-mapping.json
```

**Path Rules:**
- ✅ Use forward slashes `/` — works on macOS, Linux, and Windows
- ✅ Use absolute paths — avoids "file not found" errors
- ❌ Avoid relative paths like `../component-mapping.json`

**Output Destination:**
- ✅ Component + Icon mapping JSON printed to terminal
- ✅ **Captured in chat context ONLY** (not saved to file)
- ✅ Used for Stage 4 theming + Stage 5 code generation

This automatically:
- Maps elements → Syncfusion components (BM25 search)
- Maps icon_hint → EJ2 icons (BM25 search)
- Produces Stage 4 output JSON with BOTH components and icons
- **Results stay in conversation, not persisted to disk**

### How It Works

1. **Component Mapping**: BM25 search on `type_hint` → finds best Syncfusion component
2. **Icon Mapping**: BM25 search on `icon_hint` → finds best EJ2 icon
3. **Fallbacks**:
   - No component match → `NATIVE_HTML`
   - No icon match → emoji
4. **Output**: Complete mapping with components + icons for Stage 5

### BM25 Search Algorithm

The ComponentMapper uses **BM25 (Best Matching 25)**:

- **Tokenizes** query and component keywords
- **Calculates** term frequency (TF) in each component
- **Calculates** inverse document frequency (IDF) across all components
- **Applies** BM25 formula for semantic relevance ranking
- **Returns** ranked results with scores

**Quality:** Only components with score > 0 are matched; unmatched → NATIVE_HTML

### Output: Component & Icon Mapping

```json
{
  "component_type": "Login Form",
  "variant": "Standard",
  "mapped_components": [
    {
      "element_id": "email_input",
      "element_name": "Email Address",
      "component": "TextBoxComponent",
      "skill": "syncfusion-react-inputs",
      "score": 13.24,
      "icon": {
        "name": "Email",
        "iconCss": "e-icons e-mail",
        "score": 12.5
      }
    },
    {
      "element_id": "password_input",
      "element_name": "Password",
      "component": "TextBoxComponent",
      "skill": "syncfusion-react-inputs",
      "score": 12.87,
      "icon": {
        "name": "Lock",
        "iconCss": "e-icons e-lock",
        "score": 14.1
      }
    },
    {
      "element_id": "remember_me",
      "element_name": "Remember Me",
      "component": "CheckBoxComponent",
      "skill": "syncfusion-react-buttons",
      "score": 11.45,
      "icon": {
        "name": "Check",
        "iconCss": "e-icons e-check",
        "score": 11.8
      }
    },
    {
      "element_id": "submit_button",
      "element_name": "Submit",
      "component": "ButtonComponent",
      "skill": "syncfusion-react-buttons",
      "score": 10.89,
      "icon": {
        "name": "Arrow Right",
        "iconCss": "e-icons e-arrow-right",
        "score": 11.5
      }
    }
  ],
  "mapped_icon_elements": [
    {
      "element_id": "forgot_password_link",
      "element_name": "Forgot Password",
      "element_type": "link",
      "icon": {
        "name": "Help",
        "iconCss": "e-icons e-help",
        "score": 11.2
      }
    },
    {
      "element_id": "error_message",
      "element_name": "Error",
      "element_type": "icon",
      "icon": {
        "name": "Error",
        "iconCss": "e-icons e-error-treeview",
        "score": 13.8
      }
    }
  ]
}
```

---

## Status

✅ **FULLY AUTOMATED** - No user interaction
✅ **Single pass** - component-mapping.json created once, components + icons mapped immediately
✅ **Token efficient** - No duplication or variant selection overhead
✅ **Data-driven** - BM25 semantic search on 76+ Syncfusion components + 20+ EJ2 icons
✅ **Icon Support** - Integrated icon mapping for both component elements and icon-only elements
✅ **Ready for Stage 5** - Component + icon mapping feeds directly to code generation

---

## Architecture

- **Input**: User requirements + component type from Stage 1
- **Processing**:
  - Component analysis → JSON structure with `type_hint` + `icon_hint`
  - ComponentMapper (BM25 algorithm on 76 components)
  - IconMapper (BM25 algorithm on 20+ EJ2 icons)
- **Output**:
  - `component-mapping.json` (project root) - layout structure + icon hints for Stage 4 & 5
  - Chat summary table - element count, components matched, icons matched
   "Icons Selected: [name1], [name2], [name3]"
- **Data sources**:
  - `scripts/components.csv` (Syncfusion components)
  - `scripts/icons.csv` (EJ2 icons)
  - `scripts/components-search.cjs` (BM25 mappers)
- **Artifacts**: `component-mapping.json` (persistent artifact, reused by Stage 5)
- **Context**: Component + icon mapping results kept in conversation (no file artifact)

---

## Stage 3-4 Icon Integration Summary

| Step | Task | Input | Output | Artifact |
|------|------|-------|--------|----------|
| **Stage 3** | Analyze requirements, create component-mapping.json with icon hints | User requirements + component type | `component-mapping.json` with element + icon hints | ✅ `component-mapping.json` |
| **Stage 4** | Map elements to Syncfusion components + icons (script-based) | `component-mapping.json` | Component + icon mapping results | Context only (no file) |
| **Stage 5** | Generate code with components + icons | `component-mapping.json` + component mapping from context | React `.tsx` with icons + styling | ✅ Component files |