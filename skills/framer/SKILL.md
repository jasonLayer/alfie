---
name: framer
tier: universal
description: >
  Use when the user wants to design, edit, or publish a website or web page — creating layouts, editing sections, updating text or images, managing CMS collections and content, syncing external data, creating or modifying code components, managing color and text styles, handling localization, or publishing deployments. Trigger when the user mentions Framer, references their website or web pages, asks to edit designs, update site content, or work with any Framer project — even if they don't explicitly say 'Framer'. 
  **Mandatory precondition**: run `npx framer-dalton@latest setup` and let it complete **BEFORE** loading this skill.
allowed-tools: ["Bash(npx framer-dalton:*)", "Bash(npx framer-dalton@latest:*)", "Read(/var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/*)", "Write(/var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/*)"]
---

If you didn't run this command before loading the skill, run it now:
```bash
npx framer-dalton@latest setup
```

## Capabilities

What you can do with the Framer CLI:

- **Canvas Editing** For design tasks - creating or editing pages, sections, layouts, recreating designs from screenshots, etc.
- **CMS**: Create, read, update, delete collections and items. Sync external databases.
- **Styles**: Manage color and text styles. Sync design systems.
- **Code Components**: Create, edit, type-check, and add custom React components to the canvas.
- **Assets**: Upload and manage images and files.
- **Localization**: Manage translations programmatically.
- **Data**: Store metadata on nodes and projects for plugin state.
- **Screenshots**: Capture node screenshots as PNG/JPEG. Export nodes as SVG.
- **Publishing**: Publish projects, manage deployments, track changes.
- **Low-level Node APIs**: Create and modify individual nodes. Only use these for targeted, surgical edits to specific nodes - not for building pages or layouts.

## CLI Usage

### Permissions

The Framer CLI requires network access (to download packages and connect to Framer servers) and file-system access outside the project directory (to install skills into `~/.agents/skills/` and `~/.claude/skills/`). In sandboxed environments, you must execute `npx framer-dalton` commands with elevated permissions from the get-go. Do not try to execute them with the default sandboxed permissions; they will hang or fail.

### Required Workflow

Every task follows these steps:

#### 1. Connect (once per session)

If the user refers to a project by name or description rather than giving a URL, first inspect the recent projects:

```bash
npx framer-dalton project list
```

Use that list to infer the likely project from the names and recency. If the right project is already known, use its project ID with `session new`. If there are multiple possible matches, clarify with the user. Only ask the user for a Project URL if there is no clear match. Avoid talking about the technical contents of returned data like IDs.

Create a session against an existing project:

```bash
npx framer-dalton session new "<url or id>"
```

To create a brand new empty project and connect to it:

```bash
npx framer-dalton project new
npx framer-dalton session new "<returned project id>"
```

To remix (duplicate) an existing project and connect to the copy:

```bash
npx framer-dalton project remix "<url, project id, or remix link>"
npx framer-dalton session new "<returned project id>"
```

Note that during beta, you cannot connect to non-beta projects. If creating a session errors with a message about this, you need to move your project to beta.

#### 2. Look up the API (before EVERY code execution)

**You MUST run `npx framer-dalton docs` before writing any code.** Do not guess method names or signatures.

```bash
npx framer-dalton docs Collection           # What methods exist?
npx framer-dalton docs Collection.getItems  # What are the parameters and return type?
```

#### 3. Execute code

Only after checking docs, write your code to a unique file under `/var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/` and execute with `-f`. Name each file `<sessionId>-<short-summary>.js` where `<short-summary>` is a brief kebab-case description (e.g., `1-read-collections.js`, `1-add-team-member.js`).

```bash
npx framer-dalton exec -s 1 -f /var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/1-read-collections.js
```

#### 4. Store results in `state`

Always save results you'll need again. Don't repeat API calls.

**Do not skip step 2.** The examples below are patterns only - always verify current signatures with `npx framer-dalton docs`.

## Core Usage Principles

