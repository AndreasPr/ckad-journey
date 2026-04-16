## Helm Templating with `values.yaml` and `templates/`

Helm allows you to **parameterize Kubernetes manifests** using templates and a centralized configuration file (`values.yaml`). This makes deployments **flexible, reusable, and environment-specific**.

---

## Example Breakdown

### Template File: `templates/secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-admin-password
data:
  key: {{ .Values.passwordEncoded }}
```

### Values File: `values.yaml`

```yaml
passwordEncoded: venfcdsikn534csdkjn
```

---

## How It Works

* `{{ .Values.passwordEncoded }}` is a **template expression**
* Helm replaces it with the value from `values.yaml` during rendering
* Final rendered YAML (before applying to cluster):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-admin-password
data:
  key: venfcdsikn534csdkjn
```

---

## Key Concepts

### 1. `.Values`

* Refers to the `values.yaml` file
* Central place to configure your application
* Supports nested structures:

```yaml
database:
  user: admin
  password: secret
```

Access in template:

```yaml
{{ .Values.database.user }}
```

---

### 2. Templates + Values = Helm Chart

A **Helm Chart** consists of:

* `templates/` → Kubernetes manifests (dynamic)
* `values.yaml` → configuration (static defaults)

This separation enables:

* Reusability
* Clean configuration management
* Environment overrides

---

## Why This Is Powerful

Instead of editing multiple YAML files:

* Change **one value** in `values.yaml`
* All related templates update automatically

Example:

* Change password
* Change replica count
* Change image version

---

## Working with Helm Charts

### Search Charts

From public registry like Artifact Hub:

```bash
helm search hub wordpress
```

---

### List Repositories

```bash
helm repo list
```

---

### Install a Chart

```bash
helm install [release-name] [chart-name]
```

Example:

```bash
helm install my-wordpress bitnami/wordpress
```

---

## Managing Releases

### List Installed Releases

```bash
helm list
```

---

### Uninstall a Release

```bash
helm uninstall {release-name}
```

Removes:

* All Kubernetes resources
* Release metadata

---

## Download Chart Locally (Without Installing)

```bash
helm pull --untar bitnami/wordpress
```

This:

* Downloads the chart
* Extracts it locally

---

### Inspect Chart Contents

```bash
ls wordpress
```

Typical output:

```
Chart.yaml
values.yaml
templates/
```

---

## Customize and Install Locally

After modifying `values.yaml`:

```bash
helm install release-4 ./wordpress
```

This installs:

* Your customized version of the chart

---

## Overriding Values

### Option 1: CLI

```bash
helm install myapp ./chart --set passwordEncoded=abc123
```

---

### Option 2: Custom Values File

```bash
helm install myapp ./chart -f custom-values.yaml
```

---

## Important Notes (Best Practices)

### 1. Secrets Must Be Base64 Encoded

Kubernetes Secrets require base64 values.

Example:

```bash
echo -n "mypassword" | base64
```

---

### 2. Avoid Hardcoding Sensitive Data

Use:

* External secret managers
* Environment-specific values files

---

### 3. Use Default Values

Always provide defaults in `values.yaml` to avoid template failures.

---

## Common Useful Questions

### Q1: What is `.Values` in Helm?

It is a reference to configuration defined in `values.yaml`, used to inject dynamic data into templates.

---

### Q2: How does Helm render templates?

Helm processes Go templates and replaces placeholders with values from:

* `values.yaml`
* CLI overrides
* environment-specific files

---

### Q3: What is the benefit of separating templates and values?

* Cleaner configuration
* Reusability
* Easier environment management

---

### Q4: How do you customize a Helm deployment?

* Modify `values.yaml`
* Use `--set`
* Provide custom values file (`-f`)

---

## Summary

* Helm uses **templates + values** to generate Kubernetes manifests
* `values.yaml` controls configuration centrally
* Templates dynamically inject those values
* Charts can be reused, customized, and versioned easily


# Scenarios
## Scenario 1
Which command is used to search for a `wordpress` helm chart package from the `Artifact Hub`?

## Solution
Run `helm search hub chart-name` command to search specific charts on `Artifact Hub`.

Note: Replace the `chart-name` with the original name.

## Scenario 2
Add a `bitnami` helm chart repository in the `controlplane` node.

name - `bitnami`

chart repo name - `https://charts.bitnami.com/bitnami`

## Solution
Run the command: `helm repo add bitnami https://charts.bitnami.com/bitnami`

## Scenario 3
Which command is used to search for the `joomla` package from the added repository?

## Solution
Run `helm search repo joomla` command to search specific package from all the added repository.

## Scenario 4
How many `helm` repositories are added in the `controlplane` node?

## Solution
Run the command: `helm repo list`


## Scenario 5
Install `drupal` helm chart from the `bitnami` repository.

Release name should be `bravo`.

Chart name should be `bitnami/drupal`.

Note: Ignore the state of the application now.

## Solution
Run the command: `helm install bravo bitnami/drupal` and after that run the `helm list` command to verify it's installation.


## Scenario 6
Uninstall the `drupal` helm package which we installed earlier.

## Solution
Run the command: `helm uninstall bravo`

## Scenario 7
Download the `bitnami apache` package under the `/root` directory with version `10.1.1`

Note: Do not install the package. Just download it.

## Solution
Run the command: `helm pull --untar  bitnami/apache --version 10.1.1`

## Scenario 8
Install the `apache` Helm chart from the downloaded package.

Release name: `mywebapp`

Note: Do make changes accordingly so that 2 replicas of the webserver are running and the `http` is exposed on nodeport `30080`.

Make sure that the pods are in the ready & running state.

## Solution
Once you have modified the `values.yaml` file , run the below command to install the `apache` package on the `controlplane` node:

```bash
root@controlplane:~# helm install mywebapp ./apache
```

After installation, run the below command to list the `mywebapp` release:
```bash
root@controlplane:~# helm list     
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART            APP VERSION
mywebapp        default         1               2024-12-03 14:11:17.237760871 +0000 UTC deployed        apache-11.2.22   2.4.62   
```








