# Table of contents

* [Get started](quickstart.md)
* [Add an employment](add-employment.md)
* [Read a record](read-a-record.md)
* [Add and update works](works.md)
* [Search the registry](search.md)

## Reference

* ```yaml
  type: builtin:openapi
  props:
    models: true
    downloadLink: true
    grouping: by-operation
  dependencies:
    spec:
      ref:
        kind: openapi
        spec: Orcid-POC
  ```

## Public API reference

* ```yaml
  type: builtin:openapi
  props:
    models: true
    downloadLink: true
    grouping: by-operation
  dependencies:
    spec:
      ref:
        kind: openapi
        spec: Orcid-Public-POC
  ```