- Be concise. Don't narrate implementation details like field IDs, escaping, or internal steps. Just do the work and report what was accomplished in user-facing terms.
- Use `framer.*` for plugin API calls. Top-level methods are not globals.
- Before making changes that add/update/delete content that the user has not clearly and explicitly requested in this conversation, inform the user of what you plan to do and why, and ask them to confirm.
    - You do not need to ask for confirmation when carrying out a specific add/update/delete change that the user has already clearly requested (this counts as explicit consent).
    - You do not need to explain or ask for confirmation for non-destructive calls like reading project state.
    - If the exact action was not agreed upon, or you are inferring a broader change or choosing between multiple reasonable options using your own judgment, ask for confirmation.
    - Always ask for confirmation before destructive actions that the user did not explicitly request - especially deletes, cleanup, resets, or broad removals inferred by the agent.

## Context Variables

- `framer` - Connected Framer Server API instance
- `state` - Object persisted between calls within your session
- `console` - For output (`console.log`, `console.error`)
- `require` - Sandboxed Node.js modules: fs, path, url, crypto, buffer, util, os
- Standard globals: `fetch`, `Buffer`, `URL`, `crypto`, `setTimeout`

**Note:** `fs` operations are sandboxed to cwd, /tmp, and os.tmpdir().

## Use `state` to Avoid Repeated Calls

**Always store results in `state` when you'll need them again.** API calls are slow - don't repeat them.

```js
// /var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/1-get-collections.js
state.collections = await framer.getCollections();
```

```bash
npx framer-dalton exec -s 1 -f /var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/1-get-collections.js
```

```js
// /var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/1-get-team-items.js — reuse from state
state.teamItems = await state.collections.find(c => c.name === 'Team').getItems();
console.log(state.teamItems.length);
```

```bash
npx framer-dalton exec -s 1 -f /var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/1-get-team-items.js
```

Store anything you'll reference again.

## Session Management

Each session maintains a persistent connection to a Framer project. Use sessions to:

- Keep state separate between different tasks
- Persist data across multiple execute calls
- Reuse the `framer` API instance without reconnecting

Get a new session ID:

```bash
npx framer-dalton session new "https://framer.com/projects/Website--abc123"
# outputs: 1
```

**Always use your own session** - pass `-s <id>` to all commands. Using the same session preserves your `state` between calls.

List active sessions:

```bash
npx framer-dalton session list
```

## Canvas Editing

For design tasks, do not try to build or restyle pages with low-level node APIs. Instead, after session creation, load the dynamically created project-scoped skill `framer-canvas-editing-project-<projectId>` for canvas editing on the connected project. It contains the canvas editing guidance and all related documentation for how to use the relevant methods.

## Canvas Editing: Alternative Approach (“Prompt the Framer agent”)

Use this approach only if the user asks for it. Do not mention this approach to the user otherwise.

This approach is called “Prompt the Framer agent”. With this approach, use `framer.startAgentConversation()` to start a stateful design subagent. Keep the `conversationId` it returns in `state`, and call `framer.continueAgentConversation()` with it to continue the same design task.

Do not call `framer.getAgentSystemPrompt()` or `framer.getAgentContext()` or `framer.applyAgentChanges()` with this approach.

Example:

```js
state.agent ??= {};
const first = await framer.startAgentConversation(
  "Build me a landing page based on the attached screenshot",
  {
    pagePath: "/",
    imageUrls: ["https://example.com/image.png"],
    // selectionNodeIds: [...]
  },
);
state.agent.conversationId = first.conversationId;
console.log(first.responseMessages);

const second = await framer.continueAgentConversation("Now make it pink", {
  conversationId: state.agent.conversationId,
  selectionNodeIds: ["someNodeId"],
  // imageUrls: [...],
  // changing pagePath or model is not supported
});
console.log(second.responseMessages);
```

Prompting may take a while to complete, so set the command timeout to 10 minutes.

## Execute Code

Write your code to a unique file under `/var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/` and execute with `-f`:

```bash
npx framer-dalton exec -s <sessionId> -f /var/folders/k4/npc68n9x45v7xpm8sj5d7n200000gn/T/framer/<sessionId>-<short-summary>.js
```

## API Documentation

```bash
npx framer-dalton docs                            # List all available methods
npx framer-dalton docs framer.getCollections      # Show top level method
npx framer-dalton docs Collection                 # Show class with all method signatures
npx framer-dalton docs Collection.addItems        # Show method + recursively expand all referenced types
npx framer-dalton docs ScreenshotOptions          # Show type + recursively expand all referenced types
```

