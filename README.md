ansible-role-git_bare
=========

An Ansible role to manage bare Git repositories with support for variant-aware deployment paths, templated post-receive hooks, and custom deploy scripts. Can be made suitable for automated static site publishing and other Git-based deployments.





## Features

- Creates variant-aware bare Git repositories under a common base directory
- Automatically initializes required directory structure
- Installs per-project post-receive hooks and deploy scripts
- Supports dynamic configuration loading via sourced `.conf` files
- Clean separation of default (example) vs production (project) templates

## Testing Playbook

```yml
---
# git_bare.yml # testing role
# httpd_deploy_sites.yml
#
# Testing:
#   ansible-playbook -i inventory/dev/hosts.yml git_bare.yml -K --check --diff
#
# Limit Run:
#   ansible-playbook -i inventory/dev/hosts.yml git_bare.yml -K -l test
#
# Production Run:
#   ansible-playbook -i inventory/dev/hosts.yml git_bare.yml -K
#
# Requires:
#   cjsteel.fs_state
#   cjsteel.httpd_vhosts
#   cjsteel.git_bare
#   robertdebock.httpd
- name: Test git_bare role
  hosts: localhost
  connection: local
  gather_facts: true 
  vars:
    git_bare_user: "{{ ansible_user | default(ansible_env.USER) }}"    
    git_bare_group: "{{ ansible_user | default(ansible_env.USER) }}"
    git_bare_variant: "dev"
    git_bare_base_dir: "/tmp/deployments"
    git_bare_script_dir: "/tmp/deployments/dev/scripts"
    git_bare_venv_dir: "/tmp/fakevenv"  # Simulated venv
    git_bare_create_conf_file: true
    git_bare_create_post_receive_hook: true

    git_bare_project_conf_template: "{{ role_path }}/templates/example.conf.j2"
    git_bare_deploy_script_template: "{{ role_path }}/templates/example-project-script.sh.j2"
    git_bare_post_receive_hook_template: "{{ role_path }}/templates/example-post-receive.j2"

    git_bare_repositories:
      - git_bare_project_name: "example.org"
        git_bare_repo_name: "mkdocs.git"

  roles:
    - cjsteel.git_bare
```

## Default Directory Structure

Given the following repository declaration:

```yaml
git_bare_repositories:
  - git_bare_variant: "dev"
    git_bare_project_name: "example.org"
    git_bare_repo_name: "mkdocs.git"
```

The role will create:

```
/home/<user>/deployments/dev/example.org/
├── mkdocs.git/
├── releases/
├── logs/
├── public_html/
└── confs/
    └── example.org.conf
```

## Template System

This role uses Jinja2 templates for key scripts and configuration files.

### Default Example Templates

By default, the role uses safe placeholder templates for demonstration and onboarding:

- `example.conf` → Basic project configuration
- `example-script.sh.j2` → Minimal deploy script
- `example-post-receive.j2` → Standard post-receive hook

These are located in:

```
roles/cjsteel.git_bare/templates/
```

### Production Templates

You can override any template using your own versions, typically located in your playbook's private directory:

```yaml
vars:
  git_bare_project_conf_template: "{{ playbook_dir }}/private/templates/git_bare/project.conf.j2"
  git_bare_deploy_script_template: "{{ playbook_dir }}/private/templates/git_bare/project-deploy-mkdocs.sh.j2"
  git_bare_post_receive_hook_template: "{{ playbook_dir }}/private/templates/git_bare/project-post-receive.j2"
```

These templates are then rendered into:

- `confs/{{ project }}.conf`
- `scripts/deploy-{{ project }}-{{ repo }}.sh`
- `hooks/post-receive`

## Usage Example

```yaml
- hosts: web
  roles:
    - role: cjsteel.git_bare
      vars:
        git_bare_user: "deploy"
        git_bare_variant: "dev"
        git_bare_repositories:
          - git_bare_project_name: "example.org"
            git_bare_repo_name: "mkdocs.git"
```

## Required Variables

Ensure these variables are defined or overridden in your playbook or inventory:

- `git_bare_user` (defaults to `ansible_user`)
- `git_bare_base_dir`
- `git_bare_repo_name` (default: `mkdocs.git`)
- `git_bare_repositories` (list of per-project definitions)

## License

MIT or similar. Customize as needed.

## Author

Christopher Steel

``` 

---

Would you like this copied into your actual role folder and auto-populated with `ansible-galaxy` headers or tags?


A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
