### Domain vs regulation


```mermaid
flowchart BT
    classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none, opacity:0
    classDef bounded_context stroke-dasharray:5 5

    subgraph Domain
    subgraph p0
    DV1[User expectations<br>v1]
    DV2[User expectations<br>v2]
    DV3[User expectations<br>v3]
    end
    end

    subgraph Regulation
    subgraph p1
    RV2012[Domain <br> Regulation<br> 2022]
    RV2013[Domain <br> Regulation<br> 2023]
    ORV2015[Other <br> Regulation<br> 2025]
    RV2016[Domain <br> Regulation<br> 2026]
    end
    end

    RV2012 ---> DV1
    RV2013 ---> DV2
    ORV2015 ---> DV2
    RV2016 ---> DV3

    subgraph Ontologies
    subgraph p2
    v1.0
    v1.2
    v1.3
    v1.4
    end
    end

    v1.0 --->|models| RV2012
    v1.2 --->|models| RV2013
    v1.3 --->|models| ORV2015
    v1.4 --->|models| RV2016

    class p0,p1,p2 subgraph_padding
```

### Data semantics

```mermaid
flowchart
    classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none
    classDef bounded_context stroke-dasharray:5 5

    subgraph Personas
    ULegal
    USemantics
    UArchitect
    end
    subgraph Artifacts
    Ontologies
    Schemas
    APIs
    end

    ULegal --> Ontologies["Ontologies (RDF)"]
    USemantics --> Ontologies
    UArchitect --> Schemas["Data models (JSON Schema)"]
    ULegal --> Schemas
    UArchitect --> APIs["Access layer (OAS)"] --> Schemas

```


### Communicating Contexts

Connector pattern

```mermaid
flowchart TB
    %%title Communicating contexts
    classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none
    classDef bounded_context stroke-dasharray:5 5

    subgraph Domain
    subgraph p1 [ ]
        subgraph left[ ]
        direction TB
        SD1
        Onto[Ontologies]
        SD2
        end

        subgraph SD1 [Subdomain 1]
        end

        subgraph SD2 [Subdomain 2]
        end

        subgraph Model
        subgraph p4 [ ]
            subgraph C1 [Context 1]
            subgraph p2 [ ]
                direction TB
                DM1[Data models]
                OP1[Operations]
                UL1[Ubiquitous Language]
            end
            end

            subgraph C2 [Context 2]
            subgraph p3 [ ]
                direction TB
                DM2[Data models]
                OP2[Operations]
                UL2[Ubiquitous Language]
            end
            end
        end
        end

    end
    end

    Onto -.->|supports| UL1 & UL2
    SD1 -.- C1
    SD2 -.- C2
    %% layout
    class C1,C2 bounded_context
    class p1,p2,p3,left,p4 subgraph_padding

```

### Connector pattern

```mermaid
flowchart LR
    classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none

    ContextA((Context A)) --o |connector| ContextB((Context B))
```

### Data modeling perspective

```mermaid
%% title Data modeling perspective
flowchart LR
    classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none

    subgraph Semantic [  Domain Data Models ]
        subgraph p0 [  ]
            RDF
            Data
        end
    end

    subgraph Data [ Data Models ]
        subgraph p2 [ <br> ]
            direction TB
            JSON --- XML
            CSV --- TTL[RDF]
            linkStyle 0,1 stroke:none
        end
    end


    subgraph System [ Systems ]
        direction LR
        subgraph p1 [ ]
            direction TB
            API & A[API] & B[API]

        end
        DB
    end
    Semantic

    System -->|produce| Data
    Data -.->|embed| RDF --> |project| Data
    DB[(Datastore<br>Physical Model)]
    class p0,p1,p2 subgraph_padding

```

### DDD Experts Architects Public service

```mermaid
flowchart TD
    classDef default stroke:none,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none

    U[fa:fa-user Citizen] -.-> Sol


    subgraph BC ["Bounded contexts (API Design) &nbsp;&nbsp; <br>"]
    subgraph Padder2 [ ]
        DataModel[Data Models] ---
        Operations
        Inputs ---
        Outputs
        %% Using linkStyle and --- links to display items as a square
        linkStyle 0,1 stroke:none
    end
    end

    subgraph Sol["Design (Solution Space) &nbsp; <br>one for each project"]

        subgraph Padder3 [ ]
        end
        BC
        subgraph Padder4 [ ]
        direction RL
        T[fa:fa-user IT Architect]
        UL(Ubiquitous Language)
        DM(Domain Model)
        end
        linkStyle 0,2 stroke:none
    end
    subgraph Domain["Domain (Problem Space) &nbsp; <br>"]
        subgraph Padder1 [ ]
        S[fa:fa-user Semantic Expert] -.->|models| Ontologies
        %% linkStyle 3 stroke:none
        Sol
        end
    end

        subgraph Government
      Domain
      subgraph P0 [ ]
        R[fa:fa-user Regulator] -->|regulates| Domain
        D[fa:fa-user Domain Expert] --- R
        linkStyle 5 stroke:none
      end
    end

    T -.-> |agrees| UL
    %% T -.-o |relies on| D
    T -.-> |defines| BC
    UL -.-o Ontologies
    S -.-o |relies on| D
    %% Padders avoid overflows in subgraphs.
    class P0,Padder1,Padder2,Padder3,Padder4 subgraph_padding

```


### Sample ontology graph

```mermaid
graph LR

 classDef default stroke:#3344d0,color:#fff,clusterBkg:none,fill:#3344d0

 Dog --> |kind of| Mammal --> |kind of| Animal
 Fish --> |kind of| Animal
 Mammal -.-> |not a| Fish
 petName --> |domain| Dog
 Dog -.-> |has| petName


```

