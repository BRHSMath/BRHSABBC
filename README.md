# Mr. Le's AP Calculus — practice website (v0.1)

A practice site for **AP Calculus AB and BC**, organized by the units of the College Board Course and Exam Description (CED).

- **Randomized questions with automatic grading.** 57 question types across Units 1–10. Every question is generated from a seed, so each student gets new numbers. Answers can be numbers (`5/2`, `sqrt(3)/2`, `DNE`, `inf`), expressions (`3x^2 - 2/x`), antiderivatives (graded up to `+ C`), or multiple choice. Multi-part questions show parts (a), (b), (c) on separate lines and grade each part.
- **Formula, graph, and table together.** Every question type is tagged with how it presents information (formula, graph, table). Each unit has an AP free-response-style question that combines all three, for example "f is given by a formula, g by a graph, and h by a table; find the derivative of f(x)g(x)". Practice favors a different representation from the last question, and each day's spiral review always includes a formula + graph + table question plus at least one graph and one table.
- **Daily spiral review.** Five questions a day: two from the class's current unit plus three from earlier units, weighted toward units where the student's mastery is low or that they haven't practiced recently. Streaks skip weekends.
- **Teacher dashboard and assignments.** Join codes, a mastery heatmap (students × units), 14-day activity, "needs attention" question types, assignments where each student gets different numbers, per-item reports, student detail with PIN reset, and CSV export.
- **Step-by-step explanations and hints.** Hints open one at a time (using them slightly lowers mastery credit). After answering, the worked solution reveals one step at a time, with a solution graph where it helps.
- **Interactive graphs and tables.** Questions include live graphs (pan, zoom, trace), line-segment graphs with open and closed dots, shaded areas, slope fields, parametric and polar curves, and AP-style data tables. The Graph Lab has a draggable tangent line, f′, Riemann and trapezoidal sums, a table of values, slope fields with solution curves and Euler's method, and polar curves.
- **Two forms for every question.** Teachers can preview any question type (or build a printable worksheet) as a **student form** or a **teacher form** with the answer key, step-by-step explanation, hints, and solution graph.

Zero npm dependencies. Node 18.17 or newer (hosting uses Node 22).

---

## Quick start (your computer)

```bash
cd ap-calc-practice
npm run seed        # optional: demo teacher, 2 classes, 18 students, 3 weeks of history
npm start           # http://localhost:3000
```

After `npm run seed`, sign in as teacher **demo@school.edu / calculus123**. Students join with the class code printed by the seed script, any name in the roster (for example *Maya Chen*), and PIN **1234**.

To start with an empty site instead, skip `npm run seed`, open the site, and choose **Teachers → Create account**.

Run the tests:

```bash
npm test            # every template × many seeds, plus a full API walk-through
```

