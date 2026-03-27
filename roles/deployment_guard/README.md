# deployment_guard
Prevents unsafe infrastructure execution.



## Responsibilities
- Validates environment selection
- Blocks production execution by default
- Ensures explicit operator intent



## Variables
```
env:
    dev | staging | prod

allow_prod_changes:
    false (default)
```


## Example Usage
```yaml
- hosts: all
  roles:
    - deployment_guard
```

The guard should be executed before any infrastructure change, hence statically imported into the playbook.