### DDD Experts Architects

```mermaid
flowchart TD
    classDef default stroke:#3344d0,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none
    subgraph BC ["Bounded contexts (API Design) &nbsp;&nbsp; <br>"]
    subgraph Padder2 [ ]
        DataModel[Data Models] ---
        Operations
        Inputs ---
        Outputs
        %% Using linkStyle and --- links to display items as a square
        linkStyle 0 stroke:none
        linkStyle 1 stroke:none
    end
    end

    subgraph Sol["Design (Solution Space) &nbsp; <br>one for each project"]

        subgraph Padder3 [ ]
        end
        BC
        subgraph Padder4 [ ]
        direction RL
        T[fa:fa-user IT Architect]
        UL(Ubiquitous Language)
        DM(Domain Model)
        end
        %% linkStyle 2,3 stroke:none
    end
    subgraph Domain["Domain (Problem Space) &nbsp; <br>"]
        subgraph Padder1 [ ]
        D[fa:fa-user Domain Expert]
        S[fa:fa-user Semantic Expert] -.->|models| Ontologies
        %% linkStyle 3 stroke:none
        Sol
        end
    end

    T -.-> |agrees| UL
    %% T -.-o |relies on| D
    T -.-> |defines| BC
    UL -.-o Ontologies
    S -.-o |relies on| D
    %% Padders avoid overflows in subgraphs.
    class Padder1,Padder2,Padder3,Padder4 subgraph_padding

```


### DDD Public Sector


```mermaid
%% DDD in Public Sector
graph LR
    classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none, opacity:0
    classDef bounded_context stroke-dasharray:5 5

    subgraph Healthcare [" Healthcare  (domain) &nbsp; <br> "]
        subgraph Padder0 [ ]
            HS[Service]
        end
    end

    subgraph Tax ["Tax (domain) &nbsp; <br>"]
        subgraph Padder1 [ ]
            TS[Service]
        end
    end

    subgraph Population  ["Population (domain) &nbsp; <br>"]
        subgraph Padder2 [ ]
            PS[Service]
        end
    end

    subgraph Government["Government (domain) &nbsp; <br>"]
        subgraph Padder3 [ ]
            Healthcare
            Tax
            Population
        end
    end

    HS -->  PS
    TS -->  PS
    U[fa:fa-user Citizen] --> |uses| HS & TS
    %% Padders avoid overflows in subgraphs.
    class Padder0,Padder1,Padder2,Padder3 subgraph_padding
    class Healthcare,Tax,Population,Government bounded_context

```



## DDD Flowchart

```mermaid

graph TD
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef default fill:#3344d0,stroke:#3344d0,color:#fff,clusterBkg:none
    classDef subgraph_padding fill:none, stroke:none

    subgraph BC ["Bounded contexts (API Design) &nbsp;"]
    subgraph Padder2 [ ]
        DataModel[Data Models] ---
        Operations
        Inputs ---
        Outputs
        %% Using linkStyle and --- links to display items as a square
        linkStyle 0 stroke:none
        linkStyle 1 stroke:none
    end
    end

    subgraph Sol["Design (Solution Space) &nbsp;"]
        subgraph Padder3 [ ]
        UL(Ubiquitous Language) ---
        DM(Domain Model)
        BC
        linkStyle 2 stroke:none
        T[fa:fa-user IT Architect]
        end
    end
    subgraph Domain["Domain (Problem Space) &nbsp;"]
        D[fa:fa-user Domain Expert]
        %% --- Ontologies
        %% linkStyle 3 stroke:none
        subgraph Padder1 [ ]
        Sol
        end
    end

    %% Padders avoid overflows in subgraphs.
    class Padder1,Padder2,Padder3 subgraph_padding

```

```mermaid

graph TD
%%{init: {"theme": "base", "themeVariables": { "primaryColor": "#3344d0", "primaryTextColor": "#27364e", "clusterBkg": "none", "tertiaryBorderColor": "#3344d0", "nodeTextColor": "white", "fontFamily": "Titillium"}}}%%


    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef default fill:#3344d0,stroke:#3344d0,color:#fff,clusterBkg:none
    classDef subgraph_padding fill:none, stroke:none

    subgraph BC ["Bounded contexts (API Design) &nbsp;"]
    subgraph Padder2 [ ]
        DataModel[Data Models] ---
        Operations
        Inputs ---
        Outputs
        %% Using linkStyle and --- links to display items as a square
        linkStyle 0 stroke:none
        linkStyle 1 stroke:none
    end
    end

    subgraph Sol["Design (Solution Space) &nbsp;"]
        subgraph Padder3 [ ]
        UL(Ubiquitous Language) ---
        DM(Domain Model)
        BC
        linkStyle 2 stroke:none
        end
    end
    subgraph Domain["Domain (Problem Space) &nbsp;"]
        Ontologies
        subgraph Padder1 [ ]
        Sol
        end
    end

    %% Padders avoid overflows in subgraphs.
    class Padder1,Padder2,Padder3 subgraph_padding




```

## DDD Context Regulation


```mermaid
C4Context
  title Public Sector Domain Model

  Boundary(gov, "Public Sector (e.g. Healthcare, Registry)", "Domain") {
    Person(User, "Citizen")
    Boundary(regu, "Regulation", "Domain Model & Bounded Context"){
      Person(Designer, "Designer")
      System(impl, "Digital Public Service")
      Boundary(S1, "Context A") {
        System(s1, "foo")
      }
      Boundary(S2, "Context B") {
        System_Ext(foo, "BAR")
        }

    }
  }
  Rel(User, impl, "uses")
  Rel(Designer, impl, "designs")

```
