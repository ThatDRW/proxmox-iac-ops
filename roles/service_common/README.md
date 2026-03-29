# service_common
Generic service installation and lifecycle management.



## Purpose
Provides a reusable mechanism for managing system services without creating dedicated roles per application.



## Variables
```
service_common_services:
- name: nginx
  package: nginx

service_common_enable: true
service_common_state: started
```



## Example
```yaml
- include_role:
    name: service_common
```
Services are defined via inventory or higher-level roles.