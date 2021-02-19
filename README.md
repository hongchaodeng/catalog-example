# Catalog Structure Format

All packages are put under `/catalog/` dir (this is configurable). The directory structure follows a predefined format:

```bash
/catalog/ # a catalog consists of multiple packages 
|-- <package>
    |-- package.yaml
    |-- v1.0 # a package consists of multiple versions
        |-- modules.yaml
        |-- caps.json
        |-- conditions/
            |-- check-crd.yaml
        |-- hooks/
            |-- pre-install.yaml
    |-- v2.0
|-- <package>
```

- Note that each package directory name will be the package name. Each version directory name will be the version name.

- `package.yaml`: This is package-level metadata across all versions.

  ```yaml
  description: |
    More details about this package.
  maintainers:
    - "@github_id"
    - "email@example.com"
  license: "Apache License Version 2.0"
  url: "https://homepage.io"
  labels:
    category: test
  ```

- `caps.json`: This describes the capabilities that a pacakge (in specific version) provides. It tells the name, the type (Workload/Trait), and the json schema of inputs.

  ```json
  [
    {
      "name": "WebService",
      "type": "Workload",
      "jsonschema": "..."
    }
  ]
  ```

- `conditions/`: defining conditional checks before deploying this package. For example, check if a CRD with specific version exist, if not then the deployment should fail.

  ```yaml
  # check-crd.yaml
  conditions:
  - target:
      apiVersion: apiextensions.k8s.io/v1
      kind: CustomResourceDefinition
      name: test.example.com
      fieldPath: spec.versions[0].name
    op: eq
    value: "v1"
  ```

- `hooks/`:lifecycle hooks on deploying this package. These are the k8s jobs, and consists of pre-install, post-install, pre-uninstall, post-uninstall.

  ```yaml
  # pre-install.yaml
  pre-install:
  - job:
      apiVersion: batch/v1
      kind: Job
      spec:
        template:
          spec:
            containers:
            - name: pre-install-job
              image: "pre-install:v1"
              command: ["/bin/pre-install"]
  ```

- `modules.yaml`: defining the modules that contain the actual resources, e.g. Helm Charts or Terraform modules. Note that we choose to integrate with existing community solutions instead of inventing our own format. In this way we can adopt the reservoir of community efforts and make the design extensible to more in-house formats as we have observed.

  ```yaml
  modules:
    - helm: # Helm module to install/upgrade
        repo: https://wonderflow.info/kubewatch/archives/
        name: kubewatch
        version: 0.1.0
    - native: # Native k8s yaml to apply
        path: definitions/trait.yaml # relative path to the modules.yaml
        url: https://git.io/vPieo
  ```