`scripts/test-bank.js` checks each question type across hundreds of random versions: the LaTeX renders, the answer is verified numerically (derivatives by finite differences, integrals by Simpson's rule, series by partial sums), and the grader accepts the right answer and rejects a wrong one. Install `katex` as a dev dependency (`npm i -D katex`) to enable the LaTeX render check.

### Stand-alone demo file

```bash
npm run build:demo  # writes dist/demo.html
```

`dist/demo.html` is the whole app in one file, running in the browser with sample data and no server. Open it locally, email it to a colleague, or drop it on GitHub Pages or Netlify Drop to show people. It is only a demo: its data stays in that one browser.

---

## Free version: GitHub Pages + Google Sheets

This is the version Mr. Le's site uses. It costs nothing to host.

- **GitHub Pages** hosts the website: one `index.html` plus a tiny `config.js`.
- **A Google Sheet** stores student work. Its attached Apps Script (`Code.gs`) grades answers and saves the data. Every answer appears in the sheet's **Answers** tab.
- The answer key lives only in the Apps Script, so the public GitHub repository never contains it.

Build it with `npm run build:pages` (needs `npm i -D esbuild`). That writes `dist/github-pages/` (upload to GitHub) and `dist/google-apps-script/Code.gs` (paste into the Sheet's Apps Script). The setup steps are in `dist/github-pages/README.md`.

The Apps Script back end is in `src/gas/`. It returns the same data as the Node server, so the same browser app works with both. To try it locally against a simulated spreadsheet, run `npm run preview:pages` and open http://localhost:4300. `npm run test:gas` runs the full flow through the real `Code.gs` with mocked Google services.

Limits to know: a Google account can run 30 requests at the same moment (fine for a class), and a sheet holds 10 million cells (about a million answers). Start a fresh Sheet each school year if you like a clean slate.

## Deploying

The server stores everything in one JSON file, `db.json`, inside `DATA_DIR`. **Whatever host you use, `DATA_DIR` must be on persistent storage**, or student work disappears on the next deploy or restart.

### Configuration

| Variable | Default | What it does |
|---|---|---|
| `PORT` | `3000` | Port to listen on (hosts usually set this for you) |
| `DATA_DIR` | `./data` (or the Railway volume path) | Folder for `db.json` |
| `TIME_ZONE` | `America/New_York` | Decides when "today" starts for spiral review and streaks |
| `SITE_NAME` | `Mr. Le's AP Calculus` | Name in the header and browser tab |
| `TEACHER_SIGNUP_CODE` | *(empty)* | If set, creating a teacher account requires this code. **Set it on any public site.** |
| `TRUST_PROXY` | *(off)* | Set to `1` behind Render, Railway, nginx, or Cloudflare so secure cookies and rate limiting see the real client |

### Option A: Render (simplest)

1. Put this folder in a GitHub repository.
2. In Render, choose **New → Blueprint** and select the repository. `render.yaml` creates a web service (Starter instance) with a 1 GB persistent disk mounted at `/opt/render/project/src/data` and a health check.
3. Render asks for `TEACHER_SIGNUP_CODE`. Type a word only you know. Then visit the site and create your teacher account with it.

Render's filesystem is ephemeral by default and disks are available on paid services, so keep the disk. A service with a disk runs as a single instance and has a few seconds of downtime on each deploy. Render also keeps daily disk snapshots for seven days.

### Option B: Railway

1. Create a project from the GitHub repository. Railway builds the included `Dockerfile`.
2. Add a **Volume** to the service with mount path `/app/data`. The server also picks up `RAILWAY_VOLUME_MOUNT_PATH` automatically.
3. Because volumes mount as root and the image runs as the `node` user, add the variable `RAILWAY_RUN_UID=0`.
4. Add `TEACHER_SIGNUP_CODE`, `TIME_ZONE`, and `TRUST_PROXY=1`, then generate a public domain under **Networking**.

### Option C: Any server with Docker (school server, VPS)

```bash
# edit TEACHER_SIGNUP_CODE in docker-compose.yml first
docker compose up -d
```

Data lives in the `calc-data` volume. Put the site behind HTTPS (Caddy, nginx with Let's Encrypt, or Cloudflare Tunnel) and set `TRUST_PROXY=1`.

Without Docker: install Node 18.17+, copy the folder, and run `node server.js` under a process manager (systemd, pm2) with the variables above.

### Backups

`db.json` is the whole database. Copy it on a schedule (for example a nightly `cp data/db.json backups/db-$(date +%F).json`). To restore, stop the server, put the file back, and start it again.

---

## How it works

```
server.js                    HTTP server: routes, sessions, password hashing, static files
src/shared/rng.js            seeded random numbers (same seed → same question)
src/shared/expr.js           safe math parser: implicit multiplication, sin^2(x), |x|, live LaTeX preview
src/shared/grade.js          grading: AP 3-decimal tolerance, expression equivalence by sampling, + C handling
src/shared/bank/unitN.js     question templates for each CED unit
src/shared/bank/index.js     unit list, template registry, student view vs. teacher view
src/shared/spiral.js         daily spiral selection
src/shared/service.js        classes, students, attempts, mastery, streaks, assignments, dashboard
src/shared/demo.js           sample classes and history for the demo and `npm run seed`
src/client/                  the single-page app (app.js), graphs (graph.js), API clients
src/vendor/katex/            KaTeX, served locally so math renders even when CDNs are blocked
scripts/                     tests, seed, demo builder
```

**Answers never reach the browser before submission.** The server sends only the student view of a question (prompt, graph, table, hints, input types). Grading happens on the server by rebuilding the question from its template id and seed. The `src/shared/bank` folder is not served.

**Grading rules.** Numeric answers follow the AP convention of three decimal places (rounded or truncated); exact forms like `7/3` or `2pi/15` are accepted too. Very small answers are checked to about three significant figures. Expressions are compared at many random points in the domain. Antiderivatives are accepted if they differ from the key by a constant. `log` means natural log.

**Mastery** is a running average of first-attempt scores for each unit, weighted toward recent work. One hint gives 75% credit; two or more give 50%. Retries in practice and spiral review help students learn but don't change mastery. Assignment answers lock after the first submission.

### Adding a question type

Add an object to the right `src/shared/bank/unitN.js` file:

```js
{
  id: 'u2-my-new-type',          // unique
  unit: 2, topic: '2.5', title: 'Power rule warm-up',
  bc: false,                      // true = BC only
  gen(r) {                        // r is a seeded random generator
    const n = r.int(2, 6);
    return {
      prompt: L`Let \(f(x) = x^{${n}}\). Find \(f'(2)\).`,
      answer: { type: 'num', value: n * 2 ** (n - 1), tex: `${n * 2 ** (n - 1)}` },
      hints: [L`Power rule: \(\frac{d}{dx}x^{n} = nx^{n-1}\).`],
      steps: [L`\(f'(x) = ${n}x^{${n - 1}}\)`, L`\(f'(2) = ${n * 2 ** (n - 1)}\)`],
      verify: () => true,          // optional numeric self-check used by npm test
    };
  },
}
```

Use `parts: [{ label: 'a', prompt, answer, steps }, ...]` for multi-part questions, `graph: {...}` for an interactive graph, `table: { rows: [...] }` for a data table, and `solutionGraph` for a graph that appears with the solution. Then run `npm test`.

---

## Security notes for a public deployment

- Teacher passwords and student PINs are hashed with scrypt. Sessions are random tokens in `HttpOnly`, `SameSite=Lax` cookies (`Secure` over HTTPS with `TRUST_PROXY=1`).
- Sign-in endpoints are rate limited per IP.
- Students sign in with class code + name + PIN, so no student email addresses are collected. Teachers can reset a forgotten PIN from the student's page.
- Set `TEACHER_SIGNUP_CODE` so strangers can't create teacher accounts.

## Known limits of this first version

- The JSON file store is fine for a few classes (hundreds of students). For a whole school, move `service.js` onto SQLite or Postgres; the service only touches the `db` object, so the change is contained.
- No Google Classroom or LMS integration yet; the CSV export covers gradebooks for now.
- Free-response questions are auto-graded part by part. Written justifications ("because f′ changes from positive to negative...") are taught in the solutions and multiple-choice reasons, not graded from free text.

## Ideas for the next version

More templates per topic (aim for 3–5 per CED topic), FRQ-style sets that mirror past exam formats, a calculator-active section, Google Classroom roster import, spaced repetition at the template level, and teacher-written custom questions.

AP® is a trademark registered by the College Board, which is not affiliated with, and does not endorse, this site.
