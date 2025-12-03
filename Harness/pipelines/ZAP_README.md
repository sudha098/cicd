
```markdown
# OWASP ZAP Scans in Harness

This guide explains how to run **OWASP ZAP vulnerability scans** in Harness using:

- **CI Run steps** (free-tier compatible)  
- **STO built-in steps** (paid tier)  
- **Zest authentication scripts** for logging into applications  

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Directory Structure](#directory-structure)  
3. [Using ZAP in Free-Tier CI](#using-zap-in-free-tier-ci)  
4. [Using STO Built-in ZAP Step](#using-sto-built-in-zap-step)  
5. [Zest Scripts](#zest-scripts)  
6. [ZAP Context File](#zap-context-file)  
7. [Best Practices](#best-practices)

---

## Prerequisites

- Harness account (free tier or with STO)  
- Docker installed for local testing (optional)  
- OWASP ZAP installed locally for creating `.context` and `.zest` files  
- Target web application URL (Dev or Staging)

---

## Directory Structure

Place your files in **shared paths** for Harness:

```

/shared/customer_artifacts/
├── context/
│   └── myapp.context            # ZAP context file
├── scripts/
│   ├── authentication/
│   │   └── my_login.zest        # Zest login script
│   └── session/
│       └── my_session.zest      # Optional session handling script
└── scan_results/                # Reports will be saved here

````

---

## Using ZAP in Free-Tier CI

Harness free-tier users can run ZAP scans using a **Run step** with the official Docker image.

```yaml
pipeline:
  name: ZAP Dev CI Pipeline
  identifier: zap_dev_ci_pipeline
  projectIdentifier: my_project
  orgIdentifier: my_org
  stages:
    - stage:
        name: Dev ZAP Scan
        identifier: dev_zap_scan
        type: CI
        spec:
          cloneCodebase: false
          execution:
            steps:
              - step:
                  name: Run ZAP Scan
                  identifier: run_zap_scan
                  type: Run
                  spec:
                    image: owasp/zap2docker-stable
                    shell: Bash
                    command: |
                      echo "Starting ZAP scan against $DEV_APP_URL"

                      mkdir -p /shared/scan_results

                      zap-baseline.py \
                        -t $DEV_APP_URL \
                        -c /shared/customer_artifacts/context/myapp.context \
                        -J /shared/scan_results/report.json \
                        -r /shared/scan_results/report.html \
                        -z "-configfile /shared/customer_artifacts/scripts/authentication/my_login.zest" \
                        -daemon

                      echo "ZAP scan completed. Reports saved in /shared/scan_results"
                    envVariables:
                      - name: DEV_APP_URL
                        type: String
                        value: "<+input>"
                    outputVariables:
                      - name: scan_report_path
                        value: "/shared/scan_results/report.html"
          sharedPaths:
            - /shared/customer_artifacts/context
            - /shared/customer_artifacts/scripts/authentication
            - /shared/customer_artifacts/scripts/session
            - /shared/scan_results
````

**Notes:**

* `zap-baseline.py` is the official baseline scanner.
* `-c` points to the ZAP context file.
* `-z "-configfile …"` points to the Zest authentication script.
* Reports are saved in `/shared/scan_results`.
* `scan_report_path` is an output variable (must use only alphanumeric + `_`).

---

## Using STO Built-in ZAP Step

For paid users with STO:

```yaml
- step:
    name: ZAP Dev Scan
    identifier: zap_dev_scan
    type: ZAP
    spec:
      scanMode: active
      scanType: standard
      target:
        type: Instance
        domain: "<+stage.variables.DEV_APP_URL>"
        path: "/"
        protocol: HTTPS
        port: 443
      contextName: "myapp-context"
      scriptBasedAuth:
        enabled: true
        authScriptPath: "/shared/customer_artifacts/scripts/authentication/my_login.zest"
        sessionScriptPath: "/shared/customer_artifacts/scripts/session/my_session.zest"
      logLevel: INFO
      failOnSeverity: HIGH
```

* Automatically fails if HIGH severity vulnerabilities are detected.
* Supports `.zest` scripts natively for authentication.

---

## Zest Scripts

* `.zest` scripts automate **login and session flows**.
* Record using **OWASP ZAP → Tools → Zest → Recorder**.
* Place scripts in Harness shared paths:

```
/shared/customer_artifacts/scripts/authentication/my_login.zest
/shared/customer_artifacts/scripts/session/my_session.zest
```

---

## ZAP Context File

* `.context` defines:

  * Included/excluded URLs
  * Authentication methods
  * Session handling

* Export from ZAP locally and place in:

```
/shared/customer_artifacts/context/myapp.context
```

---

## Best Practices

1. **Test locally first**: Verify `.context` and `.zest` scripts in ZAP before CI.
2. **Shared paths**: Always mount `/shared/customer_artifacts` in the pipeline.
3. **Output variable names**: Use only alphanumeric + `_`.
4. **Free-tier CI**: Can run scans without STO; STO is optional.
5. **Authenticated scan**: Requires context + Zest script. Without them, only public pages can be scanned.

---

## References

* [OWASP ZAP Official Docker](https://github.com/zaproxy/zaproxy/wiki/Docker)
* [Zest Scripts in ZAP](https://owasp.org/www-project-zap/zest/)
* [Harness CI Documentation](https://docs.harness.io)
* [Harness STO Documentation](https://docs.harness.io/category/security-testing)