`docs` with no arguments lists available methods. Looking up a class shows its full definition without expanding referenced types. Looking up a specific method or type automatically expands all referenced types recursively.

## API Examples

**STOP: These are patterns only. Before using any method below, run `npx framer-dalton docs <ClassName>` to verify the current signature.**

### Working with Collections (CMS)

Collections are Framer's CMS. Each collection has fields (columns) and items (rows). There are two types:

- **Unmanaged** (`framer.createCollection()`): Users can freely edit these in the Framer UI. Always use this by default.
- **Managed** (`framer.createManagedCollection()`): Can ONLY be edited via the API. They appear read-only in the Framer UI. Only use managed collections when explicitly asked or when creating a script for data synchronisation.

#### Reading Collections

```js
// Get all collections
const collections = await framer.getCollections();
console.log(collections.map((c) => ({ name: c.name, id: c.id })));

// Get a specific collection by ID
const collection = await framer.getCollection("collection-id");

// Get fields (columns) - returns array of { id, type, name }
const fields = await collection.getFields();
console.log(fields);
// [{ id: "BnNuS2i3o", type: "string", name: "Title" }, ...]

// Get items (rows)
const items = await collection.getItems();
console.log(items);
// [{ id: "XTM8FSHGs", slug: "post-1", draft: false, fieldData: {...} }, ...]
```

#### Field Types

`boolean`, `color`, `number`, `string`, `formattedText` (HTML), `image`, `file`, `link`, `date`, `enum`, `collectionReference`, `multiCollectionReference`, `array` (galleries)

#### Updating Collection Items

```js
// Add or update items (if id matches existing item, it updates)
await collection.addItems([
  {
    id: "new-item-1",
    slug: "hello-world",
    fieldData: { titleFieldId: "Hello World" },
  },
]);

// Remove items
await collection.removeItems(["item-id-1", "item-id-2"]);

// Reorder items
const ids = items.map((i) => i.id).reverse();
await collection.setItemOrder(ids);
```

#### Working with CollectionItem

```js
const items = await collection.getItems();
const item = items[0];

// Update item attributes
await item.setAttributes({ slug: "new-slug" });

// Navigate to item in Framer UI
await item.navigateTo();

// Remove item
await item.remove();
```

#### Managed Collections

```js
// Create a managed collection
const managed = await framer.createManagedCollection({ name: "My Sync" });

// Set fields
await managed.setFields([
  { id: "title", name: "Title", type: "string" },
  { id: "content", name: "Content", type: "formattedText" },
  { id: "published", name: "Published", type: "date" },
]);

// Add items
await managed.addItems([
  {
    id: "1",
    slug: "first-post",
    fieldData: { title: "Hello", content: "<p>World</p>" },
  },
]);

// Store sync metadata
await managed.setPluginData("lastSync", new Date().toISOString());
const lastSync = await managed.getPluginData("lastSync");
```

### Working with Nodes

Nodes are layers on the canvas: frames, text, images, components, etc.

#### Getting Nodes

```js
// Get current selection
const selection = await framer.getSelection();

// Get canvas root
const root = await framer.getCanvasRoot();

// Get node by ID
const node = await framer.getNode("node-id");

// Get children of a node
const children = await framer.getChildren(node.id);

// Get parent
const parent = await framer.getParent(node.id);

// Find nodes by type
const frameNodes = await framer.getNodesWithType("FrameNode");
const textNodes = await framer.getNodesWithType("TextNode");

// Find nodes with specific attribute set
const nodesWithBg = await framer.getNodesWithAttributeSet("backgroundColor");
const nodesWithImage = await framer.getNodesWithAttributeSet("backgroundImage");
```

#### Creating Nodes

```js
// Create a frame
const frame = await framer.createFrameNode({
  name: "My Frame",
  width: 200,
  height: 100,
  backgroundColor: "rgba(255, 0, 0, 1)",
});

// Create text (creates TextNode, then use setText)
const textNode = await framer.createTextNode({ name: "My Text" });
await textNode.setText("Hello World");

// Add to specific parent
const child = await framer.createFrameNode({ name: "Child" }, parentNode.id);
```

#### Modifying Nodes

