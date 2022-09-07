
## DDD Flowchart Regulation

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

