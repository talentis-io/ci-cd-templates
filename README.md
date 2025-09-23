# ci-cd-templates

## Reusable workflows

GitHub only allows cross-repo reusable workflows referenced from the root of `.github/workflows/`. Nested folders are not supported.

All reusable workflows are flattened under `.github/workflows/`:

```
.github/
  workflows/
    react-run.yml
    sonar-run.yml
    webhook-run.yml
```

### Usage from a consumer repository

```yaml
jobs:
  ci:
    uses: my-org/ci-cd-templates/.github/workflows/react-run.yml@v1
    with:
      project-key: "my-org_${{ github.event.repository.name }}"
      project-name: "${{ github.event.repository.name }}"
    secrets: inherit
```

Inside `react-run.yml`, Sonar and webhook are referenced locally:

```yaml
jobs:
  sonar-analysis:
    needs: build-and-test
    uses: ./.github/workflows/sonar-run.yml
    with:
      project-key: ${{ inputs.project-key }}
      project-name: ${{ inputs.project-name }}
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

  notify-completion:
    needs: [build-and-test, sonar-analysis]
    uses: ./.github/workflows/webhook-run.yml
    with:
      webhook-type: "sonar-completed"
      project-key: ${{ inputs.project-key }}
    secrets:
      WEBHOOK_URL: ${{ secrets.WEBHOOK_URL }}
      WEBHOOK_SECRET: ${{ secrets.WEBHOOK_SECRET }}
```

### Notes

- Cross-repo `uses:` must be rooted in `.github/workflows/`.
- For organization, use filename prefixes (e.g., `react-`, `node-`, `python-`).