```js
// Update attributes
await node.setAttributes({
  name: "New Name",
  width: 300,
  height: 200,
  backgroundColor: "rgba(0, 0, 255, 0.5)",
  visible: true,
  locked: false,
});

// Move to different parent
await framer.setParent(node.id, newParentId);

// Clone a node
const clone = await node.clone();

// Remove nodes
await framer.removeNodes([node.id]);
```

#### Working with Text

```js
// Get text content
const text = await framer.getText(); // from selection
const nodeText = await textNode.getText();

// Set text
await framer.setText("New text"); // on selection
await textNode.setText("Hello World");
```

### Working with Images

```js
// Add image to canvas
await framer.addImage({
  image: "https://example.com/image.png",
  name: "My Image",
  altText: "Description",
});

// Set image on selected node
await framer.setImage({
  image: "https://example.com/image.png",
  altText: "Description",
});

// Upload image without adding to canvas (for later use)
const imageAsset = await framer.uploadImage({
  image: "https://example.com/image.png",
  name: "My Image",
  altText: "Alt text",
});

// Use uploaded image when creating a frame
await framer.createFrameNode({ backgroundImage: imageAsset });

// Get image from selection
const image = await framer.getImage();
console.log(image?.url);

// Add SVG (must be < 10kb)
await framer.addSVG({
  svg: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20"><circle cx="10" cy="10" r="8"/></svg>',
  name: "circle.svg",
});
```

### Working with Styles

#### Color Styles

```js
// List all color styles
const colorStyles = await framer.getColorStyles();

// Get specific style
const style = await framer.getColorStyle("style-id");

// Create color style (supports light/dark themes)
const newStyle = await framer.createColorStyle({
  name: "Brand/Primary", // Use / for folders
  light: "rgba(242, 59, 57, 1)",
  dark: "rgba(180, 40, 40, 1)",
});

// Update style
await style.setAttributes({ light: "rgba(0, 100, 200, 1)" });

// Remove style
await style.remove();
```

#### Text Styles

```js
// List all text styles
const textStyles = await framer.getTextStyles();

// Create text style
const heading = await framer.createTextStyle({
  name: "Typography/Heading 1",
  tag: "h1",
  fontSize: "48px",
  lineHeight: "1.2em",
  fontWeight: 700,
});

// With breakpoints for responsive typography
const responsive = await framer.createTextStyle({
  fontSize: "24px",
  minWidth: 1280,
  breakpoints: [
    { minWidth: 1024, fontSize: "20px" },
    { minWidth: 768, fontSize: "18px" },
    { minWidth: 320, fontSize: "16px" },
  ],
});

// Update text style
await heading.setAttributes({ fontSize: "52px" });
```

#### Fonts

```js
// Get available fonts
const fonts = await framer.getFonts();

// Get specific font
const font = await framer.getFont("Inter", { weight: 400 });
const boldFont = await framer.getFont("Inter", { weight: 700 });

// Use font in text style
await framer.createTextStyle({
  name: "Body",
  font,
  boldFont,
});
```

### Code Components

Code components are custom React components that run inside Framer. Use them when you need behavior that Framer's visual tools don't support:

- **Custom interactivity** — drag-to-reorder, gesture-driven animations, games, form validation
- **External data** — fetching from APIs, rendering dynamic content, real-time updates
- **Complex logic** — state machines, calculations, conditional rendering beyond simple variants
- **Third-party libraries** — maps, charts, video players, rich text editors
- **Canvas/WebGL** — custom drawing, 3D rendering, generative art

If the design can be built with Framer's built-in components, layout tools, and interactions — don't use a code component. Code components are harder to maintain and can't be visually edited.

#### Workflow

1. **Create** a code file: `framer.createCodeFile("MyComponent.tsx", code)`
2. **Edit** an existing code file: `codeFile.setFileContent(newCode)`
3. **Type check**: `codeFile.typecheck()` or `framer.typecheckCode("File.tsx", code)`
4. **Add to canvas**: `framer.addComponentInstance({ url: codeFile.exports[0].insertURL })`

Use `npx framer-dalton docs CodeFile` and `npx framer-dalton docs Framer.createCodeFile` to look up the full API.

#### Authoring guidance

Before writing component code, load the `framer-code-components` skill.

### Storing Data

Store metadata on nodes or globally in the project.

