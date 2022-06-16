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
