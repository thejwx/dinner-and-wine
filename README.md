# Dinner and Wine MCP Server Guide

A start-to-finish walkthrough for creating, testing, and integrating your **dinnerAndWinePlanner** MCP tool.

---

## 1. Create & Open Your Project Folder  
**What this does:** Sets up an isolated workspace and tells VS Code where to run your commands.

```bash
mkdir ~/VSCode_Workspaces/dinner-and-wine
```

- **In VS Code:** File > Open Folder… → select `~/VSCode_Workspaces/dinner-and-wine` → Open  
- Your integrated terminal now defaults to this folder.

---

## 2. Initialize NPM & TypeScript  
**What this does:** Bootstraps your Node project and enables writing & running TypeScript.

```bash
npm init -y
npm install --save-dev typescript ts-node @types/node
npx tsc --init   # auto-generates tsconfig.json
```

- Creates `package.json` and a ready-to-go `tsconfig.json`.

---

## 3. Install the MCP SDK & Zod  
**What this does:** Brings in the core MCP framework and runtime schema validation.

```bash
npm install @modelcontextprotocol/sdk zod
```

---

## 4. Create `.gitignore`  
**What this does:** Prevents build artifacts and local caches from being checked into Git.

In the project root, create a file named `.gitignore` containing:

```gitignore
node_modules/
dist/
*.tsbuildinfo
.DS_Store
```

---

## 5. Initialize Git & First Commit  
**What this does:** Starts version control so you can track and share your code.

1. **Ensure Git is installed**  
   ```bash
   git --version || brew install git
   ```
2. **Initialize & commit**  
   ```bash
   git init
   git add .
   git commit -m "Initial setup: TypeScript + MCP SDK"
   ```

---

## 6. Write Your MCP Server  
**What this does:** Defines the “dinnerAndWinePlanner” tool and hard-coded cuisine resource.

1. **Create** `src/index.ts` (and `mkdir src` if needed).  
2. **Paste**:

```ts
// src/index.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

(async () => {
  const server = new McpServer({ name: "Dinner and Wine", version: "0.1.0" });

  // Hard-coded cuisine list
  const cuisines = [
    "Italian","French","Mexican",
    "Japanese","Mediterranean","Indian",
    "Thai","Spanish","Korean","Moroccan",
  ];

  // Expose as config://cuisines
  server.resource(
    "cuisineList",
    "config://cuisines",
    async uri => ({
      contents: [{
        uri:      uri.href,
        text:     JSON.stringify(cuisines),
        mimeType: "application/json",
      }],
    })
  );

  // The dinnerAndWinePlanner tool
  server.tool(
    "dinnerAndWinePlanner",
    {
      mainDish:   z.string(),
      sidesCount: z.number().default(2),
      cuisine:    z.string().optional(),
    },
    async ({ mainDish, sidesCount, cuisine }) => {
      const chosenCuisine =
        cuisine ||
        cuisines[Math.floor(Math.random() * cuisines.length)];
      return {
        content: [
          {
            type: "text",
            text: `Plan some ${chosenCuisine} dinner featuring ${mainDish}: suggest ${sidesCount} sides plus a wine pairing, then explain why that pairing works.`,
          },
        ],
      };
    }
  );

  const transport = new StdioServerTransport();
  await server.connect(transport);
})();
```

3. **Save** the file.

---

## 7. Test Locally with the MCP Inspector  
**What this does:** Runs your server and opens a UI to exercise the tool over stdio.

- **Terminal A** (your server):
  ```bash
  npx ts-node src/index.ts
  ```
- **Terminal B** (the Inspector):
  ```bash
  npx @modelcontextprotocol/inspector npx ts-node src/index.ts
  ```
- **In your browser** at the printed URL:
  1. Click **Connect**
  2. Go to **Tools → dinnerAndWinePlanner**
  3. **Enter “chicken”** in the **mainDish** field. Leave **sidesCount** and **cuisine** as they are.
  4. Click **Run Tool**.
  5. Click **Run Tool** again to see a different cuisine.
  6. Type a cuisine (e.g. “Italian”) in the **cuisine** field and click **Run Tool** to persist it.
  7. (Optional) Change **sidesCount** and click **Run Tool**.

---

## 8. Publish to GitHub via CLI  
**What this does:** Creates a GitHub repo and pushes your local commit.

**Open a third terminal** (to keep A & B running):

```bash
gh --version || brew install gh
gh auth login   # follow prompts & paste back the code
cd ~/VSCode_Workspaces/dinner-and-wine
gh repo create dinner-and-wine --public --source=. --remote=origin --push
```

---

## 9. Hook into Roo Code Globally  
**What this does:** Registers your tool in Roo so it’s available in every project.

1. Click the **kangaroo** icon in VS Code’s Activity Bar.
2. In Roo’s panel, click the **MCP** icon (three stacked bars).
3. Click **Edit Global MCP** to open `mcp_settings.json`.
4. Under `"mcpServers"`, add:
   ```json
   "dinnerAndWinePlanner": {
     "transport": "stdio",
     "command":   "npx",
     "args":      ["ts-node", "src/index.ts"],
     "cwd":       "/Users/jeffwilson/VSCode_Workspaces/dinner-and-wine"
   }
   ```
5. Save and click **Done**.
6. You may see it immediately; otherwise close & reopen the MCP panel or click the ↻ **Refresh** icon.

---

## 10. Invoke Your Tool from Any Project  
**What this does:** Demonstrates your server works globally with natural language.

### 10.1 Create & open a test folder
```bash
mkdir ~/VSCode_Workspaces/temp-mcp-test && cd ~/VSCode_Workspaces/temp-mcp-test
code .
```

### 10.2 Re-open Roo Code in that folder
- Click the **kangaroo** icon again.

### 10.3 Ask for dinner ideas
- Select **dinnerAndWinePlanner**.
- Type:  
  ```
  Give me a dinner idea for chicken
  ```
- Approve the prompt and optionally check **Always Allow**.
- Roo displays the response.

### 10.4 Experiment further
- “Plan a Mexican dinner with beef and 3 sides.”
- “What cuisines are available?”
- “I want an Indian tofu dinner.”

---

🎉 **Complete!** You now have a fully integrated **dinnerAndWinePlanner** MCP server. Enjoy planning dinner and wine pairings anywhere.