```js
// Store global project data
await framer.setPluginData("myKey", "myValue");
const value = await framer.getPluginData("myKey");

// Store data on a node
await node.setPluginData("processed", "true");
const nodeData = await node.getPluginData("processed");

// List all keys
const keys = await framer.getPluginDataKeys();
const nodeKeys = await node.getPluginDataKeys();

// Delete data (set to null)
await framer.setPluginData("myKey", null);
```

### Localization

```js
// Get all locales
const locales = await framer.getLocales();
const defaultLocale = await framer.getDefaultLocale();

// Get localization groups (pages, CMS items with translations)
const groups = await framer.getLocalizationGroups();

// Update translations
const french = locales.find((l) => l.code === "fr");
await framer.setLocalizationData({
  valuesBySource: {
    [sourceId]: {
      [french.id]: { action: "set", value: "Bonjour" },
    },
  },
});
```

### Common Patterns

#### Iterate over all nodes in project

```js
const root = await framer.getCanvasRoot();
for await (const node of root.walk()) {
  console.log(node.name);
}
```

#### Sync external data to collection

```js
const collection = await framer.getManagedCollection();
const existingIds = new Set(await collection.getItemIds());

const externalData = await fetch("https://api.example.com/posts").then((r) =>
  r.json(),
);

const items = externalData.map((post) => ({
  id: post.id,
  slug: post.slug,
  fieldData: { title: post.title, content: post.body },
}));

await collection.addItems(items);

// Remove items no longer in external source
const newIds = new Set(items.map((i) => i.id));
const toRemove = [...existingIds].filter((id) => !newIds.has(id));
if (toRemove.length) await collection.removeItems(toRemove);

await collection.setPluginData("lastSync", new Date().toISOString());
```

#### Batch update all images' alt text

```js
const nodes = await framer.getNodesWithAttributeSet("backgroundImage");
for (const node of nodes) {
  if (!node.backgroundImage) continue;
  await node.setAttributes({
    backgroundImage: node.backgroundImage.cloneWithAttributes({
      altText: "Updated description",
    }),
  });
}
```

### Screenshots and SVG Export

Server API exclusive methods for capturing visual output from nodes.

```js
// Take a screenshot of a node (returns Buffer + mimeType)
const result = await framer.screenshot(node.id);
console.log(result.mimeType); // "image/png"
const os = require("os");
await require("fs").promises.writeFile(
  os.tmpdir() + "/screenshot.png",
  result.data,
);

// With options
const jpg = await framer.screenshot(node.id, {
  format: "jpeg", // "png" (default) or "jpeg"
  quality: 90, // JPEG quality 0-100 (default 100)
  scale: 2, // Pixel density: 0.5, 1, 1.5, 2, 3, or 4 (default 1)
  clip: { x: 0, y: 0, width: 100, height: 100 }, // Clip region in CSS pixels
});

// Export as SVG string
const svgString = await framer.exportSVG(node.id);
await require("fs").promises.writeFile(os.tmpdir() + "/export.svg", svgString);
```

### Publishing and Deployments

Server API exclusive methods for publishing and managing deployments.

```js
// Publish the project (creates a new deployment)
const result = await framer.publish();
console.log(result.deployment.id); // Deployment ID
console.log(result.hostnames); // Array of hostnames

// Get all deployments
const deployments = await framer.getDeployments();
for (const d of deployments) {
  console.log(d.id, d.createdAt);
}

// Deploy a specific deployment to domains
const hostnames = await framer.deploy(deploymentId, ["example.com"]);

// Get current publish info (production and staging URLs)
const info = await framer.getPublishInfo();
console.log(info.production?.url); // Production URL
console.log(info.staging?.url); // Staging URL
```

### Change Tracking

Server API exclusive methods for tracking project changes.

```js
// Get changed paths since last publish
const changes = await framer.getChangedPaths();
console.log(changes.added); // New paths
console.log(changes.modified); // Modified paths
console.log(changes.removed); // Removed paths

// Get contributors to changes (returns user IDs)
const contributors = await framer.getChangeContributors();
```

### Known Limitations

- **Pages**: Cannot change the path of a page
- **Collection Index/Detail Pages**: Cannot be created as new pages; the user must create them through the UI. Once they exist, they can be modified normally through canvas editing.
- **Code overrides**: Cannot assign overrides to nodes
- **Analytics**: No APIs exist for accessing analytics data
