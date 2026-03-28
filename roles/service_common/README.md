# service_common
Generic service installation and lifecycle management.



## Purpose
Provides a reusable mechanism for managing system services without creating dedicated roles per application.



## Variables
```
services:
- name: nginx
  package: nginx

service_enable: true
service_state: started
```



## Example
```yaml
- include_role:
    name: service_common
```
Services are defined via inventory or higher-level roles.