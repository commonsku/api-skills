# commonsku API skills

## Install the skills

Install every skill in this repository, not only the one for your task. `commonsku-api-setup` holds the rules for every call, and the other skills need it.

Paste this message into your AI agent to install the skills for you:

```text
Install the commonsku API skills from https://github.com/commonsku/api-skills. Install every skill, not only one.

- Install them the way your agent installs skills or plugins from a GitHub repository. If your agent supports plugins, install the repository as a plugin, because a plugin includes every skill.
- If you cannot work out how, or the install fails, run `npx skills add commonsku/api-skills`. This needs Node 22.20 or later.
- If a step is a slash command or a menu choice, tell me what to do instead of running it yourself.

When you finish, tell me what you installed and anything I still need to do.
```

Or choose an install method yourself:

- [Install as a plugin in your agent](#install-as-a-plugin-in-your-agent), with steps for each agent.
- [Install by hand](#install-by-hand), for chat apps and coding tools.
- [Install in any agent](#install-in-any-agent), with the `skills` command.

## Install as a plugin in your agent

Many agents can install this repository as a plugin, with their own command. A plugin includes all the skills. Find your agent below.

Each section says whether we tested it, or whether the steps come from the agent's own documentation. Agents change their commands often. If a step no longer works, check your agent's help pages, or use the `skills` command below.

<details>
<summary>Claude Code</summary>

First, add this repository as a marketplace:

```sh
claude plugin marketplace add commonsku/api-skills
```

Then, inside Claude Code, install the plugin:

```text
/plugin install commonsku-api-skills@commonsku-api-skills
```

To get updates automatically, run `/plugin` in Claude Code and turn on auto-update under Marketplaces.

</details>

<details>
<summary>OpenAI Codex</summary>

First, add this repository as a marketplace:

```sh
codex plugin marketplace add commonsku/api-skills
```

Then install the `commonsku-api-skills` plugin: run `/plugins` in the Codex CLI, or open the Plugins tab in the Codex app. Plugins are not available in the Codex IDE extension.

To get updates, run:

```sh
codex plugin marketplace upgrade commonsku-api-skills
```

From Codex's documentation, checked November 2026.

</details>

<details>
<summary>GitHub Copilot CLI</summary>

```sh
copilot plugin install commonsku/api-skills
```

To get updates, run:

```sh
copilot plugin update commonsku-api-skills
```

From GitHub's documentation, checked November 2026.

</details>

<details>
<summary>VS Code with GitHub Copilot</summary>

1. Turn on the `chat.plugins.enabled` setting.
2. Open the Command Palette and run **Chat: Install Plugin from Source**.
3. Enter `https://github.com/commonsku/api-skills`.

To get updates, run **Chat: Update Plugins**.

From VS Code's documentation, checked November 2026.

</details>

<details>
<summary>Cursor</summary>

1. In Cursor, open **Customize**.
2. Choose **Import**, then **From GitHub Repository**.
3. Enter `https://github.com/commonsku/api-skills`.

The plugin works in the Cursor editor, on the web and in the Cursor CLI. To get updates, refresh the marketplace in Cursor.

From Cursor's documentation, checked November 2026.

</details>

<details>
<summary>Gemini CLI</summary>

Gemini CLI needs `git` for this step.

```sh
gemini extensions install https://github.com/commonsku/api-skills
```

To get updates, run:

```sh
gemini extensions update commonsku-api-skills
```

Google replaced Gemini CLI with Antigravity CLI on 18 June 2026 for unpaid-tier and Google One users. If that includes you, follow the next section.

From Google's documentation, checked November 2026.

</details>

<details>
<summary>Antigravity</summary>

```sh
agy plugin install https://github.com/commonsku/api-skills
```

Plugins installed in the Antigravity app update automatically. The documentation lists no update command for the Antigravity CLI. To get a newer version there, uninstall the plugin with `agy plugin uninstall`, then install it again.

From Google's documentation, checked November 2026.

</details>

<details>
<summary>Muse Code</summary>

First, add this repository as a marketplace. This example names it `commonsku`:

```sh
muse plugins marketplace add commonsku commonsku/api-skills
```

Then install the plugin:

```sh
muse plugins install commonsku-api-skills@commonsku
```

To get updates, refresh the marketplace:

```sh
muse plugins marketplace update commonsku
```

Then remove the plugin with `muse plugins remove`, and install it again with the command above.

Muse Code is a developer preview.

From Muse Code's documentation, checked November 2026.

</details>

<details>
<summary>Pi</summary>

```sh
pi install git:github.com/commonsku/api-skills
```

Add `-l` to install it for the current project only. To get updates, run:

```sh
pi update git:github.com/commonsku/api-skills
```

</details>

<details>
<summary>Grok Build</summary>

Use the `skills` command in [Install in any agent](#install-in-any-agent). It supports Grok Build.

</details>

## Install by hand

Each skill is a folder under `skills/`, such as `skills/commonsku-api-setup`. Every tool expects the same thing: the skill's folder, with `SKILL.md` directly inside it.

### Step 1: Get the folder

On this page, click **Code** > **Download ZIP**, unzip it, and open `skills`.

For the chat apps below, zip each skill's folder on its own. Do not upload the whole download. `SKILL.md` must be at the top level of the zip, or the upload is rejected or the skill does not load.

### Step 2: Add it to your tool

**Chat apps.** Upload the zip on the app's Skills page.

| App | Where | Notes |
| --- | --- | --- |
| Claude (web and desktop) | Customize > Skills > + > Upload a skill | Code execution must be turned on. [Claude help](https://support.claude.com/en/articles/12512180-using-skills-on-claude-ai) |
| ChatGPT | Skills > Create > Upload from your computer | Not available on every plan, and workspace admins can turn it off. [ChatGPT help](https://help.openai.com/en/articles/20001066-skills-in-chatgpt) |
| Gemini (web and Mac app) | Skills > Create > Upload | Accepts a folder or a zip. Upload is not available in the mobile app. [Gemini help](https://support.google.com/gemini?p=pn_skills) |

**Coding tools.** Copy the skill's folder into the tool's skills folder. A personal folder makes it available in every project. A project folder shares it with everyone working in that repository.

| Tool | Personal folder | Project folder |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| OpenAI Codex | `~/.agents/skills/` | `.agents/skills/` |
| Gemini CLI | `~/.gemini/skills/` or `~/.agents/skills/` | `.gemini/skills/` or `.agents/skills/` |
| GitHub Copilot (VS Code, CLI) | `~/.copilot/skills/` or `~/.agents/skills/` | `.github/skills/` or `.agents/skills/` |
| Cursor | `~/.claude/skills/` or `~/.codex/skills/` | `.cursor/skills/` |

`~/.agents/skills/` is the shared location in the open standard, and more tools read it over time. Restart the tool, or start a new session, after adding the folder.

**Other tools.** Any tool that supports the Agent Skills standard can use these folders. Check its documentation for where skills go. See [agentskills.io](https://agentskills.io) for the list of tools.

**Tools without skills support.** You can get most of the value by pasting the contents of `SKILL.md` into the tool's custom instructions and attaching the files in `references/` as knowledge files, for example in a ChatGPT Project or custom GPT, or a Gemini Gem. The assistant will not switch the skill on by itself, so check its answers against the live documentation.

Tool menus and folder locations change often. These steps were checked in November 2026. If one no longer matches, your tool's help pages have the current steps.

### What the skills need from your tool

- **Internet access**, to read commonsku's live API spec. If your tool cannot reach the spec, the skills tell the assistant to say so and ask you for the file, rather than answering from memory.
- **Code execution** is strongly recommended. The API spec is a large file that many tools cut short when they read it directly, which can make an endpoint that exists look missing. With code execution, the assistant downloads the whole file and searches it. Without it, treat any answer that something "doesn't exist" with extra care.

## Install in any agent

The `skills` command-line tool installs the skills into your AI agent. It works with many agents, including Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot, Grok Build and Pi. It needs Node 22.20 or later.

```sh
npx skills add commonsku/api-skills
```

To get the latest versions of the skills, run:

```sh
npx skills update
```

## Try it

Ask your assistant things like:

- "I want new commonsku invoices to reach our accounting system. What are my options?"
- "Can I set the customer PO on a sales order through the commonsku API?"
- "My commonsku webhook signature check keeps failing. Here is my handler."
- "GET /products returns 403 but GET /clients works with the same key. Why?"
- "Our bill export to the webhook stays pending in commonsku. What are we missing?"

## Find the API documentation

The commonsku public API documentation is at `https://api.commonsku.com/public_api_doc.php`. The OpenAPI spec is the same address with `?format=json` added.

## Accuracy

The skills tell your assistant to read the live API spec before answering anything specific, so endpoint and field answers follow the API as it is today. The reference files are dated snapshots and say so.

AI tools can still be wrong. Check anything that matters against the live documentation, and test against a record you do not mind changing before you run an integration on real orders. commonsku does not review what your assistant produces.

## Questions and feedback

For questions, bug reports, or problems with the skills themselves, email <support@commonsku.com>.
