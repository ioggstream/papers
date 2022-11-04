# Add semantic context to APIs / Schemas

API providers usually specify semantic information in PDF or out-of-band
documents; at best, this will be human readable in the API specification
document.

The goal is simplifying the work of attaching semantic information to
APIs described using OpenAPI Specification (OAS3).

API providers may have minimal knowledge of semantic technologies.
Moreover, full-semantic technologies do not always provide easy ways of
defining / constraining the syntax of an object (tools like SHACL and
OWL restrictions being complex to use and computationally intensive to
process).

This document is based on the following materials. Readers should have
at least some familiarity with them:

-   [*https://w3c.github.io/json-ld-syntax/*](https://w3c.github.io/json-ld-syntax/)
-   [*https://www.w3.org/TR/json-ld11-framing/*](https://www.w3.org/TR/json-ld11-framing/)
-   [*https://json-schema.org/*](https://json-schema.org/)
-   [*https://github.com/OAI/OpenAPI-Specification/*](https://github.com/OAI/OpenAPI-Specification/)
-   [*https://www.w3.org/2019/wot/json-schema*](https://www.w3.org/2019/wot/json-schema)

### Acknowledgements

Thanks to the following people for their feedback:

Giorgia Lodi,
Matteo Fortini,
Saverio Pulizzi,
Pierre-Antoine Champin,
Vladimir Alexiev
and Rob Atkinson.


### Goals

-   Describe **in a single document** both the syntax and semantics of a
    JSON object
-   Describe semantically the contents defined in an API spec or a
    json-schema
-   Support for [*OAS3.0*](https://swagger.io/specification/) /
    JsonSchema Draft4
-   Easy for non-semantic experts
-   Low implementation complexity

### Design choices:

-   The semantic context of a JSON object will be described using
    [*json-ld*](https://json-ld.org/) and its keywords
-   The [*@context*](https://w3c.github.io/json-ld-syntax/#the-context)
    can always be provided via APIs using a Link header according to
    [*Section 6.1 of
    json-ld-syntax*](https://w3c.github.io/json-ld-syntax/#interpreting-json-as-json-ld)
-   Property names must not contain characters such as ":" because
    that will break code-generation tools such as swagger-codegen

### Non-Goals:

-   define a way to convert automatically RDF to JSON-schema/OAS. We
    just want to describe in a single document both the semantic and
    syntax of a JSON object. You can either edit that information by
    hand or generate it automatically using your favorite tools.
-   define a way to infer semantic information where that is not
    provided.

### Reading notes:

-   For readability, all JSON payloads are serialized using YAML
-   Described hypothesis are not in order of preference

# FAQ

Q1: Why not use a full-semantic approach?

A1: The goal is not to disrupt the current API providers and Digital
Service implementers' tooling, and they use json-schema and xsd to
define schemas.

Q2: Can't you use json-ld framing to transform JSON in JSON-LD

A2: the information source of the Framing spec is a JSON-LD document
(eg. a RDF file), while we want to interpret a [*JSON document as a
JSON-LD
doc*](https://w3c.github.io/json-ld-syntax/#interpreting-json-as-json-ld).

Q3: Can you


# Hypothesis 1 - Attach semantic information as json-schema
keywords

Register keywords in json-schema (eg. using a [*meta-schema like this
one*](https://gist.github.com/ioggstream/8e858509a3ca535c5af230986aeefaf7)
in [*draft
2020-12*](https://json-schema.org/draft/2020-12/json-schema-core.html),
or using x-jsonld- prefix in [*draft
4*](https://json-schema.org/draft-04/json-schema-core.html)) to
reference json-ld @context and @type.

Developers can reuse the provided @context, and make it accessible at a
given URL too.

Everything under "x-jsonld-context" is interpreted as a json-ld
context, while "x-jsonld-type" references an rdf:type. Applications
interested in @context can then process this field to [*derive
associated
ontologies*](https://github.com/ioggstream/json-semantic-playground/blob/master/dati_playground/schema.py#L169)
and vocabularies.

### Example 1: embedded context in OAS3 (JSON schema draft-4)

```yaml
Person:
    "x-jsonld-type": https://schema.org/Person
    "x-jsonld-context":
    "@vocab": "https://schema.org/"
    custom_id: null # detach this property from the @vocab
    country:
    @id: addressCountry
    @language: en
    type: object
    required:
    - given_name
    - family_name
    properties:
    familyName: { maxLength: 255, type: string }
    givenName: { maxLength: 255, type: string }
    country: { maxLength: 255, type: string }
    custom_id: { maxLength: 255, type: string }
```

### Example 2: referenced context in OAS3 (JSON schema
draft-4)

```yaml
Person:
 "x-jsonld-context": https://conte.xt/person.jsonld
 type: object
 required:
 - given_name
 - family_name
 properties:
   familyName: { maxLength: 255, type: string }
   givenName: { maxLength: 255, type: string }
   country: { maxLength: 255, type: string }
   custom_id: { maxLength: 255, type: string }
```

### Example 3: reuse context in another schema in (JSON schema draft-4)

```yaml
Person:
  "x-jsonld-context": &PersonContext
    "@vocab": "https://schema.org/"
    custom_id: null # detach this property from the @vocab
    country:
      @id: addressCountry
      @language: en
  type: object
  required:
  - given_name
  - family_name
  properties: &PersonProperties
    familyName: { maxLength: 255, type: string }
    givenName: { maxLength: 255, type: string }
    country: { maxLength: 255, type: string }
    custom_id: { maxLength: 255, type: string }

# The JSON-LD schema is easier to maintain.
PersonLD:
  type: object
  required:
  - given_name
  - family_name
  - "@context"
  properties:
    <<: *PersonProperties
  "@context":
    type: object
    enum:
    - <<: *PersonContext
```


Since [*json-ld does not allow specifying a rdf:type in
@context*](https://www.w3.org/2018/json-ld-wg/Meetings/Minutes/2018/2018-07-27-json-ld#section2-1),
the keyword `x-jsonld-type` (OAS3 / json-schema draft 4) or
jsonld-type (json-schema draft 2020-12) can be introduced to cover this
use case. This information is especially useful at design time.

Pros:

-   All json-ld features such as "@vocab" and "@language" are
    directly accessible without re-defining a new vocabulary for each
    json-ld feature
-   Conciseness, easy to add a @context to existing schemas
-   Easy to implement: a json-schema parser can be extended to support
    the "@context" keyword; processing can be done using existing
    json-ld library
-   The json-schema instance written for application/json, can be reused
    for the application/ld+json json-schema instance

Cons:

-   [*@context*](https://github.com/w3c/json-ld-syntax/issues/386#issuecomment-1026864738)[*
    does not aim to allow you to express any arbitrary JSON to RDF
    *](https://github.com/w3c/json-ld-syntax/issues/386#issuecomment-1026864738)transformation:
    for example there's no way to use @context to attach a rdf:type to
    JSON objects that do not contain this information. This can be
    addressed extending the keywords (eg. x-jsonld-type)
-
-   Management of nested schemas could be complex.
-   there's no way to attach a context to a string, so it is not
    possible to annotate a non-object schema (eg. `string` or
    `numeric`)

### Examples

This approach does not require implementing support for other json-ld
directives such as "@language", "@container" and so on\...

```yaml
@context:
  "@vocab": "https://w3id.org/italia/Example/"
  occupation: {@language: en}
  occupation_fr: {@language: fr}
occupation:
  type: string
  example: Student
occupation_fr:
  type: string
  example: Etudiant
```

Given the following schema and its associated context

```yaml
Person:
  type: object
  x-refersTo: https://w3id.org/italia/onto/CPV/Person
  # This custom property defines the associated json-ld
  # context that can be used to semantically describe
  # the instances.
  x-jsonld-context:
    "@vocab": "https://w3id.org/italia/onto/CPV/"
    given_name: givenName
    family_name: familyName
    education_level:
     "@id": "hasLevelOfEducation"
     "@type": "@id"
     "@context": # This sub-context attaches the vocabulary URL to the code-list.
         "@base": "http://w3id.org/italia/controlled-vocabulary/classifications-for-people/education-level/"

  properties:
    tax_code: ...
    education_level:
        type: string
        enum: [ ...values from vocabulary... ]
    family_name: ..
    given_name: ..
    examples:
        given_name: Mario
        family_name: Rossi
        education_level: NED # The string `NED` must be interpreted according to the Education Level vocab
```

Applying the json-ld context on the examples, we'll get the following
RDF ([*see in
jsonld-playground*](https://json-ld.org/playground/#startTab=tab-normalized&json-ld=%7B%22%40context%22%3A%7B%22%40vocab%22%3A%22https%3A%2F%2Fw3id.org%2Fitalia%2Fonto%2FCPV%2F%22%2C%22education_level%22%3A%7B%22%40id%22%3A%22hasLevelOfEducation%22%2C%22%40type%22%3A%22%40id%22%2C%22%40context%22%3A%7B%22%40base%22%3A%22http%3A%2F%2Fw3id.org%2Fitalia%2Fcontrolled-vocabulary%2Fclassifications-for-people%2Feducation-level%2F%22%7D%7D%2C%22given_name%22%3A%22givenName%22%2C%22family_name%22%3A%22familyName%22%7D%2C%22given_name%22%3A%22Roberto%22%2C%22family_name%22%3A%22Polli%22%2C%22education_level%22%3A%22LD%22%7D))
and the "NED" JSON string has a semantic context (eg. accessible at
[*https://w3id.org/italia/controlled-vocabulary/classifications-for-people/education-level/NED*](https://w3id.org/italia/controlled-vocabulary/classifications-for-people/education-level/NED)
and loadable in various formats - eg.
[*text/turtle*](https://ontopia-lodview.agid.gov.it/controlled-vocabulary/classifications-for-people/education-level/LD?output=text%2Fturtle)
or
[*ld+json*](https://ontopia-lodview.agid.gov.it/controlled-vocabulary/classifications-for-people/education-level/LD?output=application%2Fld%2Bjson))

```turtle
@prefix cpv: https://w3id.org/italia/onto/CPV/

_:c14n0 cpv:familyName "Mario" .
_:c14n0 cpv:givenName "Rossi" .
_:c14n0 cpv:hasLevelOfEducation
<http://w3id.org/italia/controlled-vocabulary/classifications-for-people/education-level/NED>
.
```

### Playground

There's a playground editor to test this solution at https://ioggstream.github.io/swagger-editor/

1.  start writing a simple, annotated schema referencing an ontology
    class like https://w3id.org/italia/onto/CPV/Person, and the editor
    will show all class properties


# Hypothesis 2 - Annotate OAS and json-schemas referencing
semantic assets

Every object / property is annotated according to a specific vocabulary.
A couple of vocabulary proposals are
[*[SOAS]*](http://www.intelligence.tuc.gr/~petrakis/publications/SOAS4.pdf)
using annotations like: x-refersTo, x-kindOf or
[*[JLD]*](https://www.w3.org/ns/json-ld) .

This EU document (2018)[* Deliverable 3.2.c: Semantic Interoperability
Design for Smart Object Platforms and
Services*](https://ec.europa.eu/research/participants/documents/downloadPublic?documentIds=080166e5bca75de2&appId=PPGMS)
has a similar approach using the "rdfAnnotation" keyword in
json-schema

### Example 1: x-refersTo annotations

```yaml
Person:
 x-refersTo: https://schema.org/Person
 required:
 - given_name
 - family_name
 type: object
 properties:
 family_name:
 x-refersTo: https://schema.org/familyName
 maxLength: 255
 type: string
 given_name:
 x-refersTo: https://schema.org/givenName
 maxLength: 255
 type: string
 country:
 x-refersTo: https://schema.org/addressCountry

 "jsonld:language": en
 maxLength: 255
 type: string
```

### Example 2: using json-ld annotation directly in

```yaml
Person:
 @type: https://schema.org/Person
 required:
 - given_name
 - family_name
 type: object
 properties:
 family_name:
 @id: https://schema.org/familyName

 maxLength: 255
 type: string
 given_name:

 @id: https://schema.org/givenName
 maxLength: 255
 type: string
 country:

 @id: https://schema.org/addressCountry
 @language: en
 maxLength: 255

 type: string
 tax_code:
 $ref: #/TaxCode
TaxCode:

 @id: https://w3id.org/italia/onto/CPV/taxCode
 type: string
```


Pros:

-   Every property is managed atomically
-   Each property references a well-identified vocabulary
-   Can attach semantic information at various levels (object,
    properties, \...) and in nested schemas
-   The parser can process/validate each field in isolation

Cons:

-   **The @context becomes a byproduct of the json-schema**: to cover
    all json-ld features like "@vocab", "@base", "@language",
    \... the
    [*[SOAS]*](http://www.intelligence.tuc.gr/~petrakis/publications/SOAS4.pdf)
    vocabulary has to be expanded significantly or integrated with
    [*[JLD]*](https://www.w3.org/ns/json-ld): this can be a lot of
    work [[*twitter
    thread*](https://twitter.com/jasondesrosiers/status/1445054495675748352)]
-   Continuous efforts to align the
    [*[SOAS]*](http://www.intelligence.tuc.gr/~petrakis/publications/SOAS4.pdf)
    vocabulary to the evolving json-ld specifications
-   Together with a vocabulary, a parser must be implemented
-   No native support for json-ld framing
-   Using json-ld annotations like @id and @type in json-schema can be
    semantically inaccurate, since @type refers to schema-istances, not
    , eg.

```yaml
"mailto": mailto:robipolli@gmail.com:
 "@type": Person
 givenName: Roberto
```

Instead using type should be probably done as in

```yaml
Person:
 "@type": https://www.w3.org/2019/wot/json-schema#DataSchema
 properties: {...}
```

# Hypothesis 3 - Attach context as a json-schema property

Add @context and @type property to every schema definition. Their
values will be defined as `enum: [ ..static value \...]` in OAS3
json-schema draft 4 or `const` in OAS3.1 json-schema draft 2020-12,
meaning it does not change between different instances.

Developers can reuse the provided @context, that can be accessible at a
given URL too.

Other json-ld keywords can be specified as well using the specific
annotations.

```yaml
Person:
 type: object
 required:
 - given_name
 - family_name
 properties:
   "@context":
      "const":
          "@vocab": "https://schema.org/"
          "@type":
              type: string
              default: Person
              enum: [Person] # Constraint @type to Person

   family_name:
      maxLength: 255
      type: string
   given_name:
      maxLength: 255
      type: string
```

Pros:

-   All json-ld features such as "@vocab" and "@language" are
    directly accessible without re-defining a vocabulary for each
    json-ld feature
-   Conciseness
-   Easy to implement: a custom json-schema parser has just to
    acknowledge "@context" property is a json-ld context, and use a
    suitable parser to process it
-   The schema is usable for both application/json and
    application/ld+json media-types. "@context" is optional though

Cons:

-   "@context" is a normal property and has no semantic meaning, so
    it is @type
-   the schema is actually different from the original one
-   "const" is not valid json-schema draft4, but a type: object + enum
    will work

[]{#anchor-181}Hypothesis 4 - full RDF

It is possible to describe a json-schema using [*Draft:JSON Schema in
RDF*](https://www.w3.org/2019/wot/json-schema) and json-ld and attaching
a context. Using this approach, both the json-schema and the @context
are byproducts of the json-ld.

### Example

```yaml
'@context':
 json-schema: https://www.w3.org/2019/wot/json-schema#
 jsonld: http://www.w3.org/ns/json-ld#
 jsonld:iri:
    type: @id
    '@id': http://sche.ma/Person
    '@type': json-schema:ObjectSchema
json-schema:properties:
- '@id': _:b0
  '@type': json-schema:StringSchema
  json-schema:propertyName: family_name
- '@id': _:b1
  '@type': json-schema:StringSchema
  json-schema:propertyName: given_name
- '@id': _:b2
  '@type': json-schema:StringSchema
   json-schema:propertyName: country
- '@id': _:b3
  '@type': json-schema:StringSchema
  json-schema:propertyName: custom_id
jsonld:context:
 jsonld:definitions:
 - @type: jsonld:TermDefinition
   jsonld:term: family_name
   Jsonld:iri: http://schema.org/familyName
 - @type: jsonld:TermDefinition
   jsonld:term: given_name
   Jsonld:iri: http://schema.org/givenName
 - @type: jsonld:TermDefinition
   jsonld:term: country
   Jsonld:iri: http://schema.org/addressCountry
```

Pros:

-   Theoretically consistent approach

Cons:

-   Artifacts such as json-schemas and json-ld contexts used in API
    gateways, development frameworks and tools are byproducts of RDF.
    This is risky because if the generated schemas/contexts are not
    correctly or consistently processed by tools, they may require
    further editing
-   Heavy usage of vocabularies and specifications not yet consolidated
    / consistent
-   Vocabularies need to continuously adapt to JSON Schema changes
    (draft-4, draft 2020-12, \...)
-   No actual benefits for the underlying services (API
    provider/consumer) from treating a JSONSchema as an RDF

[]{#anchor-213}Hypothesis 5 - Hybrid RDF

Using [JS-LD]
[*context*](https://gist.githubusercontent.com/ioggstream/b3fcde9a56e0b63436753ab6139fbe38/raw/eaa32eaa50f1fadb0d2d976e1e07c8c38f98ac61/jsonschema.context.jsonld)
it's possible to convert a json-schema to an RDF document, that could
be further annotated, indexed and processed.

This is quite experimental, since using [JS-LD] requires at least some
schema adaptations.

### Example
a
```yaml
{
 "@context": "https://gist.githubusercontent.com/ioggstream/b3fcde9a56e0b63436753ab6139fbe38/raw/eaa32eaa50f1fadb0d2d976e1e07c8c38f98ac61/jsonschema.context.jsonld",
 "Person": {
 "type": "object",
 "additionalProperties": false,
 "properties": {
 "given_name": {"type": "string", "maxLength":
 "family_name": {"type": "string", "maxLength":
 }t
 }
}
```

Becomes

```d
@prefix xsd:
@prefix json-schema:
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

_:b0 <json-schema:Person> _:b1 .

_:b1 <rdf:type> <json-schema:ObjectSchema> .
_:b1 <json-schema:additionalProperties> "false"^^<xsd:boolean> .
_:b1 <json-schema:properties> _:b2 .
_:b1 <json-schema:properties> _:b3 .

_:b2 <rdf:type> <json-schema:StringSchema> .
_:b2 <json-schema:maxLength> "255"^^<xsd:integer> .
_:b2 <json-schema:propertyName> "family_name" .

_:b3 <rdf:type> <json-schema:StringSchema> .
_:b3 <json-schema:maxLength> "255"^^<xsd:integer> .
_:b3 <json-schema:propertyName> "given_name" .
```

For example, since @context contains `@vocab: json-schema`, all
schema names are treated as json-schemas. To fix this up, we need to
modify the above schema attaching `$id`s. This makes it difficult to
support OAS3.0 schemas

Pros:

-   theoretically consistent approach, with limitations

Cons:

-   Vocabularies need to continuously adapt to JSONSchema changes
    (draft-4, draft 2020-12, \...)
-   Needs different vocabularies for JSONSchema draft 4 and 2020-12
-   No actual benefits for the underlying services from treating a
    JSONSchema as an RDF

[]{#anchor-238}Hypothesis 6 - Attach context and term definitions as a
json-schema property

This is actually a mix between hypothesis 2 and 3 above.

```yaml
Person:
  type: object
  x-jsonld-context:
    s: "https://schema.org/"
  required:
  - given_name
  - family_name
  properties:
    "@type":
        type: string
        default: Person
        enum: [Person] # Constraint @type to Person
    family_name:
        x-jsonld-termdef:
            @id: "s:familyName"
            @language: en
        maxLength: 255
        type: string
    given_name:
        type: string
        x-jsonld-termdef:
            @id: "s:givenName"
            maxLength: 255
```

The actual context could be assembled from the different parts in
x-jsonld-context and x-jsonld-termdef.

Pros:

-   human readability, easy to maintain
-   better than Hypothesis 2

Cons:

-   @context a byproduct of the schema: the final context may be
    constrained by the possibilities of the composition, while treating
    it as a whole provides all the flexibility of json-ld;
-   assembled context should still be reviewed, and possibly edited, to
    ensure they are consistent.
