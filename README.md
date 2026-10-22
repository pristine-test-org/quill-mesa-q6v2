# Quill Mesa: Monorepo · Vercel and Render

A monorepo with three apps on two hosts: `clove-k4d` and `dill-m2x` on **Vercel**, `marjoram-r8n` on
**Render** (a static site). Preview links are **public** on both. Static Vite + TypeScript, no
framework. The page reads a live list from the shared Supabase test project. Row in the support
matrix: REN-1099 (Linear, project Onboarding evals).


## Deployment links

| App | Folder | Host | Production | Preview gate |
|---|---|---|---|---|
| Clove | `apps/clove-k4d` | Vercel | https://quill-mesa-q6v2-clove-k4d.vercel.app | no |
| Dill | `apps/dill-m2x` | Vercel | https://quill-mesa-q6v2-dill-m2x.vercel.app | no |
| Marjoram | `apps/marjoram-r8n` | Render | https://quill-mesa-q6v2-marjoram-r8n.onrender.com (static site `srv-dbd1g1avcj2c739bsqvg`) | no |

## Names

Every prepared repository gets a random name of its own, never reused, for example
`amber-falcon-7k2q`. In a monorepo, each of the three apps also gets a random name (folder, package
name and page title). App names must not repeat across repositories: when `todo-monorepo-2` reused
`todo-monorepo`'s app names, the dashboard opened the old repository's apps.

## Prepare it for an eval

Render's GitHub app must see every repository in `pristine-test-org` (check:
`gh api orgs/pristine-test-org/installations --jq '.installations[].app_slug'` lists `render`),
or Render can't build the repository or its pull requests.

1. Copy (don't move) this folder to `tmp/eval-repos/<name>/`, `<name>` being the new random name.
2. Rename the repository folder, the root `package.json` name, and each app: its folder under `apps/`, its `package.json` name, and its title in `index.html`. Replace `clove-k4d`, `dill-m2x`, `marjoram-r8n` and `Clove`, `Dill`, `Marjoram` everywhere. Run `npm install` so `package-lock.json` has the new names.
3. All three apps are renamed and `npm run build` passes for each, then `git init -b main`, commit, and push to a new repository
   `pristine-test-org/<name>` (`gh repo create`; push with the token on the URL). Then add a plain `origin`,
   `git remote add origin https://github.com/pristine-test-org/<name>.git`, with no token in it: the
   command quits without one.
4. **The two Vercel apps** (`clove-k4d`, `dill-m2x`): for each, create a Git-connected Vercel project
   named `<name>-<app>` in the `rg-test` team:
   `vercel api "/v11/projects?teamId=team_iuofJm6Q925kcDS0oDMTk9pb" --method POST --input body.json --scope rg-test`,
   body: `name`, `"framework": "vite"`, `"rootDirectory": "apps/<app>"`, `"gitRepository": {"type": "github", "repo": "pristine-test-org/<name>"}`.
   - Right after, PATCH `/v9/projects/<id>?teamId=team_iuofJm6Q925kcDS0oDMTk9pb` with
     `gitProviderOptions={"createDeployments":"enabled"}`. Projects made through the API start with it
     disabled, and the command finds previews on GitHub's deployments.
   - Turn Vercel Authentication **off** (new projects start with Standard Protection): PATCH
     `/v9/projects/<id>?teamId=team_iuofJm6Q925kcDS0oDMTk9pb` with `"ssoProtection": null`.
5. **The Render app** (`marjoram-r8n`): create a Git-connected static site named `<name>-<app>` in the
   workspace **My Workspace** (`render workspace set tea-dbcte37b9u1s73crd0r0` once):
   `render services create --type static_site --name <name>-<app> --repo https://github.com/pristine-test-org/<name> --branch main --root-directory apps/<app> --build-command "npm ci && npm run build" --publish-directory dist --previews automatic --confirm --output json`.
   `--previews automatic` gives every pull request its own preview site; without it a PR gets none.
6. Push a commit that changes a file in every app's folder (an empty commit builds nothing: the
   hosts skip a project or site a commit doesn't touch). That starts the Vercel projects' first
   production builds, and proves each host builds on push.
7. Each production URL answers 200 and shows rows in its "Live from the shared list" strip. The
   Render one is the service's `serviceDetails.url`.
8. Write the production URLs into **Deployment links** above, delete the template
   notice at the top, and push that to `main`.

A pull request's Render preview is its own static site, `https://<name>-<app>-pr-<n>.onrender.com`,
public like production. Render reports it on GitHub only as a deployment (`<branch> - <name>-<app> PR #<n>`); Vercel's previews are deployments by vercel[bot] (seen 2026-10-22). Render builds the site only when a commit touches its root directory, like Vercel.

## Supabase

Every page reads the latest five rows of the `todos` table in the shared test project
(`https://hszqtfynyogshhltamep.supabase.co`), the same one the todo monorepos use. The URL and publishable key are
defaults in `src/supabase.ts`, so a host needs no environment variables.

**When a new deployment or preview URL comes up**, check Supabase lets it in:

- **Data (CORS):** nothing to add. Supabase's Data API answers any origin; on 2026-10-19 a
  request from a made-up preview origin came back allowed. Open the deployment and check the
  "Live from the shared list" strip shows rows, not "Couldn't reach the list".
- **Auth URLs:** the only per-URL setting Supabase has is Auth → URL Configuration (Site URL and
  Redirect URLs). It matters only for an app that signs people in through Supabase Auth with a
  redirect. These pages don't, so skip it. If an app ever does, add its preview URL pattern to
  Redirect URLs with the Supabase CLI (`supabase link --project-ref hszqtfynyogshhltamep`, then
  `supabase config push`). Careful: `config push` writes the shared project's whole auth config.

## Run the eval

A repo eval, not `npm run eval`: the steps are in `REPO-EVAL.md` beside the templates. In short:
open the repo's app on impeccable.reviews, wait for the scan (Rescan if apps are missing), click
Setup, run the command it gives in `tmp/eval-repos/<name>/`, and wait for Impeccable's review on
the test PR.
