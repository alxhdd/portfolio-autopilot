# Portfolio Automation & green GitHub forever 🤖

A dead-simple GitHub Action that scans your repos daily and keeps your portfolio current — automatically.

Every project you work on gets a JSON file. Mine is called portf.json but I'm bad with names. You can call yours whatever you want, just update the YAML in that case. The Action reads those files across all your repos and writes two JSON outputs your portfolio site can fetch:

- `showcase.json` — all projects flagged for the portfolio
- `latest.json` — the single project with the most recent commit (your "currently working on")

No database. No backend. No credentials on the frontend. The portfolio just fetches a file.

As a side effect: daily commits to your output repos keep your GitHub contribution graph green without doing anything extra.

---

## How it works

```
your repos (private or public)
    └── portf.json  ← describes the project

          ↓ GitHub Action scans daily

portfolio-projects/showcase.json   ← all portfolio projects
latest-and-greatest/latest.json    ← most recently committed project

          ↓ your portfolio site fetches

yoursite.com  ← always current, zero maintenance
```

---

## Setup

### 1. Create two output repos

Create two new public repos in your GitHub account:

- `portfolio-projects`
- `latest-and-greatest`

They can be completely empty to start.

### 2. Create a Personal Access Token

Go to **GitHub Settings → Developer Settings → Personal Access Tokens → Tokens (classic)**.

Generate a new token with `repo` scope. Copy it.

### 3. Add the token as a secret

In whichever repo will host the Action, I suggest a private one:

**Settings → Secrets and variables → Actions → New repository secret**

- Name: `GH_TOKEN`
- Value: the token you just created

### 4. Add the workflow

Create `.github/workflows/update-portfolio.yml` in that repo and paste in the YAML from `update-portfolio.yml` in this repo.

Change the `USERNAME` value at the top of the `env` block to your GitHub username.

### 5. Add JSON file to your project(s)

Copy contents of `portf.json` from this repo into the root of any project you want in the loop. Fill it in — or leave the placeholders and come back to it later. Mine is automatically added when I spin any new project, but you can handle it manually. 

Trigger the Action manually once (**Actions → Auto-Update Portfolio → Run workflow**) to verify everything works.

After that, it runs every day at 04:00 UTC on its own.

---

## The JSON fields

| Field                | Type    | Required | Description                                            |
| -------------------- | ------- | -------- | ------------------------------------------------------ |
| `public_name`        | string  | yes      | Display name for the project                           |
| `kicker`             | string  | no       | One-line tagline shown under the name                  |
| `url`                | string  | yes      | Project URL. Use `"#"` if not live yet                 |
| `description`        | string  | yes      | Short description for the portfolio card               |
| `skip`               | boolean | yes      | Set to `true` to exclude this project entirely         |
| `portfolio`          | boolean | yes      | Set to `true` to include in `showcase.json`            |
| `card_type`          | string  | no       | Defaults to `"standard"`. I use it for different frontend renders |
| `tech`               | array   | no       | Tech stack tags                                        |
| `detail_description` | array   | no       | Array of paragraphs for an expanded project view       |
| `links`              | array   | no       | CTA links. Each needs a `label` and `url`              |

**The `skip` flag** takes priority over everything. Even if `portfolio: true`, a skipped project is ignored by the Action. Useful when you're working on something you're not ready to surface. 

---

## Reading the output on your site

This heavily depends on your setup. My portfolio is done with next:

```typescript
// Currently working on
async function PortfolioCard() {
  const GIT_URL =
    "https://raw.githubusercontent.com/YOUR_USERNAME/latest-and-greatest/main/latest.json";
  let data = null;

  const res = await fetch(GIT_URL);
  if (res.ok) {
    data = await res.json();
  } else {
    console.error("Failed to fetch data from GitHub :( :", res.status, res.statusText);
  }

// Full portfolio
const GIT_URL =
  "https://raw.githubusercontent.com/YOUR_USERNAME/portfolio-projects/main/showcase.json";

async function Projects() {
  let data = null;

  const res = await fetch(GIT_URL);
  if (res.ok) {
    data = await res.json();
  } else {
    console.error(
      "Failed to fetch data from GitHub :( :",
      res.status,
      res.statusText,
    );
  }
```

Replace `YOUR_USERNAME` with your GitHub username. That's the entire frontend integration.

---

## Adapting it

The Action is intentionally straightforward bash — no obscure dependencies, no framework to learn. The main scan loop is in the `Scan repos and aggregate` step. Common things you might want to change:

- **Cron schedule** — the `0 4 * * *` expression runs at 04:00 UTC. Adjust to whatever suits you.
- **Repo limit** — the Action fetches up to 100 repos per page. If you have more than 100, you'll need to add pagination to the curl call.
- **Output repo names** — if you want different names than `portfolio-projects` and `latest-and-greatest`, update the checkout steps and the commit/push steps accordingly.

---

## Why two output repos instead of one?

Separation of concerns. The `latest.json` output changes almost every day (whenever you commit anything). The `showcase.json` output changes much less frequently — only when you flag a new project or update a description. Keeping them separate means a portfolio page that only shows the showcase doesn't trigger a rebuild every time you push a commit somewhere.

---

Built and maintained by [alxhdd](https://alxhdd.com). Discussed in detail on Medium — [Part 1: the automation](#) · [Part 2: the portfolio](#).
