# MCP Ansible

## Setup

```bash
pipenv shell
pipenv install
export ANSIBLE_CONFIG=./ansible.cfg
```

## Playbooks

All playbooks should use the role `bootstrap`. The role is responsible for:
* capture the MCP IP Address
* Store the token variable in [cache](tmp/localhost) for subsequent playbooks
* Set the Products variable for subsequent playbooks

Example variable output:

```json
{
    "products_by_resourceType": {
        "ifd.v6.resourceTypes.L0ClientServiceIntentFacade": "336f19f0-8f44-11ec-9feb-897322cec140",
        "ifd.v6.resourceTypes.L0ServiceIntentFacade": "335884b0-8f44-11ec-9feb-897322cec140",
        "ifd.v6.resourceTypes.L0ServiceIntentMRFacade": "33635a20-8f44-11ec-9feb-897322cec140",
        "ifd.v6.resourceTypes.L1NCPClientServiceIntentFacade": "3385af30-8f44-11ec-9feb-897322cec140",
        "ifd.v6.resourceTypes.L2ServiceIntentFacade": "341ac340-8f44-11ec-9feb-897322cec140"
    }
}
```

This variable can then be used in any playbook as follows:

```yml
    - name: create L2 Facade
      ansible.builtin.import_tasks: tasks/create_resource.yml
      vars:
        body:
          productId: "{{ products_by_resourceType['ifd.v6.resourceTypes.L2ServiceIntentFacade'] }}"
          properties:
            serviceType: EVPL
          .........
```

### Getting started

Reusable tasks are in the tasks directory.

The ansible config uses jsonfile caching. This is handled automatically by relevant tasks. Explore [cache](tmp/localhost) after running a playbook.

To save a custom variable during execution to the cache file, set the `store_name` variable:

```yml
  vars:
    service_name: foo
  tasks:
    - name: ------------------- CREATE L2 Facade -------------------
      ansible.builtin.include_tasks: tasks/create_resource.yml
      vars:
        body: '{{ lookup("template", "templates/EPL.yml.j2") }}'
        store_name: "{{ service_name }}"
```

This will save the result of that call to the cache as:

```json
{
  "store": {
    "foo": {
      "autoClean": false,
      "createdAt": "2022-06-16T17:28:50.722Z",
      "desiredOrchState": "active",
      ......
```

#### print_vars.yml

Print the vars of products in MCP

`ansible-playbook print_vars.yml`
