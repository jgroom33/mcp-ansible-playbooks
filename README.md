# MCP Ansible

## Setup

```bash
pipenv shell
pipenv install
```

## Example execution

```mermaid
sequenceDiagram
    Note over App,/tron/api/v1/tokens: ///// Get Token ////// POST
    App->>+/tron/api/v1/tokens: username:admin password:adminpw
    /tron/api/v1/tokens-->>-App: token
    Note over App,market: ///// Create Facade ////// POST --> /bpocore/market/api/v1/resources
    App->>+market: productId: {{ FacadeProduct }} and  {{ other data }}
    market-->>-App: facade id
    Note over App,market: ///// Get Facade ////// GET --> /bpocore/market/api/v1/resources/{{ facade id }}
    loop Repeat until orchState=active
        App->>+market: 
        market-->>-App: orchState
    end
    Note over App,market: ///// Get Facade Relationship ////// GET --> /bpocore/market/api/v1/relationships?q=sourceId:{{ facade id }}
    App->>market: 
    market->>App: items[0].targetId
    Note over App,market: ///// Calculate Routes ////// POST --> /bpocore/market/api/v1/resources/{{ targetId }}/operations
    App->>market: interface: requestRoutes
    market->>App: operation id
    Note over App,market: ///// Get Routes ////// GET --> /bpocore/market/api/v1/resources/{{ targetId }}/operations/{{ operation id }}
    loop Repeat until state=successful
        App->>+market: 
        market-->>-App: state, outputs.routes[]
    end
    Note over App,market: ///// Create FRE ////// POST --> /bpocore/market/api/v1/resources/{{ targetId }}/operations
    App->>market: interface: commitRoute, inputs.routes: [ {{ routes from calculation }} ]
    market->>App: fre id
```


## Playbooks

All playbooks rely on the base playbook, `bootstrap.yml`, which is included in the `tasks` directory. The bootstrap playbook is responsible for:
* capture the MCP IP Address
* Set the token variable for subsequent playbooks
* Set the Products variable for subsequent playbooks

Example variable utput:

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

Notice the helper task `tasks/create_resource.yml`

### Getting started

There are example playbooks for executing API calls against MCP:

* print_products.yml
  * Print the list of products in MCP
* print_token.yml
  * Print the token used to authenticate against MCP

`ansible-playbook print_vars.yml --extra-vars "@vars.yml"`
