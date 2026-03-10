# sigma-rules-enriched

Sigma detection rules from [SigmaHQ](https://github.com/SigmaHQ/sigma) and [LOLRMM](https://github.com/magicsword-io/LOLRMM) enriched with LLM-generated investigation guides.

Each rule has a `note` field containing a structured investigation guide with:
- **Technical Context** -- what the rule detects and relevant MITRE ATT&CK mappings
- **Investigation Steps** -- actionable, vendor-agnostic response guidance
- **Prioritization** -- severity reasoning for enterprise environments
- **Blind Spots and Assumptions** -- detection gaps and required log sources

## How it works

A [GitHub Actions workflow](.github/workflows/enrich-rules.yml) runs daily:

1. Clones the latest SigmaHQ and LOLRMM repos
2. Runs [sigma-llm-doc](https://github.com/InfoSecJay/sigma-llm-doc) to generate investigation guides
3. Only processes new or changed rules (content hashing + cache)
4. **Removes orphaned rules** — enriched files whose upstream source no longer exists are deleted
5. Commits the enriched output back to this repo (using `git add -A` to capture deletions)

## Structure

```
sigma-rules-enriched/
  sigmahq/                    # Enriched SigmaHQ rules (mirrors rules/ directory structure)
  sigmahq-emerging-threats/   # Enriched SigmaHQ Emerging Threats rules
  lolrmm/                     # Enriched LOLRMM Sigma rules
```

## Setup

To run this yourself, fork this repo and add your `OPENAI_API_KEY` as a repository secret:

1. Go to **Settings > Secrets and variables > Actions**
2. Add a new secret named `OPENAI_API_KEY` with your OpenAI API key
3. The workflow will run daily at 6:00 AM UTC, or trigger it manually from the Actions tab

## Downstream usage

These enriched rules feed into a detection-as-code pipeline:

```
sigma-rules-enriched (this repo)
        |
        v
   Sigma-to-TOML converter
        |
        v
   NDJSON via Elastic detection-rules tool
        |
        v
   Elastic SIEM
```
