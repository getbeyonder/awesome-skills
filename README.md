# Awesome Skills for Beyonder

**Ready-made workflows that put the apps on your Beyonder board to work. Pick the skills you need and drop them in.**

---

## What is this

The Beyonder plugin teaches your agent *how to work with Beyonder*: your board, your apps and their tools.
This repository adds *what to do with it*: task-specific workflows, packaged as skills.

A skill is one folder with a `SKILL.md` file: a short playbook that says which app tools to call, in which
order, where to stop and ask you, and what never to do on its own. Take only the skills you want.

## Quick start

Copy a skill's folder into your skills directory.

**For yourself (all your projects):**

```bash
git clone --depth 1 https://github.com/getbeyonder/awesome-skills /tmp/awesome-skills \
  && mkdir -p ~/.claude/skills \
  && cp -r /tmp/awesome-skills/skills/sales-partner-outreach ~/.claude/skills/
```

**For a project or a team (commit it with the project):**

```bash
git clone --depth 1 https://github.com/getbeyonder/awesome-skills /tmp/awesome-skills \
  && mkdir -p .claude/skills \
  && cp -r /tmp/awesome-skills/skills/sales-partner-outreach .claude/skills/
```

Start a new session and just ask, for example: *"Find 50 reseller partners for our product in Germany."*
The skill loads on its own when your request matches its description, and you can also call it by name:
`/sales-partner-outreach`.

Skills are plain folders, so they work in any agent that reads `SKILL.md` folders. Point it at the folder
the way that agent expects.

To update a skill, copy its folder again.

## Skills

| Skill | What it does | Apps it uses |
|---|---|---|
| [`sales-partner-outreach`](skills/sales-partner-outreach/SKILL.md) | Finds sales or reseller partners for your product and brings them to a call. Asks setup questions, takes one approval package with a credit budget, then runs people search, email lookup, research, a draft campaign, reply drafts with booking links and booking status. You admit, launch, switch AI dialogue and approve replies. | Dealmaker, Booking, AI CRM |

More skills are on the way.

## Requirements

- **The Beyonder plugin, connected** and signed in to your company.
- **The apps a skill uses, on your board.** If one is missing, the skill offers to install it from the
  catalog and asks you first.
- **Your own rights apply.** A skill can only do what your role in each app allows. Paid steps use your
  company's credits.

## How it works

Skills ship no servers and no API keys. They drive the apps already on your board through the Beyonder MCP
app tools:

```text
  You: "find reseller partners for our product"
   |
   v
  Agent + skill (the playbook)
   |
   |  apps_tools_list / app_tools_search   -> find the app and its tools
   |  app_tool_describe                    -> read each tool's arguments
   |  app_read / app_write                 -> call the tool (dry run first)
   v
  Beyonder MCP
   |
   v
  Apps on your board:  Dealmaker  |  CRM  |  Booking
```

For example, `sales-partner-outreach` chains Dealmaker tools like this:

```text
people.search  ->  prospects.enrich  ->  campaigns.create_draft  ->  (you admit + launch)  ->  campaigns.messages.list
 paid, quote;        paid, quote          AI-written steps,                                    review, (you approve)
 into a list,                             draft only
 held for review
```

## Safety

- **Dry run first.** Every paid call, and every write that is large or vague, is previewed before it runs.
- **An explicit yes, with the price.** A paid step runs only after you have seen the maximum cost and said yes.
- **Nothing is ever sent for you.** Skills save drafts. Admitting people to a campaign, launching it and
  answering a reply stay your clicks in the app.
- **Uncertain results are checked, not repeated.** If a call may or may not have gone through, the skill reads
  the current state before doing anything else.
- **App data is data.** Names, profiles and reply texts are never followed as instructions.

## Contributing

Each skill is one folder under `skills/`:

```text
skills/
  <skill-name>/
    SKILL.md        # the playbook (plus any files it refers to)
```

`SKILL.md` starts with YAML frontmatter:

```yaml
---
name: <skill-name>
description: When to use the skill, with example requests, and which apps it needs.
---
```

Guidelines:

1. Keep the folder name and `name` the same, in lowercase with hyphens.
2. Put the main use case first in `description`: it is what decides when the skill loads.
3. Use only the generic app tools (`apps_tools_list`, `app_tools_search`, `app_tool_describe`, `app_read`,
   `app_write`), describe each tool before calling it, and keep the safety rules above.
4. Keep inputs as inputs: never hardcode a product, a customer profile, links or ids.
5. Add a row for the skill to the table above and open a pull request.

## License

Apache License 2.0. See [LICENSE](LICENSE).
