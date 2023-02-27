# DefectDojo Import Scan Report

This GitHub Action uploads a supported scan report generated by your pipeline to a DefectDojo instance. A list of supported reports can be found here - https://documentation.defectdojo.com/integrations/parsers/file/

## About DefectDojo

[DefectDojo](https://github.com/DefectDojo/django-DefectDojo) is a security orchestration and vulnerability management platform. DefectDojo allows you to manage your application security program, maintain product and application information, triage vulnerabilities and push findings to systems like JIRA and Slack. DefectDojo enriches and refines vulnerability data using a number of heuristic algorithms that improve with the more you use the platform.

## Inputs

| Input Name                   | Required |
| ---------------------------- | -------- | 
| defectdojo-url               | True     |
| defectdojo-username          | True     |
| defectdojo-password          | True     |
| defectdojo-product-type      | True     |
| defectdojo-product           | True     |
| defectdojo-environment-type  | True     |
| defectdojo-scan-type         | True     |
| defectdojo-engagement-name   | True     |
| scan-results-file-name       | True     |

## Examples

### Using this action with with semgrep

#### About semgrep

[Semgrep](https://github.com/returntocorp/semgrep) is a fast, open-source, static analysis engine for finding bugs, detecting vulnerabilities in third-party dependencies, and enforcing code standards.

In this example we run a Semgrep SAST scan and then use this action to import it into a defectdojo instance.

```
name: semgrep sast scan and import to defectdojo
on:
  push
Jobs:
  semgrep-sast-scan:
    name: semgrep sast scan
    runs-on: ubuntu-latest
    container:
      image: returntocorp/semgrep
    if: (github.actor != 'dependabot[bot]')
    steps:
      - name: checkout
        id: checkout
        uses: actions/checkout@v3
      - name: semgrep scan
        id: semgrep-scan
        run: |
          mkdir -p semgrep/results
          semgrep --config auto --error --json --output=semgrep/results/semgrep.json
      - name: upload semgrep results
        id: upload-semgrep-results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: semgrep-results
          path: semgrep/results/

  import-semgrep-report
    name: import semgrep report
    runs-on: ubuntu-latest
    steps:
      - name: download scan results artifact
        id: download-scan-results-artifact
        uses: actions/download-artifact@v3
        with:
          name: semgrep-results
      - name: import scan results into defectdojo
        id: import-scan-results-into-defectdojo
        uses: portswigger-cloud/defectdojo-github-action@main
        with:
          defectdojo-url: defectdojo.example.com
          defectdojo-username: ${{ secrets.defectdojo-username }}
          defectdojo-password: ${{ secrets.defectdojo-password }}
          defectdojo-product-type: example-product-type
          defectdojo-product: example-product
          defectdojo-environment-type: Production
          defectdojo-scan-type: Semgrep JSON Report
          defectdojo-engagement-name: Github Actions Initiated SAST Scan
          scan-results-file-name: semgrep.json
```
