---
title: "Domain Driven Design and Ontologies"
---

# Abstract

Ontologies and Controlled Vocabularies are meant to standardize
semantics in specific domains, for providing a knowledge base for better
understanding those domains. The interactions between vocabularies and
the creation of digital services is not trivial though, especially in
the context of digital public services where the "domain" is faceted
by different perspectives: citizen expectations, regulation and
organizational procedures. After a brief presentation of domain driven
design, this document analyzes the relations between the ontologies'
modeling process and the usability of those models for the creation of
APIs.

# Introduction


## Preliminary notes

While this paper relies on domain driven design concepts, there can be slight differences between the original 2003 definition and how they are interpreted in time by the various teams or authors. For example, the increasing interest in event-driven patterns suggested the introduction of  [domain events](https://www.oreilly.com/library/view/reactive-microsystems/9781491994368/ch04.html)  in contrast to domain objects; or in some contexts the Ubiquitous Language is considered valid throughout the entire domain.


## Domain Driven Design

Domain-driven design is a software design methodology described by [Eric
Evans in 2003](https://www.amazon.it/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) focusing on modeling software based on the input from
Domain's experts. This is roughly based on the dichotomy
problem/solution.

The Problem space is represented by:

- a Domain (e.g. banking, healthcare) is a specific field of
    application. A Domain can be recursively divided in further
    sub-domains associated to specific responsibilities until the
    problem is small enough to be modeled (the identification of the
    solution space);

The Solution space is characterized by:

- a Model (Domain Model) is an abstraction of the Domain that is
    useful in pursuing a specific business inside the Domain; it is thus
    related to the solution. Likewise, Models can be general to the
    Domain, or confined in specific sub-domains;

- a [Bounded Context]
    includes all the logical boundaries (inputs, outputs, events,
    requirements, processes, [aggregates] and data models) associated with a Domain or
    a Sub-Domain. This avoids that specific technical choices impact on  the whole project.
    When designing an API, it is possible to use
    an API Canvas to model the entities associated
    to a context (e.g. inputs, outputs, users, ...);

[Bounded Context]: https://www.martinfowler.com/bliki/BoundedContext.html
[aggregates]: https://martinfowler.com/bliki/DDD_Aggregate.html

- an [Ubiquitous Language],
    that is a formalized language based on the Model that allows
    technical experts (e.g. IT architects, software engineers, ...) and
    domain experts (e.g. experts in healthcare, procurement, finance,
    ...) to communicate without ambiguities.

[Ubiquitous Language]: https://martinfowler.com/bliki/UbiquitousLanguage.html

```{mermaid}
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
    %% Using linkStyle and --- links to display items as a
    %% square
    linkStyle 0 stroke:none
    linkStyle 1 stroke:none
    end
    end

 subgraph Sol["Design (Solution Space) &nbsp;"]
 subgraph Padder3 [ ]
    T[fa:fa-user IT Architect] ---
    UL(Ubiquitous Language) ---
    DM(Domain Model)
    BC
    linkStyle 2,3 stroke:none
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

As such, there's not just one possible Domain Model; instead,
**different Domain Models and Bounded Contexts can be constructed to
solve the business problems**: these depend on the available time and
resources as well as non-functional requirements such as availability,
latency and security. Even **the Ubiquitous Language is expected to
evolve** together with the understanding of the domain.

Service designers and solutions architects identify the solution space
defining the Domain Model and the Bounded Contexts, and agree with
[domain experts] on an [Ubiquitous Language] in order to explore the
domain and define digital services.

Services are designed and implemented iteratively, according to the
Domain Model and the Bounded Contexts which, in turn, can be tweaked to
avoid changes in a given context affecting the other ones.

In time, business requirements change and require implementation to
evolve: designers and architects need to minimize the impacts of those
changes on the various contexts to ensure a seamless evolution of the
platform.

## Ontologies model the problem space

Controlled vocabularies are a computer science tool that uses URI for disambiguating terms.
Terms are identified by URIs prefixed with the
vocabulary name which provides a namespace.

*Example: The term "Dog" according to the
[https://dbpedia.org/data/](https://dbpedia.org/data/Dog.ttl)
vocabulary is defined by the following string "The dog is a four legged
animal" and can be referenced via the URI
[https://dbpedia.org/data/Dog](https://dbpedia.org/data/Dog)*

An ontology is a vocabulary that describes a specific field of knowledge
using a formal language named Resource Description Framework ([RDF])
featuring concepts like classes, subclasses and some set theory.

Example: a minimal ontology defining the concept of Dog.

```turtle
@prefix db: <http://ontology.example/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <> .

db:Dog a owl:Class;
    rdfs:subClassOf db:Mammal;
    rdfs:label "dog";
    rdfs:comment "A dog is a four legged animal"
.

db:petName a rdf:Property;
    rdfs:domain db:Dog;
    rdfs:label "name"
.

db:Mammal a owl:Class;
    rdfs:subClassOf db:Animal;
    owl:disjointWith db:Fish
.
```

**RDF focuses on knowledge and semantic relations**.
For example, from
the above sample ontology we can infer that a `Dog` is an `Animal` but not a
`Fish`, and can have a name. Its extensions, such as RDF Schema and OWL,
provide the concept of
[class](https://www.w3.org/TR/rdf-schema/#ch_classes)
(rdfs:Class) to *"divide resources into groups"*.
To support knowledge mapping, the **RDF data model is based on graphs** .

Example: the following graph describes the relationships defined in the previous ontology.

```{mermaid}
graph LR

 classDef default stroke:#3344d0,color:#fff,clusterBkg:none,fill:#3344d0

 Dog --> |kind of| Mammal --> |kind of| Animal
 Fish --> |kind of| Animal
 Mammal -.-> |not a| Fish
 petName --> |domain| Dog
 Dog -.-> |has| petName
```

Ontologies are developed to describe a specific field using a formal
language, usually classifying entities "in their essence" and not
necessarily by "their behavior": this is aligned with the [classical
notion of ontology](https://en.wikipedia.org/wiki/Ontology) coming
from Greek philosophy.
This means that ontologies are focused on
describing knowledge, regardless of service implementations.
Technical experts are expected to use this formal knowledge to support the
creation of different kinds of digital services:
from mobile applications to backend microservices.

From a Domain-Driven perspective, **semantic web experts focus on the
domain, while technical experts focus on dynamic bounded contexts**.
RDF is thus a great tool to support the definition of an Ubiquitous Language
and the formalization of the entities belonging to a Domain Model,
but ontologies are focused on the problem space.
**To fit the solution
space, ontologies should address specific business cases and may lose
their genericity**.

```{mermaid}
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


### Semantic technologies are not optimized for syntax

[SHACL]: https://www.w3.org/TR/shacl/
[RDF]: https://www.w3.org/TR/rdf11-concepts/

To support semantic interoperability there have been various attempts to
use semantic technologies such as [RDF] and [SHACL] for data modeling.
It is important that, inside a domain, knowledge is formally described using
ontologies, and that **it should be possible to map a given datum to
RDF**, but **the physical data model and processing don't need to be
based on semantic web technologies**.

In [Ontologies model the problem
space](#ontologies-model-the-problem-space) we explain that this
would require a shift of ontologies from a generic problem space to a
service-specific solution space,
but it is possible to acknowledge this
fact and use RDF to design specific application profiles that are not
ontologies, and that are used for data definition.

The first problem is that semantic technologies are not optimized to
constrain syntax:

- while RDF terminology is similar to the one used in object-oriented
    programming (e.g. see the instance,
    [property](https://www.w3.org/TR/rdf-schema/#ch_property) and
    [subclass](https://www.w3.org/TR/rdf-schema/#ch_subclassof)
    terms), the underlying concepts may differ (see
    [OWL2OtherTech](https://www.w3.org/2007/OWL/wiki/OWL2OtherTech#Object-oriented_Programming)).
    Moreover, RDF types are not always data types.
- RDF modeling is based on graphs, while JSON and XML models are
    simple and more computational efficient trees. In particular,
    developers prefer trees since they are directed and do not contain
    cycles;
- semantic constraints (e.g. [OWL restrictions](https://www.w3.org/TR/owl-ref/),
    [rdfs:range](https://www.w3.org/TR/rdf-schema/#ch_range)) are
    meant for logical reasoning and not for constraining syntax;

- tools like Shape Constraint Language (SHACL) can address some
    syntax use cases, but they are considered overly complex and
    computationally intensive by API developers.
- interoperability is limited to XML Schema, since RDF does not
    natively support other data definition languages (e.g. [JSON Schema],
    [gRPC], ...).

[gRPC]: https://grpc.io/
[JSON Schema]: https://json-schema.org/

This means that, when we use RDF for data modeling, we need further
technologies to provide enough constraints to be able to use them for
serialization and deserialization of messages, and generate an
application profile.

> Example: the [knowsLanguage](https://schema.org/knowsLanguage)
property defined by schema.org can reference either a `Language` or a `Text`
datatype. To be used as a data schema for a digital service, it must be
assigned a specific type: if we chose the Text one, we then need to
constrain its pattern and/or define an enumeration of allowed values.

Moreover, if our API is based on JSON Schema or gRPC and not on XML
Schema, we need to ensure that the resulting schemas are consistent with
the expectations that arise from the Bounded Contexts.

Example: converting the above knowsLanguage property to JSON Schema
could result in a model that is overly complex since KnowsLanguage
allows both structured and unstructured values:

```yaml
# A JSON Schema for the KnowsLanguage object property data type.
Language:
  type: object
  required: [url]
  properties:
    name: {type: string}
    url: {type: string, format: url}
KnowsLanguage:
  anyOf:
  - string
  - $ref: #/Language

```

Summary: While ontologies are great tools to map the knowledge inside a domain,
there are various shortcomings when they are used verbatim for designing
digital services data schemas and as an Ubiquitous Language for
Domain-Driven Design.

## Interoerability concepts

[EIF]: https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/european-interoperability-framework-detail
[legal-interoperability]: https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/glossary/term/legal-interoperability
[organisational-interoperability]: https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/glossary/term/organisational-interoperability
[technical-interoperability]: https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/glossary/term/technical-interoperability
[semantic-interoperability]: https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/glossary/term/semantic-interoperability
[interoperability-model]: https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/glossary/term/interoperability-model


This document relies on the interoperability concepts described in the European Interoperability Framework (EIF) [EIF]. The EIF is a set of concepts and principles that can be used to design interoperable systems.
It is very helpful in identifying interoperability requirements and associated issues.

The EIF identifies four interoperability layers: legal, organizational, semantic, and technical.
Two systems require a certain degree of interoperability
at every layer to communicate reliably.


# Domain driven in Public Services

This section highlights the differences in Domain-Driven design between public and private services.

## What does Domain mean for public services?

**The Domain in the private sector is defined by customer expectations** and can evolve accordingly.
Unsatisfied users can change providers depending on the degree of competition in the specific business areas.
Regulation and compliance are often non-functional requirements:
their impacts on usability are mitigated by designers being free to adapt workflows and processes internally
in order to accommodate both functional and non-functional aspects.

**In the public sector**, citizen expectations in a specific Domain (e.g. healthcare) are not directly addressed by services.
Instead, **regulation provides the solution space via bureaucratic procedures**
that define the [Domain Model] and the [Bounded Context]s;
this is usually done accommodating the needs of both the citizens and the organizational structures of the agencies.
For new services, this activity is done *a priori*,
since experimenting a regulatory framework is seldom
feasible.
Digital public services' requirements are thus provided in terms
of regulation and bureaucratic procedures:
this means that their problem
space is different from the citizen's one.

Moreover, while for the citizens the problem space remains the same (e.g. to get medication),
regulatory changes might require a complete
redesign of the existing services.
Shortly, public sector digital
services can be seen from different perspectives:

- theoretically, the problem space is related to citizen needs, and
    the regulatory framework provides the Domain Model that is expected
    to deliver solutions to these needs;
- in practice, digital service implementers consider the regulation
    and all the bureaucratic procedures as the Domain,
    and the problem
    space is related to the technical implementation of all the analogic
    procedures - sometimes independently of their outcomes on the
    citizens.

```{mermaid}
flowchart LR
    classDef default stroke:#3344d0,color:#fff,clusterBkg:none,fill:#3344d0
    classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
    classDef subgraph_padding fill:none, stroke:none

subgraph Politics [Politics Problem Space]
    Domain
    Regulator
    Citizen
end

subgraph IT [IT Problem Space]
    Regulation
    Architect
    Implementation
end
Domain -.-|problem| Regulator[fa:fa-user Regulator]
Regulator ---|solution| Regulation
Domain -.-|problem| Citizen[fa:fa-user Citizen]
Citizen ---|solution| Implementation
Regulation -.- |problem| Architect[fa:fa-user Architect]
Implementation ---|solution| Architect

Politics ---  IT

linkStyle 0,2,4 color:red, stroke:red
linkStyle 1,3,5 color:green, stroke:Green
linkStyle 6 color:white, stroke:green, stroke-width:40px
```

This means that the outcome of digital public services may diverge from citizen expectation:
laws and organizational procedures are both domain and
model, problem and solution. This has various consequences, including:

- costly and ineffective digital public services, when bureaucratic
    procedures are transposed from analogic ones without a proper
    redesign; since changing regulation and procedures is complex and
    time-consuming this kind of issues may have impacts that last for years;

- bureaucratic procedures are tightly coupled with the specific organizational structures of the agencies: this may impact beyond what is necessary on the
    service design (see [How the Government’s Multibillion-Dollar Plan to Modernize Its Tech Could Go Horribly Wrong by J. Pahlka](https://onezero.medium.com/our-kill-it-with-fire-moment-f900aaabd743)). Moreover, since public sector organizations can
    suddenly change (e.g. a ministry can be split after an election to
    accommodate a newer political landscape, or two municipalities can
    be merged for demographics reasons) the identified Bounded Contexts
    can be disrupted without any new requirement change coming from citizens;

- Changes in regulations are unpredictable (for instance, as a result of court rulings, local ordinances, or federal laws), and it is difficult to predict how they will affect already-deployed services. For instance, such changes could disrupt the Design Model, forcing adjustments that spill over into other established Bounded Contexts. Larger nations tend to experience these shifts more frequently because they must deal with more use cases, and nations with federal organizations must deal with comparable use cases in various ways;

- different jurisdictions have different regulations, so different solutions to similar problem spaces;

- agencies exposing APIs intended for general/cross-border consumption
    have a hard time to identify usability requirements.

The following image shows two digital services relying on a third-party API provided by a Population Registry:
the architecture of this API
(that pre-dates the services) depends by its solution space ([Domain Model] and [Bounded Context]s) and will affect the usability and the
performance of the relying services.

```{mermaid}
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

As a last point, it is important to note that in the government sector, it is typical for regulation to serve as the primary source of the solution space.
Stakeholders need to acknowledge that and mitigate the associated issues.

## The solution space in digital public services

This section shows the elements that impacts on the
solution space in digital public services, and which strategies can mitigate the risks associated with the different perspectives
of  consumers' and IT architects' goals illustrated in [What
does Domain mean for public
services?](#what-does-domain-mean-for-public-services).

From the IT architect perspective, the problem space
shifts from user expectations to the regulation.
Semantic experts work to align ontologies to the regulation:
this affects the solution space too, including the [Ubiquitous Language].

> The image below, shows that ontology definition
> pre-dates the solution space.

```{mermaid}
%% The solution space in Public Services
flowchart TD
 classDef default stroke:#3344d0,color:#fff,clusterBkg:none,fill:#3344d0
 classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px

classDef subgraph_padding fill:none, stroke:none

 U[fa:fa-user User] -.-> |Uses| Service


 subgraph BC ["Bounded contexts (API Design) &nbsp;&nbsp; <br>"]

Service([Service])
 subgraph Padder2 [ ]
 DataModel[Data Models] ---
 Operations
 Inputs ---
 Outputs
 %% Using linkStyle and --- links to display items as a square
 end
 end

linkStyle 2 stroke:none

 subgraph Sol["Design (Solution Space) &nbsp; <br>one for each project"]

 subgraph Padder3 [ ]

end
 BC
 subgraph Padder4 [ ]
 direction RL
 T[fa:fa-user IT Architect]
 UL(Ubiquitous Language)
 DM(Domain Model)
 linkStyle 1 stroke:none
 end
 end
 subgraph Domain["Regulation (Problem Space) &nbsp; <br>"]
 subgraph Padder1 [ ]
 S[fa:fa-user Semantic Expert] -.->|models| Ontologies
 Sol
 end
 end


subgraph Government [Domain]
 Domain
 subgraph P0 [ ]

R[fa:fa-user Regulator] -->|regulates| Domain
 D[fa:fa-user Domain Expert] --- R
 end
 linkStyle 5 stroke:none
 end

 T -.-> |agrees| UL
 T -.-> |defines| BC
 UL -.-o Ontologies
 S -.-o |relies on| D
 %% Padders avoid overflows in subgraphs.

class P0,Padder1,Padder2,Padder3,Padder4 subgraph_padding

```

Service design needs to take into account:

- functional requirements based on regulation and procedures that can constrain the Domain Model (e.g. suboptimal grouping of operations inherited from analogic procedures, convoluted data models based on paper forms, ...);
- non-functional requirements like latency, durability, confidentiality, integrity and availability;
- friction in addressing issues, regulation and procedures.

When working inside a single domain, it is possible to mitigate some of these problems with [digital
checks](https://joinup.ec.europa.eu/collection/better-legislation-smoother-implementation/news/digital-readiness-better-regulation-agenda)
at legal and organizational level
(e.g. ensuring that regulation and procedures do not hinder the creation of digital services and avoid laws
providing too much procedural or technical details that should instead be delegated to the implementation) or adopting a [user-centric
approach](https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/solution/eif-toolbox/underlying-principle-6-user-centricity)
when designing specific digital services.
When multiple domains (and regulations) are involved, this is more complex.

Cross-domain interoperability is complicated by the fact that
every institution (e.g. a ministry, a department, a country):

1. has its own understanding of the interested domains;

2. addresses different problems with different regulations and procedures;

3. defines its services in terms of its own Model and Bounded Contexts, and has its internal [Ubiquitous Language] even when a common domain knowledge exists.

Agencies tend to address cross-domain interoperability shifting problems
away from the legal and organizational layers,
down to the technical layer.
Developing *ad hoc* connectors that are specific to each
interaction is considered simpler than to uniform procedures and regulations,
but this increases costs, complexity, and the attack
surface of platforms, while reducing usability.

**Who**                       | **Problem Space**         |**Solution Space**
------------------------------| ------------------------- |-------------------------
Regulator (can be an agency)  | Citizen needs             |Regulatory framework
Agencies                      | Regulatory framework      |Bureaucratic procedures
Designer / Supplier           | Bureaucratic procedures   |Compliance


Solutions involve working at all [EIF] interoperability layers:
legal, organizational, semantic, and technical.

1. address the legal and organizational layer first. This is to
   minimize the interoperability issues to be tackled at technical
   level;
2. agree on a shared terminology (and ontologies) in order to have a
   common understanding of the problem space(s);
3. avoid tightly coupling data models and operations with regulations;
   instead, try to design the Domain Model according to the problem
   space as the user sees it.

## Domain terminology in the public and the private sector

Domain terminology is key to the creation of interoperable services.
Yet, there are differences between the public and the private sector.
Since both kinds of organizations publish their common terminology using
the Resource Description Framework (see [Ontologies model the problem
space](#ontologies-model-the-problem-space)), in this section we
will rely on examples from different published vocabularies .

[Schema.org](https://schema.org/) is the result of joint efforts
from web companies like Google and Facebook to provide a common
terminology for the creation of commercial web services, and provide
both [semantic and syntactic interoperability](https://joinup.ec.europa.eu/collection/nifo-national-interoperability-framework-observatory/glossary/term/semantic-interoperability)
for indexing web pages or identifying commercial events such as flights
and concerts conveyed by messages on the Internet. As the domain grows,
for example when mashing-up APIs from different fields in order to
create an integrated service, identifying an Ubiquitous Language is more
complex: this can be seen by the complexity of the
[https://schema.org/Person](https://schema.org/Person) schema.

Anyway, **in the private sector** the domain is driven by user
expectations, and data management is based on user and provider consent.
This means that **models and contexts are easier to manage**: users are
customers, identification is generally based on the ability to be billed
(e.g. having a credit card), companies may change the terms of a service
and terminate user accounts accordingly just as users can unsubscribe
from services.

**Public sector terminologies and concepts are more fine-grained** than
the ones defined in schema.org, like shown in the examples below.
Moreover they are impacted by unforeseeable changes in regulation (e.g.
the concept of "Family" in the Taxation domain may change periodically
according to the fiscal regulation).

*Example: The Schema.org Vocabulary uses the URI
[https://schema.org/givenName](https://schema.org/givenName) to
reference the concept of given Name.*

*Example: The Italian Core Person Vocabulary uses the URI
[https://w3id.org/italia/onto/CPV/RegisteredFamily](https://w3id.org/italia/onto/CPV/RegisteredFamily)
to reference the concept of "Registered Family" that is based on a
specific regulation ([article 4 of Decreto del Presidente della
Repubblica 30/05/1989, n.
223](https://www.normattiva.it/eli/id/1989/06/08/089G0287/CONSOLIDATED)).*

*Example: The European Person Core Vocabulary includes concepts like the
patronymic name (Iceland does not have a concept of family name in the
way that many other European countries do, see*
*[https://www.w3.org/ns/person\#patronymicName](https://www.w3.org/ns/person#patronymicName))
and the birth name (see*
*[https://www.w3.org/ns/person\#birthName](https://www.w3.org/ns/person#birthName))
which can be different from the concatenation of first and last name
since these may change in time, as in the case of [Margaret Thatcher,
neé
Roberts](https://www.britannica.com/biography/Margaret-Thatcher).*

Regulation changes can affect the institutional asset of a country or
reshape administrative areas. These changes affect many domains, and may
have unintended consequences that are usually addressed by subsequent,
minor, organizational tweaks (similar to earthquake aftershocks). The
changes required to adapt digital services to the new scenario are not
driven by citizen needs that are explicit and specific to the domain.

Example: In Italy, administrative units feature "provinces" and
"metropolitan cities", introduced in 2014. Some provinces were
abolished in favor of metropolitan cities (e.g. the Metropolitan city of
Rome), others include the area of various provinces (e.g.
[Padova-Venezia-Treviso](https://it.wikipedia.org/wiki/PaTreVe))
without replacing them. For statistical purposes, the concept of
supra-municipal administrative unit was minted. Private services
continue relying on the sole concept of province (e.g. for delivery
purposes), while public services need to distinguish between the
different entities. For an example of "aftershocks", see
[https://temi.camera.it/leg18/temi/tl18_province-1.html](https://temi.camera.it/leg18/temi/tl18_province-1.html)
(in Italian and English).

## Ontologies and the solution space

expose their inner data models unnecessarily.

Ontology design needs to balance compliance with the regulation and
usefulness in describing the problem space from the citizens'
perspective.

Ontologies describe the problem space and provide a basis for Ubiquitous
Languages to be used in the various contexts. Terminology grows with the
domain (e.g. as new subdomains are added), and to be effective for
service designing, ontologies need to be developed in coordination with
all stakeholders. This is the kind of work done on schema.org (e.g. see
[github issues](https://github.com/schemaorg/schemaorg/issues?q=is%3Aissue+is%3Aopen+label%3A"schema.org+vocab")).

```{mermaid}
%% title: Communicating contexts
flowchart TB
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

From a development perspective, public sector ontologies can be seen as
software artifacts whose problem space is the regulation: this means
that they are dynamic by design, and are supposed to evolve together
with the business (the regulation). Ontologies tightly coupled with the
regulation are ineffective to model the problem space from the
citizen/user perspective, since not all regulatory changes are driven by
citizens' expectations, even when they affect the providers' ability
to erogate the service.

```{mermaid}
%% title User Expectations
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

Instead of mutuating all concepts from specific regulations, ontology
design should stay focused on the problem space as the user sees it. For
ontologies used in G2G interaction, this requires a cooperative work
between legal and domain experts from different knowledge fields.

A track history of regulatory changes, or a comparison between different
regulations, will help to identify concepts that are less likely to
change: these concepts usually have stronger relation with the problem
space.

Since ontologies are designed independently of services, it is complex
to evolve them minimizing the impacts on existing data and processes;
yet, it is advisable to engage with IT architects to make an impact
assessment on existing services.

Designing for invariants is key, though it is not always feasible.

Ontologies are used in specific domains as a base for describing data
entries: this ensures that organization's data has a clear semantic. It
is important to remark that it is not required to store data in RDF to
achieve this goal.

Semantic consistency can be achieved simply by mapping a given datum to
RDF: this can be done using different technologies and standards, as
long as this process is automated.

The diagram below shows how data stored according to a convenient
physical model can be serialized in various formats without losing
semantic information relying on mapping technologies like [the one
described in the JSON-LD 1.1
specifications](https://www.w3.org/TR/json-ld11/#interpreting-json-as-json-ld).
Thanks to these mappings, information can be rendered as RDF; moreover,
it can further be projected to lower-dimension data models using
projection maps like JSON-LD framing.

```{mermaid}
%% title Data modeling perspective

flowchart LR
 classDef default stroke:white,color:#fff,clusterBkg:none,fill:#3344d0
 classDef cluster font-weight: bold,fill:none,color:darkgray,stroke:#3344d0,stroke-width:2px
 classDef subgraph_padding fill:none, stroke:none

 subgraph Semantic [ Domain Data Models ]
 subgraph p0 [ ]
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
 direction TB
 subgraph p1 [ ]
 direction TB
 API & A[API] & B[API]

 end
 subgraph p11 [ ]
 DB

Ontologies
 end
 p1 -.-o DB & Ontologies
 end
 Semantic


System -->|produce| Data
 Data -.->|embed| RDF --> |project| Data
 DB[(Datastore<br>Physical Model)]
 class p0,p1,p2,p11 subgraph_padding

```

A drawback for processing data in RDF is the lack of support for syntax,
and the computational efficiency of working with graphs instead of using
simple trees. Inside a specific domain these issues are mitigated by the
trust - that allows reducing validation checks - and by the lower data
variance that reduces the number of ontologies involved.

When we need to model data to be used in cross-domain exchanges, it is
better to avoid modeling using domain-specific ontologies and use
mapping technologies instead. This allows services to be semantically
consistent, reducing the need for changes that are not directly related
to the business.

*Example: a service provides location information like the following:*

```
{ city: Roma, province: RM }
```

*where the province field is mapped to the URI
http://places.example/province
using a JSON-LD

```yaml
"@context":
    city: http://places.example/hasCity
    province: http://places.example/hasProvince
```

It is possible to adapt the service to the new ontology changing the
@context instead of the property name

```yaml
"@context":
    city: http://places.example/hasCity
    province: http://places.example/hasProvinceOrMetropolitanCity
```

The above example shows that modeling data on the basis of ontologies is
not always a good choice. Data models are part of the Bounded contexts,
together with the definition of procedures, inputs and outputs, and are
affected by non-functional requirements such as latency, throughput,
confidentiality and business continuity.

When used for data modeling, RDF classes shift from the Domain
description to the Bounded Contexts and should be designed and evolved
without disrupting those contexts. In many cases this conflicts with the
expectations that organizations have on [ontologies], such as their
immutability and their *a priori* definition with respect to digital
services (see [Ontologies model the problem
space](#ontologies-model-the-problem-space)).

## Aligning the solution space between Users and Architects

The solution space for architects is compliance, not user needs.

Similar considerations apply to API design: to align the solution space
between users and architects, we need to apply user-centric design to
public sector APIs. This requires challenging requirements coming from
regulation to check whether they represent legal constraints or just
habits from the bureaucratic world. Part of this work needs to be done
when designing ontologies.

In the latter case, it is possible to remove those constraints.

Another element is designing around data. While processes and data
meaning change, public sector datasets are quite static.

---

Controlled vocabularies are a computer science tool that uses URI for
disambiguating terms: every term is identified by an absolute URI,
contextualized with the vocabulary name which provides a namespace.
Vocabularies contain collection of terms and:

- define concepts and relationships in a specific domain (e.g.
    healthcare, finance, ...)
- validated by a designated authority (not necessarily a public
    authority)
- formally described using the text/turtle media type or its JSON
    counterpart: JSON-LD which is a W3C specification

Those formats are equivalent, and you can convert from turtle to json-ld
and back without losing relevant information.
Complex vocabularies are called ontologies,
while simple list of terms are called codelists.
