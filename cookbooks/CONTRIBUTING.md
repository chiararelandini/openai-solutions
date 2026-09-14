# Contributing a cookbook

This page covers what is specific to `cookbooks/`. For bug reports, the code of conduct,
security disclosure and licensing, see the [repository contributing guide](../CONTRIBUTING.md).

A cookbook here is a short, runnable lesson: someone arrives wanting to know how to do one
thing with an OpenAI model on Amazon Bedrock, and leaves having done it. That goal decides
most of the conventions below, so where a rule seems fussy it is usually protecting the
reader rather than the repository.

## What a cookbook looks like

```text
cookbooks/<group>/<nn>-<slug>/
├── README.md              the lesson: front matter, then the narrative
├── python/<slug>.py       the runnable script
├── typescript/            a README placeholder until the port exists
└── data/                  synthetic inputs, only if the recipe needs them
```

There are five groups — `01-foundations`, `02-reasoning-and-output`,
`03-grounding-and-multimodal`, `04-agents` and `05-production`. They are navigation, not
taxonomy: pick the one a reader would look in.

Directory names are lowercase and hyphenated. Python filenames use underscores, because a
hyphen is not importable. The narrative lives in the recipe's own `README.md` and is not
duplicated into `python/`, since the lesson is the same in any language. `data/` sits at
the recipe root for the same reason.

**Recipes are executable `.py` files, not notebooks.** A script is diffable and lintable,
and notebook output cells are the most common way a secret reaches a public repository.

## Front matter

Every recipe README opens with a YAML block. It is shorter than it looks, because the keys
fall into three groups.

**You fill these in.**

| Key | What it is for |
| --- | --- |
| `title` | The lesson in one line. Quote it if it contains a colon, or the YAML silently stops parsing. |
| `models` | The exact model IDs the recipe runs on, so a reader knows what produced the output they are looking at. |
| `region` | The Region the recipe runs in. Always explicit — a model is not served everywhere. |
| `apis` | Which APIs the recipe uses, such as `[responses]`. |
| `languages` | Which implementations exist today, such as `[python]`. |
| `level` | `beginner`, `intermediate` or `advanced`, from the reader's point of view rather than the code's. |
| `estimated_cost` | `low`, `medium` or `high`. See below — the README has to justify it. |
| `industry` | An industry if the framing changes an engineering decision, otherwise `—`. |
| `industry_scenario` | One or two sentences of setting. Worth writing even when it repeats the opening. |
| `iam_actions` | The actions the script actually calls. Never a managed policy name: a policy's contents change underneath you, and guessing wide is how a reader ends up attaching full access. |
| `status` | `draft` until you have run the recipe end to end yourself. |

**A maintainer adds these.** `capabilities` and `primary_capability` are classification tags
that come from a taxonomy kept outside this repository. Leave them out and we will fill them
in during review — that is not something we expect a contributor to guess.

**These only appear when they apply.** `last_validated` and `validated_with` go in when
`status` is `validated`, recording the date you ran it and the Python and SDK versions you
ran it with. `dependency_groups` goes in only when your recipe needs a group beyond the base
environment; `agents` is the one that exists today.

## Models, Regions and cost

**Run on GPT-5.6 or later.** Today that means `openai.gpt-5.6-sol`,
`openai.gpt-5.6-terra` or `openai.gpt-5.6-luna`. An earlier generation may appear only as
the "before" half of a migration or a comparison, never as the way to do something.

**Name the model and the Region in the script, in a marked constants block**, and print them
when the script runs. Do not resolve a model from an alias: a reader needs to see which model
produced the output in front of them. Note that constructing the client without `region=`
does not fail — it quietly resolves from the environment and then the local AWS config, so
the recipe would run wherever the reader's laptop happens to point.

**The cost band has to be justified in the README.** Say how many calls the recipe makes,
what the output is capped at, and whether anything is billed per operation rather than per
token. `low` is a handful of small calls. `medium` is dozens of calls, or reasoning-heavy
output, or a per-operation fee, or a lot of injected context. `high` is a sustained loop or
long-context work at scale. The band describes that recipe's own run, not the pattern at
production volume.

**No prices.** Token counts are facts and belong in the recipe; prices go stale, so link the
pricing page instead.

## Writing the README

Readers decide from the first screen whether these models will do their job, so the prose
matters as much as the code.

Write in complete sentences, including inside bullet lists — the pattern that works is a
short bold lead-in followed by a full sentence, not a label. Say a thing once. Show the call,
then explain what could not be inferred from reading it.

**Do not end on the model's answer alone.** End on something measurable: token usage, cache
behaviour, citations, a trace of what the loop did. That is usually the part a reader came
for.

**Never imply a default is wrong.** A default is a decision made by people serving every
workload. When your recipe departs from one, the interesting content is why this workload
wants something else, which teaches the reader to make the same judgement themselves.

**Document a limit only if a reader will meet it,** and pair it with its workaround in the
same sentence. A limit with no response is trivia.

Every recipe needs a section on data handling and security, and a cleanup section. If the
recipe creates an AWS resource, it creates it in a visible numbered step and deletes it in
code — cleanup is executable, not a paragraph telling the reader what to go and delete.

**No placeholders.** No promised diagram, no link to a page that does not exist yet. A recipe
has to read completely as it stands.

## Data, secrets and claims

Sample data is **fabricated, not anonymised**: invented company names, invented people,
`+1-555-01xx` numbers, `example.com` domains, invented identifiers. Never a real IBAN, card
number, tax ID or medical record number, even a lapsed one. Keep it small enough to read by
eye, commit it, make it realistically messy — clean data hides the problems the recipe exists
to solve — and say in the prerequisites that it is synthetic so nobody mistakes it for a
benchmark.

No credentials, account IDs, real ARNs, internal links or customer names, including in
printed output. Recipes take credentials from the AWS credential chain and never read a key
from a file.

**Do not state a platform fact you have not checked.** A live API call beats a documentation
page, and a documentation page beats recollection. If you are unsure, write that it is
unverified rather than rounding up to a confident claim. If you measured something, publish
the number.

## Opening a pull request

Work on a branch of your own rather than your fork's `main`. Anything you push to your
default branch afterwards lands in the same pull request, which is how two unrelated changes
end up sharing one review. A descriptive name is enough; `cookbook/<slug>` reads well.

One change per pull request. A new recipe and a fix to an existing one are two reviews, and
bundling them means the quick one waits for the slow one.

Three checks run automatically on anything touching `cookbooks/`, and they are advisory
rather than blocking:

- **Python lint.** `ruff check` over `cookbooks/`, configured in `pyproject.toml`. The
  formatter is deliberately not run: recipes align their printed request blocks by hand so
  the output reads as a table.
- **Relative links resolve.** Every relative Markdown link and image points at something that
  exists, and a doubled `](target)](target)` suffix is refused.
- **No stray files.** No `.DS_Store`, resource forks, `__pycache__`, `.ruff_cache` or `.venv`.

You can run the first two locally from `cookbooks/`:

```bash
uv sync
uvx ruff check .
```

Everything the checks cannot see — whether the lesson is clear, whether the cost band is
honest, whether a limitation is missing — is what review is for. Expect questions about the
prose as well as the code; that is not a sign something is wrong.
