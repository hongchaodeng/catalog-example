# Catalog Structure Format

All packages are put under `/catalog/` dir (this is configurable). The directory structure follows a predefined format:

```bash
/catalog/ # a catalog consists of multiple packages 
|-- <package> # a package consists of multiple versions
    |-- package.yaml
    |-- 1.0
        |-- modules.yaml
        |-- caps.yaml
        |-- conditions/
            |-- check-crd.yaml
        |-- hooks/
            |-- pre-install.yaml
    |-- 2.0 # must be semver
|-- <package>
```

- Note that each package directory name will be the package name. Each version directory name will be the version name.

- `package.yaml`: This is package-level metadata.

  ```yaml
  description: |
    More details about this package.
  maintainers:
    - "@github_id"
    - "email@example.com"
  license: "Apache License Version 2.0"
  url: "https://homepage.io"

  tags: # This will be indexed and serve in package searching.
    - helm
    - autoscaling
    - light-weight
    - cloud-native
  ```

- `definitions.yaml`: This defines the definitions that a pacakge (in specific version) provides. It tells the name, the type (Application/Component/Trait), and the json schema exposed for user input.

  ```yaml
  - name: WebService
    type: Component
    schema:
      path: ../schema/webservice.schema.json
  - name: Routing
    type: Trait
    schema:
      path: ../schema/routing.schema.json
  - name: MyApp
    type: Application
    schema:
      path: ../schema/myapp.schema.json
  ```

- `modules.yaml`: It defines the modules that contain the actual resources, e.g. Helm Charts or Terraform modules. Note that the design extensible to more module/package formats (e.g. CloudFormation templates).

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

- `conditions/`: It defines conditional checks before deploying this package. For example, check if a CRD with specific version exist, if not then the deployment should fail.

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
