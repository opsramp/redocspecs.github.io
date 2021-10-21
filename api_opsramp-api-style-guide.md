# OpsRamp API Design Guidelines


# Introduction

The OpsRamp platform is a collection of reusable services that encapsulate well-defined business capabilities. Developers are encouraged to access
these capabilities through Application Programming Interfaces (APIs).This facilitates a great developer experience and the ability to quickly
compose complex business processes by combining multiple, complementary capabilities as building blocks.

OpsRamp  APIs follow the RESTful architectural style as much as possible. To support our objectives, we have developed a set of rules,
standards, and conventions that apply to the design of RESTful APIs.This document provides guidelines and examples to help developers design simple, consistent and easy-to-use RESTful APIs.
To provide the seamless development experience for developers, it's important to have APIs that follow consistent design guidelines. 
Consistency allows teams to leverage common code, patterns, documentation and design decisions meet the needs of a wide variety of use cases.
These guidelines aim to achieve the following objectives:

* ***Define consistent practices and patterns for all REST APIs.***
* ***Adhere as closely as possible to accepted REST/HTTP good practices.***

The material details the common guidelines when developing RESTful web services for OpsRamp in the following areas as listed below.

- **Resource and representation design**
- **URIs**
- **Security**
- **Batch processing**
- **Error handling**
- **Versioning**

# Document Semantics, Formatting, and Naming

The keywords in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).
The words "REST" and "RESTful" MUST be written as presented here, representing the acronym as all upper-case letters. This is also true of "JSON," "XML," and other acronyms.
Machine-readable text, such as URLs, HTTP verbs, and source code, are represented using a fixed-width font.
URIs containing variable blocks are specified according to [URI Template RFC 6570](https://tools.ietf.org/html/rfc6570). For example, a URL containing a variable called resource_id would be shown as https://foo.com/resources/{resource_id}/.
HTTP headers are written in camelCase + hyphenated syntax, e.g. Foo-Request-Id.

# Table Of Contents

* [Interpreting the OpsRamp API Guidelines](#interpretation)
    * [Terms Used](#terms)
* [Service Design Principles](#service-design-principles)
    * [Loose Coupling](#loose_coupling)
    * [Encapsulation](#encapsulation)
    * [Stability](#stability)
    * [Reusable](#reusable)
    * [Contract-based](#contract-based)
    * [Consistency](#consistency)
    * [Ease of Use](#ease-of-use)
    * [Externalizable](#externalizable)
* [OpsRamp Rest Principles](#opsramp-rest-principles)
    * [Resource Modeling](#resource-modeling)
* [URI](#uri)
    * [Resource Path](#resource-path)
    * [URI Component](#uri-component-names)
    * [URI Authority](#uri-authority-design)
* [Resource Name](#resource-names)
* [Query Parameters](#query-parameters)
* [HTTP Methods, Headers and Statuses](#http-methods-headers-and-statuses)
    * [Data Resources/HTTP Methods](#data-resources)
	   * [HTTP Methods](#http-methods)
	   * [Processing](#processing)
    * [HTTP Headers](#http-headers)
	   * [Assumptions](#assumptions)
	   * [HTTP Standard Request Headers](#http-standard-request-headers)
	   * [HTTP Custom Headers](#http-custom-headers)
	   * [HTTP Standard Response Headers](#http-standard-response-headers)
	* [Standard Response Formats](#standard-response-formats)
	* [Pagination](#pagination)
    * [HTTP Status Codes](#http-status-codes)
	   * [Status Code Ranges](#status-code-ranges)
	   * [Status Reporting](#status-reporting)
	   * [Allowed Status Codes List](#allowed-status-codes)
	   * [HTTP Method to Status Code Mapping](#mapping)
* [JSON Schema](#json-schema)
    * [API Contract Description](#api-contract-description)
* [Common Types](#common-types)
    * [Address](#address)
    * [Internationalization](#internationalization)
    * [Date,Time and Timezone](#date-time)
    * [Formats](#formats)
* [Error Handling](#error-handling)
    * [Error Schema](#error-schema)
    * [Error Samples](#error-samples)
    * [Error Declaration In API Specification](#error-declaration)
    * [Samples With Error Scenarios In Documentation](#userguide-errors)
    * [Error Catalog](#error-catalog)
* [API Versioning](#api-versioning)
    * [API Lifecycle](#api-lifecycle)
    * [API Versioning Policy](#api-versioning-policy)
    * [Backwards Compatibility](#backwards-compatibility)
    * [End of Life Policy](#eol-policy)
* [Deprecation](#deprecation)
    * [Terms Used](#deprecation-terms-used)
    * [Background](#deprecation-background)
    * [Requirements](#deprecation-requirements)
    * [Solution](#deprecation-solution)
* [References](#references)

<h1 id="interpretation">Interpreting The Guidelines</h1>

<h2 id="terms">Terms Used</h2>

<h4 id="resource">Resource</h4>

The key abstraction of information in REST is a resource. According to [Fielding's dissertation section 5.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2), any information that can be named can be a resource: a document or image, a temporal service (e.g. "today's temperature  in Hyderabad"), a collection of other resources, a non-virtual object (e.g. a person), and so on. A resource is a conceptual mapping to a set of entities, not the entity that corresponds to the mapping at any particular point in time. More precisely, a resource R is a temporally varying membership function `MR(t)`, that for time `t` maps to a set of entities, or values, that are equivalent. The values in the set may be resource representations and/or resource identifiers.

A resource can also map to the empty set, that allows references to be made to a concept before any realization of that concept exists.

<h4 id="resource-identifier">Resource Identifier</h4>

REST uses a resource identifier to identify the particular resource instance involved in an interaction between components. The naming authority (an organization providing APIs, for example) that assigned the resource identifier making it possible to reference the resource, is responsible for maintaining the semantic validity of the mapping over time (ensuring that the membership function does not change). - [Fielding's dissertation section 5.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)

<h4 id="representation">Representation</h4>

REST components perform actions on a resource by using a representation to capture the current or intended state of that resource and by transferring that representation between components. A representation is a sequence of bytes, plus representation metadata to describe those bytes - [Fielding dissertation section 5.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2).


<h4 id="domain">Domain</h4>

According to Wikipedia, a [domain model](https://en.wikipedia.org/wiki/Domain_model) is a system of abstractions that describes selected aspects of a sphere of knowledge, influence, or activity. The concepts include the data involved in a business, and the rules that the business uses in relation to that data. As an example, the OpsRamp domain model abstracts IT Operations management entities which includes Resources, Alerts, Metrics, Configurations, Instrumentation, Tenancy.

<h4 id="capability">Capability</h4>

Capability represents a business-oriented and customer-facing view of an organization's business logic. Capabilities could be used to organize portfolio of APIs as a stable, business-driven view of its system, consumable by customers and experiences. Examples of capability are: Monitoring Metric data, Monitoring Resources, Service entities and other IT infrastructure related entities.

Capabilities drive the interface, while domains are more coarse-grained and closer to the code and the organization's structure. Capability and domain are seen as orthogonal concerns from a service perspective.

<h4 id="namespace">Namespace</h4>

Capabilities drive service modeling and namespace concerns in an API portfolio. Namespaces are part of the Business Capability Model. Examples of namespace are: Resource Management, Service Management, Automation, 3rd-Party Integrations.

Namespaces should reflect the domain that logically groups a set of business capabilities. Domain definition should reflect the customer's perspective on how platform capabilities are organized. Note that these may not necessarily reflect the company's hierarchy, organization, or (existing) code structure.

<h4 id="service">Service</h4>

Services provide a generic API for accessing and manipulating the value set of a [resource](#resource), regardless of how the membership function is defined or the type of software that is handling the request. Services are generic pieces of software that can perform any number of functions. It is, therefore, instructive to think about the different types of services that exist.

<h4 id="client">Client, API Client, API Consumer</h4>

An entity that invokes an API request and consumes the API response.


<h1 id="service-design-principles">Service Design Principles</h1>

This section captures the principles guiding the design of the services that expose APIs to internal and external developers, adjacencies, partners and affiliates. A service refers to functionality pertaining to a particular capability, exposed as an API.

Following are the core design principles for a service.

<h2 id="loose_coupling">Loose Coupling</h2>

<strong>Services and consumers must be loosely coupled from each other.</strong>

Coupling refers to a connection or relationship between two things. A measure of coupling is comparable to a level of dependency. This principle advocates the design of service contracts, with a constant emphasis on reducing (*loosening*) dependencies between the service contract, its implementation, and service consumers.

The principle of Loose Coupling promotes the independent design and evolution of a service’s logic and implementation while still emphasizing baseline interoperability with consumers that have come to rely on the service’s capabilities.

This principle implies the following:

* A service contract should not expose implementation details
* A service contract can evolve without impacting existing consumers
* A service in a particular domain can evolve independently of other domains


<h2 id="encapsulation"/>Encapsulation</h2>

<strong>A domain service can access data and functionality it does not own through other service contracts only.</strong>

A service exposes functionality that comprises the functionality and data it owns and implements, as well as the functionality and data it depends upon which it does not own. This principle advocates that any functionality or data that a service depends on and which it does not own must be accessed through service contracts only.

This principle implies the following:

* A service has a clear isolation boundary - a clear scope of ownership in terms of functionality and data
* A service cannot expose the data it does not own directly

<h2 id="stability"/>Stability</h2>

<strong>Service contracts must be stable.</strong>

Services must be designed in such a way that the contract they expose remains valid for existing customers. Should the service contract need to evolve in an incompatible fashion for the consumer, this should be communicated clearly.

This principle implies the following:

* Existing clients of a service must be supported for a documented period of time
* Additional functionality must be introduced in a way that does not impact existing consumers
* Deprecation and migration policies must be clearly stated to set consumers' expectations

<h2 id="reusable"/>Reusable</h2>

<strong>Services must be developed to be reusable across multiple contexts and by multiple consumers.</strong>

The main goal of an API platform is to enable applications to be developed quickly and cost effectively by using and combining services. This is possible only if the service contracts have been developed with flexibility for multiple use-cases and multiple consumers. This principle advocates that services be developed in a manner that enables them to be used by multiple consumers and in multiple contexts, some of which may evolve over time.

This principle implies the following:

* A service contract should be designed for not just the immediate context, but with support and/or extensibility to be used by multiple consumers in different contexts
* A service contract may need to incrementally evolve to support multiple contexts and consumers over time

<h2 id="contract-based"/>Contract-based</h2>

<strong>Functionality and data must only be exposed through standardized service contracts.</strong>

A service exposes its purpose and capabilities via a service contract. A service contract comprises of functional aspects, non-functional aspects (such as availability, response-time), and business aspects (such as cost-per-call, terms and conditions). _Standardized_ means that the service contracts must be compliant with the contract design standards.

This principle advocates that all functionality and data must only be exposed through standardized service contracts. Consumers of services can, therefore, understand and access functionality and data only through service contracts.

This principle implies the following:

* Functionality and data cannot be understood or accessed outside of service contracts
* Each piece of data (such as that managed in a datastore) is owned by only one service

<h2 id="consistency"/>Consistency</h2>

<strong>Services must follow a common set of rules, interaction styles, vocabulary and shared types.</strong>

A set of rules prescribes the definition of services in order to expose those in a consistent manner. This principle increases the ease of use of the API platform by reducing the learning curve for consumers of new services.

This principle implies the following:

* A set of standards is defined for services to comply with
* A service should use vocabulary from common and shared dictionaries
* Compatible interaction styles, service granularity and shared types are key for full interoperability and ease of service compositions

<h2 id="ease-of-use"/>Ease Of Use</h2>

<strong>Services must be easy to use and compose in consumers (and applications).</strong>

A service that is difficult and time consuming to use reduces the benefits of a microservices architecture by encouraging consumers to find alternate mechanisms to access the same functionality. Composability means that services can be combined easily because the service contracts and access protocols are consistent, and each service contract does not have to be understood differently. 

This principle implies the following:

* A service contract is easily discoverable and understandable
* Service contracts and protocols are consistent on all aspects that they can be - e.g. identification and authentication mechanisms, error semantics, common type usage, pagination, etc.
* A service has clear ownership, so that consumer providers can reach service owners regarding SLAs, requirements, issues
* A consumer provider can easily integrate, test, and deploy a consumer that uses this service
* A consumer provider can easily monitor the non-functional aspects of a service

<h2 id="externalizable"/>Externalizable</h2>

<strong>Service must be designed so that the functionality it provides is easily externalizable.</strong>

A service is developed for use by consumers that may be from another domain or team, another business unit or another company. In all of these cases, the functionality exposed is the same; what changes is the access mechanism or the policies enforced by a service, like authentication, authorization and rate-limiting. Since the functionality exposed is the same, the service should be designed once and then externalized based on business needs through appropriate policies.

This principle implies the following:

* The service interface must be derived from the domain model and the intended use-cases it is meant to support
* The service contract and access (binding) protocols supported must meet the consumer's needs
* The externalization of a service must not require reimplementation, or a change in service contract
</ul>


<h1 id="REST-principles">OpsRamp REST Principles</h1>


#### URIs matter 
RESTful APIs use Uniform Resource Identifiers (URIs) to address resources. And, a good resource model makes an API easy to produce, consume, discover, and extend.
#### Parameters matter 
Use a standard and easy-to-guess set of optional parameters for each API call.
#### Data format matters
Make it straightforward for programmers to understand the data the API expects, what kind of data it will return, and how to change that.
#### Return codes matter
Don't return 200 (OK) when you should be returning 201 (Created) and a Location header pointing to the location of the new resource.
#### Versioning matters
Always version your API. It's critical that clients can count on services to be stable over time, and it's critical that services can add features and make changes. By default, all requests receive the latest version of the REST API
 
###  Resource oriented design appoach

When designing resource-oriented APIs, follow these steps

- Determine what types of resources an API provides
- Determine the relationships between resources
- Decide the resource name schemes based on types and relationships
- Decide the resource schemas
- Attach a minimum set of methods to the resources

<h2 id="resource-modeling">Resource Modeling</h2>
Resource modelling is an exercise that establishes your API’s key concepts. Before diving directly into the design of
URI paths, it may be helpful to first think about the REST API’s resource model.

### Resource Archetypes
When modelling an API’s resources, we can start with the some basic resource archetypes. Like design patterns, the 
resource archetypes help us consistently communicate the structures and behaviours that are commonly found in REST API 
designs. A REST API is composed of four distinct resource archetypes: **document**, **collection**, **store**, and 
**controller**.

### Document
A document resource is a singular concept that is akin to an object instance or a database record. A document’s state 
representation typically includes both fields with values and links to other related resources. With its fundamental  
field and link-based structure, the document type is the conceptual base archetype of the other resource archetypes.
In other words, the three other resource archetypes can be viewed as specialisations of the document archetype.

Each URI below identifies a document resource:
```
https://api.xyz.opsramp.com/alerts/policies
https://api.xyz.opsramp.com/incidents/comments
```

A document may have child resources that represent its specific subordinate concepts. With its ability to bring many 
different resource types together under a single parent, a document is a logical candidate for a REST API’s root 
resource, which is also known as the "service root". The example URI below identifies the service root, which is the 
Superannuation business area's REST API entry point:
```
https://try.xyz.api.opsramp.com/
```

The service root should be considered a “namespace”. Namespaces should reflect the consumer's perspective on how a 
resource-oriented API should work, and not necessarily the producer organisations.

### Collection
A collection resource is a server-managed directory of resources. Clients may propose new resources to be added to a 
collection. However, it is up to the collection to choose to create a new resource, or not. A collection resource 
chooses what it wants to contain and also decides the URIs of each contained resource.

Each URI below identifies a collection resource:
```
https://api.xyz.opsramp.com/alerts
https://api.xyz.opsramp.com/incidents
```
### Store
A store is a client-managed resource repository. A store resource lets an API client put resources in, get them back 
out, and decide when to delete them. On their own, stores do not create new resources; therefore a store never 
generates new URIs. Instead, each stored resource has a URI that was chosen by a client when it was initially put into 
the store.

The example update on an incident with ID 1234567
```
POST  https://api.xyz.opsramp.com/incidents/1234567
```

### Controller
A controller resource models a procedural concept. Controller resources are like executable functions, with parameters 
and return values; inputs and outputs. Like a traditional web application’s use of HTML forms, a REST API relies on 
controller resources to perform application-specific actions that cannot be logically mapped to one of the standard 
methods (create, retrieve, update, and delete).

Controller names typically appear as the last segment in a URI path, with no child resources to follow them in the 
hierarchy. The example below shows a controller resource (from a fictional REST API) that allows a client to resend an 
alert to a user:
```
POST /alerts/4321/acknowledge
```

<h1 id="uri">URI</h1>

<h2 id="resource-path">Resource Path</h2>

An API's [resource path](#uri-components) consists of URI's path, query and fragment components. It would include API's major version followed by namespace, resource name and optionally one or more sub-resources. For example, consider the following URI.

`https://api.foo.com/v1/CLIENT_INFO/alerts/alert-ID

Following table lists various pieces of the above URI's resource path.

|Path Piece|Description|Definition|
|---|---|---|
|`v1`|Specifies major version 1 of the API|The API major version is used to distinguish between two backward-incompatible versions of the same API. The API major version is an integer value which MUST be included as part of the URI. |
|`CLIENT_INFO`|The namespace|Namespace identifiers are used to provide a context and scope for resources. They are determined by logical boundaries in the business capability model implemented by the API platform.|
|`alerts`|The resource name|If the resource name represents a collection of resources, then the `GET` method on the resource should retrieve the list of resources. Query parameters should be used to specify the search criteria.|
|`alert-ID`|The resource ID|To retrieve a particular resource out of the collection, a resource ID MUST be specified as part of the URI. Sub-resources are discussed below.|

Naming conventions for URIs, query parameters, resources, fields and enums are described in this section. Let us emphasize here that these guidelines are less about following the conventions exactly as described here but they are more about defining some naming conventions and sticking to them in a consistent manner while designing APIs. For example, we have followed [snake_case](https://en.wikipedia.org/wiki/Snake_case) for field and file names, however, you could use other forms such as [CamelCase](https://en.wikipedia.org/wiki/Camel_case) or something else that you have devised yourself. It is important to adhere to a defined convention.

<h2 id="uri-component-names">URI Component Names</h2>

URIs follow [RFC 3986][8] specification. This specification simplifies REST API service development and consumption. The guidelines in this section govern your URI structure and semantics following the RFC 3986 constraints.

<h3 id="uri-components">URI Components</h3>

According to RFC 3986, the generic URI syntax consists of a hierarchical sequence of components referred to as the scheme, authority, path, query, and fragment as shown in example below.


```
         https://example.com:8042/over/there?name=ferret#nose
         \___/   \_______________/\_________/\_________/\__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
```

<h3 id="uri-naming-convention">URI Naming Conventions</h3>

Following is a brief description of the URI specific naming convention guidelines for APIs. This specification uses parentheses "( )" to group, an asterisk " * " to specify zero or more occurrences, and brackets "[ ]" for optional fields.

```
[scheme"://"][host[':'port]]"/v" major-version '/'namespace '/'resource ('/'resource)* '?' query

```

* URIs MUST start with a letter and use only lower-case letters.
* Literals/expressions in URI paths SHOULD be separated using a hyphen ( - ).
* Literals/expressions in query strings SHOULD be separated using underscore ( _ ).
* URI paths and query strings MUST percent encode data into [UTF-8 octets](https://en.wikipedia.org/wiki/Percent-encoding).
* Plural nouns SHOULD be used in the URI where appropriate to identify collections of data resources.
  * `/alerts`
  * `/incidents`
* An individual resource in a collection of resources MAY exist directly beneath the collection URI.
  * `/alerts/{alert_id}`
* Sub-resource collections MAY exist directly beneath an individual resource. This should convey a relationship to another collection of resources (invoice-items, in this example).
  * `/incidents/{incident_id}/alerts`
* Sub-resource individual resources MAY exist, but should be avoided in favor of top-level resources.
  * `/incidents/{incident_id}/alerts/{alert_id}`
  * Better: `/incident-alerts/{incident_alert_id}`
* Resource identifiers SHOULD follow [recommendations](#resource-identifiers) described in subsequent section.

**Examples**

* `https://api.foo.com/v1/CLIENT_INFO/alerts`
* `https://api.foo.com/v1/CLIENT_INFO/alerts/unique-ID-7LT50814996943336KESEVWA`
* `https://api.foo.com/v1/CLIENT_INFO/alerts/actions/unique-ID-XX/alert-action-ID`


**Formal Definition:**

|Term|Defiition|
|---|---|
|URI|[end-point] '**/**' resource-path ['**?**'query]|
|end-point|[scheme "**://**"][ host ['**:**' port]]|
|scheme|"**http**" or "**https**"|
|resource-path|"**/v**" version  '**/**' namespace-name '**/**' resource ('**/**' resource)|
|resource|resource-name ['**/**' resource-id]|
|resource-name|Alpha (Alpha \| Digit \| '-')* |
|resource-id|value|
|query|name '**=**' value ('**&**' name = value)* |
|name|Alpha (Alpha \| Digit \| '_')* |
|value|URI Percent encoded value|


Legend

```
'   Surround a special character with single quotes
"	Surround strings with double quotes
()	Use parentheses for grouping
[]	Use brackets to specify optional expressions
*	An expression can be repeated zero or more times
```

## URI Authority Design

This section discusses the naming conventions that should be used for the authority portion of a REST API.

### Consistent subdomain names should be used for your APIs
The top-level domain and first subdomain names (e.g., xyz.app.opsramp.com, xyz.try.opsramp.com) of an API should identify its service 
owner. These are custome branded domains.The full domain name of an API should add a subdomain named api.

For example:
```
https://[ Custom branded url].api.opsramp.com
https://xyz.api.opsramp.com
```
API status:
* An API status page. See: https://www.status.opsramp.io/

<h1 id="resource-names">Resource Names</h1>

When modeling a service as a set of resources, developers MUST follow these principles:

* Nouns MUST be used, not verbs.
* Resource names MUST be singular for singletons; collections' names MUST be plural.
    * A description of the alerts on a resource
        * `GET /alerts` returns the full representation
    * A collection of hypothetical resources:
        * `GET /resources` returns a list of resources managed
        * `POST /resources` creates a new  resource, `/resources/1234`
        * `GET /resources/1234` returns a full representation of a resource

* Resource names MUST be lower-case and use only alphanumeric characters and hyphens.
    * The hyphen character, ( - ), MUST be used as a word separator in URI path literals. **Note** that this is the only place where hyphens are used as a word separator. In nearly all other situations, the underscore character, ( _ ), MUST be used.


<h3 id="subresource-path">Sub-Resources</h3>

Sub-resources represent a relationship from one resource to another. The sub-resource name provides a meaning for the relationship. If cardinality is 1:1, then no additional information is required. Otherwise, the sub-resource SHOULD provide a sub-resource ID for unique identification. If cardinality is 1:many, then all the sub-resources will be returned. No more than two levels of sub-resources SHOULD be supported.

|Example|Description|
|---|---|
|`GET https://api.foo.com/CLIENT_INFO/alerts/incidents/INC-ID`|This call should return all the incidents associated with dispute ABCD1234.|
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents/102030`|This call should return only the details for a particular document associated with this dispute. Keep in mind that this is only an illustrative example  to show how to use sub-resource IDs. In practice, two step invocations SHOULD be avoided. If second identifier is unique, top-level resource (e.g. `/v1/customer-support/documents/102030`) is preferred.|
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/transactions`|The following example should return all the transactions associated with this dispute and their details, so there SHOULD NOT be a need to specify a particular transaction ID. If specific transaction ID retrieval is needed, `/v1/customer-support/transactions/ABCD1234` is preferred (assuming IDs are unique).|

<h3 id="resource-identifiers">Resource Identifiers</h3>

[Resource identifiers](#resource-identifier) identify a resource or a sub-resource. These **MUST** conform to the following guidelines.

* The lifecycle of a resource identifier MUST be owned by the resource's domain model, where they can be guaranteed to uniquely identify a single resource.
* APIs MUST NOT use the database sequence number as the resource identifier.
* A [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier), Hashed Id ([HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) based) is preferred as a resource identifier.
* For security and data integrity reasons all sub-resource IDs MUST be scoped within the parent resource only.<br />
**Example:** `/users/1234/roles/ABCD`<br /> Even if account "ABCD" exists, it MUST NOT be returned unless it is linked to user: 1234.
* Enumeration values can be used as sub-resource IDs. String representation of the enumeration value SHOULD be used.
* There MUST NOT be two resource identifiers one after the other.<br/>
**Example:** `https://api.foo.com/v1/CLIENT_INFO/alerts/12345`
* Resource IDs SHOULD try to use either Resource Identifier Characters or ASCII characters. There **SHOULD NOT** be any ID using UTF-8 characters.
* Resource IDs and query parameter values MUST perform URI percent-encoding for any character other than URI unreserved characters. Query parameter values using UTF-8 characters MUST be encoded.



<h3 id="query-parameter-names">Query Parameter Names</h3>

* Literals/expressions in query strings SHOULD be separated using underscore ( _ ).
* Query parameters values MUST be percent-encoded.
* Query parameters MUST start with a letter and SHOULD be all in lower case. Only alpha characters, digits and the underscore ( _ ) character SHALL be used.
* Query parameters SHOULD be optional.
* Some query parameter names are reserved, as indicated in [Resource Collections](#resource-collections).

For more specific info on the query parameter usage, see [URI Standards](#uri-standards-query).

<h3 id="field-names">Field Names</h3>

The data model for the representation MUST conform to [JSON][30]. The values may themselves be objects, strings, numbers, booleans, or arrays of objects.

* Key names MUST be lower-case words, separated by an underscore character, ( _ ).
    * `foo`
    * `bar_baz`
* Prefix such as `is_` or `has_` SHOULD NOT be used for keys of type boolean.
* Fields that represent arrays SHOULD be named using plural nouns (e.g. authenticators-contains one or more authenticators, products-contains one or more products).

<h3 id="enum-names">Enum Names</h3>

Entries (values) of an enum SHOULD be composed of only upper-case alphanumeric characters and the underscore character, ( _ ).

* `FIELD_10`
* `NOT_EQUAL`

If there is an industry standard that requires us to do otherwise, enums MAY contain other characters.


<h1 id="query-parameters">Query Parameters</h1>


Query parameters are name/value pairs specified after the resource path, as prescribed in RFC 3986.
[Naming Conventions](#naming-conventions) should also be followed when applying the following section.

#### Filter a resource collection

* Query parameters SHOULD be used only for the purpose of restricting the resource collection or as search or filtering criteria.
* The resource identifier in a collection SHOULD NOT be used to filter collection results, resource identifier should be in the URI.
* Default sort order SHOULD be considered as undefined and non-deterministic. If a explicit sort order is desired, the query parameter `sort` SHOULD be used with the following general syntax: `{field_name}|{asc|desc},{field_name}|{asc|desc}`. For instance: `/accounts?sort=date_of_birth|asc,zip_code|desc`


#### Cache-friendly APIs
In rare cases where a resource needs to be highly cacheable (usually data with minimal change), query parameters MAY be utilized as opposed to `POST` + request body. As `POST` would make the response uncacheable, `GET` is preferred in these situations. This is the only scenario in which query parameters MAY be required.

#### Query parameters with POST
When `POST` is utilized for an operation, query parameters are usually NOT RECOMMENDED in favor of request body fields. In cases where `POST` provides paged results (typically in complex search APIs where `GET` is not appropriate), query parameters MAY be used in order to provide hypermedia links to the next page of results.

#### Passing multiple values for the same query parameter

When using query parameters for search functionality, it is often necessary to pass multiple values. For instance, it might be the case that a resource could have many states, such as `OPEN`, `CLOSED`, and `INVALID`. What if an API client wants to find all items that are either `CLOSED` or `INVALID`?

* It is RECOMMENDED that APIs implement this functionality by repeating the query parameter. This is inherently supported by HTTP standards and already built in to most client libraries.
    * The query above would be implemented as `?status=CLOSED&status=INVALID`.
    * The parameter MUST be marked as repeatable in API specifications using `"repeated": true` in the parameter's definition section.
    * The parameter's name SHOULD be singular.

* URIs have practical length limits that are quite low - most conservatively, about [2,000 characters](https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers#417184). Therefore, there are situations where API designers MAY choose to use a single query parameter that accepts comma-separated values in order to accommodate more values in the query-string. Keep in mind that server and client libraries don't consistently provide this functionality, which means that implementers will need to write additional string parsing code. Due to the additional complexity and divergence from HTTP standards, this solution is NOT RECOMMENDED unless justified.
    * The query above would be implemented as `?statuses=CLOSED,INVALID`.
    * The parameter MUST NOT be marked as repeatable in API specifications.
    * The parameter MUST be marked as `"type": "string"` in API specifications in order to accommodate comma-separated values. Any other `type` value MUST NOT be used. The parameter description should indicate that comma separated values are accepted.
    * The query-parameter name SHOULD be plural, to provide a hint that this pattern is being employed.
    * The comma character (Unicode `U+002C`) SHOULD be used as the separator between values.
    * The API documentation MUST define how to escape the separator character, if necessary.


<h1 id="json-schema">JSON Schema</h1>

We would assume that [JSON Schema](http://json-schema.org/) is used to describe request/response models. Determine the version of JSON Schema to use for your APIs

<h2 id="api-contract-description">API Contract Description</h2>

[OpenAPI](https://github.com/OAI/OpenAPI-Specification) is a vendor neutral API description format as defined by latest schema of OpenAPI [Schema Object](https://github.com/OAI/OpenAPI-Specification/blob/master/schemas/v3.1/schema-base.json) 
In addition, there are extensions provided by the  specification to allow for more complete documentation.
The API implementation SHOULD instead enforce the conformance of requests and responses to an API contract by hard validating the requests and
responses against the defined API contract at run-time.

<h2 id="common-types">Common Types</h2>

Resource representations in API MUST reuse the [common data type] (TBD) definitions where possible. Following sections provide some details about some of these common types.

<h3 id="address">Address</h3>
TBD

<h3 id="phone-no">Phone no</h3>
TBD
<h3 id="internationalization">Internationalization</h3>

The following common types MUST be used with regard to global country, currency, language and locale.

* [`country_code`](v1/schema/json/draft-04/country_code.json)
  * All APIs and services MUST use the [ISO 3166-1 alpha-2](http://www.iso.org/iso/country_codes.htm) two letter country code standard.
* [`currency_code`](v1/schema/json/draft-04/currency_code.json)
  * Currency type MUST use the three letter currency code as defined in [ISO 4217](http://www.currency-iso.org/). For quick reference on currency codes, see [http://en.wikipedia.org/wiki/ISO_4217](http://en.wikipedia.org/wiki/ISO_4217).
* [`language.json`](v1/schema/json/draft-04/language.json)
 	* Language type uses [BCP-47](https://tools.ietf.org/html/bcp47) language tag.
* [`locale.json`](v1/schema/json/draft-04/locale.json)
 	* Locale type defines the concept of locale, which is composed of `country_code` and `language`. Optionally, IANA timezone can be included to further define the locale.
* [`province.json`](v1/schema/json/draft-04/province.json)
    * Province type provides detailed definition of province or state, based on [ISO-3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) country subdivisions, with room for variant local, international, and abbreviated representations of province names. Useful for logistics, statistics, and building state pull-downs for on-boarding.

<h3 id="date-time">Date, Time and Timezone</h3>

When dealing with date and time, all APIs MUST conform to the following guidelines.

* The date and time string MUST conform to the `date-time` universal format defined in section `5.6` of [RFC3339][21]. In use cases where you would require only a subset of the fields (e.g `full-date` or `full-time`) from the RFC3339 `date-time` format, you SHOULD use the [Date Time Common Types](#date-common-types) to express these.

* All APIs MUST only emit [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) time (aka [Zulu time](https://en.wikipedia.org/wiki/List_of_military_time_zones) or [GMT](https://en.wikipedia.org/wiki/Greenwich_Mean_Time)) in the responses.

* When processing requests, an API SHOULD accept `date-time` or time fields that contain an offset from UTC. For example, `2016-09-28T18:30:41.000+05:00` SHOULD be accepted as equivalent to `2016-09-28T13:30:41.000Z`. This helps ensure compatibility with third parties who may not be capable of normalizing values to UTC before sending requests. In such cases the offset SHOULD only be used to calculate the equivalent UTC time before it is persisted in the system (because of known platform/language/DB interoperability issues). A UTC offset MUST NOT be used to derive anything else.

* If the business logic requires expressing the timezone of an event, it is RECOMMENDED that you capture the timezone explicitly by using a separate request/response field. You SHOULD NOT use offset to derive the timezone information. The offset alone is insufficient to accurately transform the stored UTC time back to a local time later. The reason is that a UTC offset might be same for many geographical regions and based on the time of the year there may be additional factors such as daylight savings. For example, an offset UTC-05:00 represents Eastern Standard Time during winter, Central Dayight Time during summer, and year-round offset for Panama, Columbia, and Peru.

* The timezone string MUST be per [IANA timezone database](https://www.iana.org/time-zones) (aka **Olson** database or **tzdata** or **zoneinfo** database), for example *America/Los_Angeles* for Pacific Time, or *Europe/Berlin* for Central European Time.

* When expressing [floating](https://www.w3.org/International/wiki/FloatingTime) time values that are not tied to specific time zones such as user's date of birth, expiry date, publication date etc. in requests or responses, an API SHOULD NOT associate it with a timezone. The reason is that a UTC offset changes the meaning of a floating time value. For examples, all countries with timezones west of prime meridian would consider a floating time value to be the previous day.

<h4 id="date-common-types">Date Time Common Types</h4>

The following common types MUST be used to express various date-time formats:

* [`date_time.json`](v1/schema/json/draft-04/date_time.json) SHOULD be used to express an RFC3339 `date-time`.
* [`date_no_time.json`](v1/schema/json/draft-04/date_no_time.json) SHOULD be used to express `full-date` from RFC 3339.
* [`time_nodate.json`](v1/schema/json/draft-04/time_nodate.json) SHOULD be used to express `full-time` from RFC3339.
* [`date_year_month.json`](v1/schema/json/draft-04/date_year_month.json) SHOULD be used to express a floating date that contains only the **month** and **year**. For example, card expiry date (`2016-09`).
* [`time_zone.json`](v1/schema/json/draft-04/time_zone.json) SHOULD be used for expressing timezone of a RFC3339 `date-time` or a `full-time` field.

<h1 id="http-methods-headers-status">HTTP Methods, Headers, and Statuses</h1>
Various business [capabilities](#capability) of OpsRamp are exposed through APIs as a set of [resources](#resource).  Functionality MUST not be 
duplicated across APIs; rather resources (e.g., user account, roles, etc.) are expected to be re-used as needed across use-cases.
The supported HTTP verbs and it's usage is being listed.

<h3 id="http-methods">HTTP Methods</h3>

Most services will fall easily into the standard data resource model where primary operations can be represented by the acronym CRUD (Create, Read, Update, and Delete). These map very well to standard HTTP verbs.

|HTTP Method|Description| Is Idempotent |
|---|---|---|
| `GET`| To _retrieve_ a resource. | True  |
| `POST`| To _create_ a resource, or to _execute_ a complex operation on a resource. | **False**  |
| `PUT`| To _update_ a resource. | True |
| `DELETE`| To _delete_ a resource. | True |
| `PATCH`| To perform a _partial update_ to a resource. | **False**  |


The actual operation invoked MUST match the HTTP method semantics as defined in the table above.

* The **`GET`** method MUST NOT have side effects. It MUST NOT change the state of an underlying resource.
* **`POST`**: method SHOULD be used to create a new resource in a collection.
    * **Example:** To create an alert over a resource, `POST https://api.foo.com/v1/alert`
    * Idempotency semantics: If this is a subsequent execution of the same invocation (including the [`Foo-Request-Id`](#http-custom-headers) header) and the resource was already created, then the request SHOULD be idempotent.
* The **`POST`** method SHOULD be used to create a new sub-resource and establish its relationship with the main resource.
    * **Example:** To  attach an alert on incident with ID 12345: `POST
     https://api.foo.com/v1/alerts/alertId/incidents/12345/attach`
* The **`PUT`** method SHOULD be used to update resource attributes or to establish a relationship from a resource to an existing sub-resource; it updates the main resource with a reference to the sub-resource.

**Note:** Not all resources will support all methods, but all resources using these methods must conform to their usage.

The recommended usage of the HTTP’s POST method for each of the four resource archetypes:

|     | Document | Collection            | Store | Controller          |
| ----| -------- | --------------------- | ----- | ------------------- |
| POST| _error_  | Create a new resource |_error_| Execute the command | 


<h2 id="processing">Processing</h2>

It is assumed throughout these guidelines that
request bodies and response bodies MUST be sent using [JavaScript Object Notation (JSON). JSON is a light-weight data representation for an object composed of unordered key-value pairs. JSON can represent four primitive types (strings, numbers, booleans, and null) and two structured types (objects and arrays). When processing an API method call, the following guidelines SHOULD be followed.

<h3 id="data-model">Data Model</h3>

The data model for representation MUST conform to the JSON Data Interchange Format as described in [RFC 7159](https://tools.ietf.org/html/rfc7159).


<h3 id="serialization">Serialization</h3>

*  Resource endpoints MUST support `application/json` as content type.
* If an `Accept` header is sent and `application/json` is not an acceptable response, a `406 Not Acceptable` error MUST be returned.

<h3 id="strictness">Input and Output Strictness</h3>

APIs MUST be strict in the information they produce, and they SHOULD be strict in what they consume as well.

Since we are dealing with programming interfaces, we need to avoid guessing the meaning of what is being sent to us as much as possible. Given that integration is typically a one-time task for a developer and we provide good documentation, we need to be strict with using the data that is being received. [Postel's law](https://en.wikipedia.org/wiki/Robustness_principle) must be weighed against the many dangers of permissive parsing.



<h2 id="http-headers">HTTP Headers</h2>

The purpose of HTTP headers is to provide metadata information about the body or the sender of the message in a uniform, standardized, and isolated way. HTTP header names are NOT case sensitive.

* HTTP headers SHOULD only be used for the purpose of handling cross-cutting concerns.
* API implementations SHOULD NOT introduce or depend on headers.
* Headers MUST NOT include API or domain specific values.
* If available, HTTP standard headers MUST be used instead of creating a custom header.

<h3 id="http-standard-headers">HTTP Standard Request Headers</h3>

HTTP defines a set of standard headers, some of which provide information about a requested resource. Other  headers indicate something about the representation carried by the message. And, a few headers serve as directives to 
control intermediary caches.
 
All header values must follow the syntax rules set forth in the specification where the header field is defined. 
Many HTTP headers are defined in [RFC 7231](https://tools.ietf.org/html/rfc7231), however a complete list of approved 
headers can be found in the [IANA Header Registry](http://www.iana.org/assignments/message-headers/message-headers.xhtml).

Below is a table of request headers that should be used by ATO RESTful API services. Using these headers is not mandated, 
but if used they must be used consistently.

| Header          | Type          | Description                                                                 |
| --------------- | ------------- | --------------------------------------------------------------------------- | 
| Authorization   | String        | Authorisation header for the request.                                       |
| Date            | Date          | Timestamp of the request, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time format. |
| Accept          | Content type  | The requested content type for the response e.g., "application/json".       |
| Accept-Encoding | Gzip, deflate | REST endpoints should support GZIP and DEFLATE encoding.                    |
| Accept-Language | "en", etc.    | Specifies the preferred language for the response.                          |
| Accept-Charset  | "UTF-8"       | The default is UTF-8.                                                       |
| Content-Type    | Content type  | The [mime](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) type of request body (POST/PUT/PATCH). |

<h3 id="http-custom-headers">HTTP Custom Headers</h3>

Following are some custom headers used in these guidelines. These are not part of the HTTP specifications.

| HTTP Header Name | Description |
|:-------------|------------|
| `Foo-Request-Id`    | API consumers MAY choose to send this header with a unique ID identifying the request header for tracking purpose. <br/><br/>Such a header can be used internally for logging and tracking purpose too. It is RECOMMENDED to send this header back as a response header if response is synchronous or as request header of a webhook as applicable. |

<h3 id="http-response-headers">HTTP Standard Response Headers</h3>

Services should return the following response headers, where required.

| Header             | Type          | Description                                                                 |
| ------------------ | ------------- | --------------------------------------------------------------------------- | 
| Date               | Date          | Timestamp of the response, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time format. |
| Content-Type       | Content type  | The [mime](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) type of response body. |
| Content-Encoding   | Gzip, deflate | GZIP or DEFLATE, as appropriate.                                            |
| Preference-Applied | return=minimal, return=representation  | Whether a preference indicated in the Prefer request header was applied. |


**Note:** [RFC 5322](https://tools.ietf.org/html/rfc5322#section-3.3) is a profile of [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). 

<h2 id="response-formats">Standard Response Formats</h2>

If an organisation wants to provide the best possible experience for developers, then they must provide data in the 
formats developers are accustomed to using. For example, when a mobile or other low-bandwidth client is involved, 
developers expect JSON message payloads.

Guidelines:
* Services should provide JSON as the default encoding.
* Use camelCase for field names.
* JSON must be well-formed.
* Use [JSON Schema](https://spacetelescope.github.io/understanding-json-schema/index.html) to validate the structure of JSON data.
* XML and other formats may optionally be used for resource representation.

<h2 id="Pagination">Pagination</h2>

Pagination should be supported using the Link header introduced by [RFC 5988](https://tools.ietf.org/html/rfc5988#page-6).
An API that uses the Link header can return a set of ready-made links so the API consumer doesn't have to construct 
links themselves. This is especially important when pagination is [cursor](https://developers.facebook.com/docs/graph-api/using-graph-api/#paging) based. 


<h2 id="http-status-codes">HTTP Status Codes</h2>

RESTful services use HTTP status codes to specify the outcomes of HTTP method execution. HTTP protocol specifies the outcome of a request execution using an integer and a message. The number is known as the _status code_ and the message as the _reason phrase_. The reason phrase is a human readable message used to clarify the outcome of the response. HTTP protocol categorizes status codes in ranges.

<h3 id="status-code-ranges">Status Code Ranges</h3>

When responding to API requests, the following status code ranges MUST be used.

|Range|Meaning|
|---|---|
|`2xx`|Successful execution. It is possible for a method execution to succeed in several ways. This status code specifies which way it succeeded.|
|`4xx`|Usually these are problems with the request, the data in the request, invalid authentication or authorization, etc. In most cases the client can modify their request and resubmit.|
| `5xx`| Server error: The server was not able to execute the method due to site outage or software defect. 5xx range status codes SHOULD NOT be utilized for validation or logical error handling. |

<h3 id="status-reporting">Status Reporting</h3>

Success and failure apply to the whole operation not just to the SOA framework portion or to the business logic portion of code execution.

Following are the guidelines for status codes and reason phrases.

* Success MUST be reported with a status code in the `2xx` range.
* HTTP status codes in the `2xx` range MUST be returned only if the complete code execution path is successful. This includes any container/SOA framework code as well as the business logic code execution of the method.
* Failures MUST be reported in the `4xx` or `5xx` range. This is true for both system errors and application errors.
* There MUST be a consistent, JSON-formatted error response in the body as defined by the [`error.json`][TBD] schema. This schema is used to qualify the kind of error. Please refer to [Error Handling](#error-handling) guidelines for more details.
* A server returning a status code in the `4xx` or `5xx` range MUST return the `error.json` response body.
* A server returning a status code in the `2xx` range MUST NOT return response following `error.json`, or any kind of error code, as part of the response body.
* For client errors in the `4xx` code range, the reason phrase SHOULD provide enough information for the client to be able to determine what caused the error and how to fix it.
* For server errors in the `5xx` code range, the reason phrase and an error response following [`error.json`][TBD] SHOULD limit the amount of information to avoid exposing internal service implementation details to clients. This is true for both external facing and internal APIs. Service developers should use logging and tracking utilities to provide additional information.

<h3 id="allowed-status-codes">Allowed Status Codes List</h3>

All REST APIs MUST use only the following status codes. Status codes in **`BOLD`** SHOULD be used by API developers. The rest are primarily intended for web-services framework developers reporting framework-level errors related to security, content negotiation, etc.

* APIs MUST NOT return a status code that is not defined in this table.
* APIs MAY return only some of status codes defined in this table.

| Status Code | Description |
|-------------|-------------|
| **`200 OK`** | Generic successful execution. |
| **`201 Created`** | Used as a response to `POST` method execution to indicate successful creation of a resource. If the resource was already created (by a previous execution of the same method, for example), then the server should return status code `200 OK`. |
| **`202 Accepted`** | Used for asynchronous method execution to specify the server has accepted the request and will execute it at a later time. Possible explanation of such error codes<TBD>. |
| **`204 No Content`** | The server has successfully executed the method, but there is no entity body to return.|
| **`400 Bad Request`** | The request could not be understood by the server. Use this status code to specify:<br/> <ul><li>The data as part of the payload cannot be converted to the underlying data type.</li><li>The data is not in the expected data format.</li><li>Required field is not available.</li><li>Simple data validation type of error.</li></ul>|
| `401 Unauthorized` | The request requires authentication and none was provided. Note the difference between this and `403 Forbidden`. |
| **`403 Forbidden`** | The client is not authorized to access the resource, although it may have valid credentials. API could use this code in case business level authorization fails. For example, accound holder does not have enough funds. |
| **`404 Not Found`** | The server has not found anything matching the request URI. This either means that the URI is incorrect or the resource is not available. For example, it may be that no data exists in the database at that key. |
| `405 Method Not Allowed` | The server has not implemented the requested HTTP method. This is typically default behavior for API frameworks.
| `406 Not Acceptable` | The server MUST return this status code when it cannot return the payload of the response using the media type requested by the client. For example, if the client sends an `Accept: application/xml` header, and the API can only generate `application/json`, the server MUST return `406`. |
| `415 Unsupported Media Type` | The server MUST return this status code when the media type of the request's payload cannot be processed. For example, if the client sends a `Content-Type: application/xml` header, but the API can only accept `application/json`, the server MUST return `415`. |
| **`422 Unprocessable Entity`** | The requested action cannot be performed and may require interaction with APIs or processes outside of the current request. This is distinct from a 500 response in that there are no systemic problems limiting the API from performing the request. |
| `429 Too Many Requests` | The server must return this status code if the rate limit for the user, the application, or the token has exceeded a predefined value. Defined in Additional HTTP Status Codes [RFC 6585](https://tools.ietf.org/html/rfc6585). |
| **`500 Internal Server Error`** | This is either a system or application error, and generally indicates that although the client appeared to provide a correct request, something unexpected has gone wrong on the server. A `500` response indicates a server-side software defect or site outage. `500` SHOULD NOT be utilized for client validation or logic error handling. |
| `503 Service Unavailable` | The server is unable to handle the request for a service due to temporary maintenance. |

<h3 id="mapping">HTTP Method to Status Code Mapping</h3>

For each HTTP method, API developers SHOULD use only status codes marked as "X"  in this table. If an API needs to return any of the status codes marked with an **`X`**, then the use case SHOULD be reviewed as part of API design review process and maturity level assessment. Most of these status codes are used to support very rare use cases.

| Status Code | 200 Success | 201 Created |202 Accepted | 204 No Content | 400 Bad Request |  404 Not Found |422 Unprocessable Entity | 500 Internal Server Error |
|-------------|:------------|:------------|:------------|:---------------|:----------------|:---------------|:------------------------|:--------------------------|
| `GET`       | X         |               |               |                  | X             | X              | **`X`**           | X                       |
| `POST`      | X         | X         | **`X`**           |                 |  X             | **`X`**              | **`X`**               | X                       |
| `PUT`       | X             |               | **`X`**           | X           | X             | X              | **`X`**           | X                       |
| `PATCH`     | X             |               |               | X           | X            | X              | **`X`**           | X                       |
| `DELETE`    | X             |               |               | X           | X             | X              | **`X`**               | X                       |



* `GET`: The purpose of the `GET` method is to retrieve a resource. On success, a status code `200` and a response with the content of the resource is expected. In cases where resource collections are empty (0 items in `/v1/namespace/resources`), `200` is the appropriate status (resource will contain an empty `items` array). If a resource item is 'soft deleted' in the underlying data, `200` is not appropriate (`404` is correct) unless the 'DELETED' status is intended to be exposed.

* `POST`: The primary purpose of `POST` is to create a resource. If the resource did not exist and was created as part of the execution, then a status code `201` SHOULD be returned.
    * It is expected that on a successful execution, a reference to the resource created (in the form of a link or resource identifier) is returned in the response body.
    * Idempotency semantics: If this is a subsequent execution of the same invocation (including the [`Foo-Request-Id`](#http-custom-headers) header) and the resource was already created, then a status code of `200` SHOULD be returned. For more details on idempotency in APIs, refer to [idempotency](#idempotency).
	* If a sub-resource is utilized ('controller' or data resource), and the primary resource identifier is non-existent, `404` is an appropriate response.

* `POST` can also be used while utilizing the [controller](#controller), `200` is the appropriate status code.

* `PUT`: This method SHOULD return status code `204` as there is no need to return any content in most cases as the request is to update a resource and it was successfully updated. The information from the request should not be echoed back.
    * In rare cases, server generated values may need to be provided in the response, to optimize client flow (if the client necessarily has to perform a `GET` after `PUT`). In these cases, `200` and a response body are appropriate.

* `PATCH`: This method should follow the same status/response semantics as `PUT`, `204` status and no response body.
	* `200` + response body should be avoided at all costs, as `PATCH` performs partial updates, meaning multiple calls per resource is normal. As such, responding with the entire resource can result in large bandwidth usage, especially for bandwidth-sensitive mobile clients.

* `DELETE`: This method SHOULD return status code `204` as there is no need to return any content in most cases as the request is to delete a resource and it was successfully deleted.

    * As the `DELETE` method MUST be idempotent as well, it SHOULD still return `204`, even if the resource was already deleted. Usually the API consumer does not care if the resource was deleted as part of this operation, or before. This is also the reason why `204` instead of `404` should be returned.

<h1 id="error-handling">Error Handling</h1>

As per HTTP specifications, the outcome of a request execution could be specifiedusing an integer and a message. The number is known as the _status code_ and the message as the _reason phrase_. The reason phrase is a human readable message used to clarify the outcome of the response. The HTTP status codes in the `4xx` range indicate client-side errors (validation or logic errors), while those in the `5xx` range indicate server-side errors (usually defect or outage). However, these status codes and human readable reason phrase are not sufficient to convey enough information about an error in a machine-readable manner. To resolve an error, non-human consumers of RESTful APIs need additional help.

Therefore, APIs MUST return a JSON error representation that conforms to the a defined schema TBD (error.json)

<h2 id="error-schema">Error Schema</h2>


An error response following `error.json` as schema MUST include the following fields:

* `name`: A human-readable, unique name for the error. It is recommended that this value would be retrieved from the error catalog (TBD) before sending the error response.
* `details`: An array that contains individual instance(s) of the error with specifics such as the following. This field is required for client side errors (`4xx`).
	* `field`: The field in error if in body, else name of the path parameter or query parameter.
	* `value`: Value of the field in error.
	* `issue`: Reason for error. It is recommended that this value would be retrieved from the error catalog [TBD] before sending the error response.
	* `location`: The location of the field in the error, either `query`, `path`, or `body`. If this field is not present, the default value is `body`.
* `trace_id`: A unique error identifier generated on the server-side and logged for correlation purposes.
* `message`: A human-readable message, describing the error. This message MUST be a description of the problem NOT a suggestion about how to fix it. 
* It is recommended that this value would be retrieved from the error catalog (TBD) before sending the error response.


<h3 id="input-errors">Input Validation Errors</h3>

In validating requests, there are a variety of concerns that should be addressed in the following order:

|Request Validation Issue | HTTP status code|
|------------- | -------------|
|Not well-formed JSON. | `400 Bad Request`|
|Contains validation errors that the client can change. | `400 Bad Request`|
|Cannot be executed due to factors outside of the request body. The request was well-formed but was unable to be followed due to semantic errors. | `422 Unprocessable Entity`|

<h2 id="error-samples">Error Samples</h2>

This section provides some samples to describe usage of error responsed in various scenarios.

<h4 id="sampleresponse-invalid">Validation Error Response - Single Field</h4>

The following sample shows a validation error of type `VALIDATION_ERROR` in one field. Because this is a client error, a `400 Bad Request` HTTP status code should be returned.

```

{
   "name":"VALIDATION_ERROR",
   "details":[
      {
         "field":"#/alerts/alert_id",
         "issue":"Required field is missing",
         "location":"body"
      }
   ],
   "trace_id":"123456789",
   "message":"Invalid data provided",
   "information_link":"http://developer.foo.com/apidoc/blah#VALIDATION_ERROR"
}
```

<h4 id="sampleresponse-multi">Validation Error Response - Multiple Fields</h4>

The following sample shows a validation error of the same  type, `VALIDATION_ERROR`, in two fields. Note that `details` is an array listing all the instances in the error. Because both these are a client errors, a `400 Bad Request` HTTP status code should be returned.

```

{
   "name": "VALIDATION_ERROR",
   "details": [
      {
         "field": "/incidents/incident_id",
         "issue": "Required field is missing",
         "location": "body"
      },
      {
         "field": "/timezones/code",
         "value": "XYZ",
         "issue": "Current code is invalid",
         "location": "body"
      }
   ],
   "trace_id": "123456789",
   "message": "Invalid data provided",
   "information_link": "http://developer.foo.com/apidoc/blah#VALIDATION_ERROR"
}

```

<h4 id="sampleresponse-bulk">Validation Error Response - Bulk </h4>

For heterogenous types of client-side errors shown below, `OBJECT_NOT_FOUND_ERROR` and `MULTIPLE_CORE_BUNDLES`, an array named `errors` is returned. Each error instance is represented as an item in this array. Because these are client validation errors, a `400 Bad Request` HTTP status code should be returned.

```

   "errors": [
      {
         "name": "OBJECT_NOT_FOUND_ERROR",
         "trace_id": "38cdd677a83a4",
         "message": "Bundle is not found.",
         "information_link": "<link to public doc describing OBJECT_NOT_FOUND_ERROR error>",
         "details": [
            {
               "field": "/bundles/0/bundle_id",
               "value": "33333",
               "issue": "BUNDLE_NOT_FOUND",
               "location": "body"
            }
         ]
      },
      {
         "name": "MULTIPLE_CORE_BUNDLES",
         "trace_id": "52cde38284sd3",
         "message": "Multiple CORE bundles.",
         "information_link": "<link to public doc describing MULTIPLE_CORE_BUNDLES error>",
         "details": [
            {
               "field": "/bundles/5/bundle_id",
               "value": "88888",
               "issue": "MULTIPLE_CORE_BUNDLES",
               "location": "body"
            }
         ]
      }
   ]
```

<h4 id="sampleresponse-422">Semantic Validation Error Response </h4>

In cases where client input is well-formed and valid but the request action may require interaction with APIs or processes outside of this URI, an HTTP status code `422 Unprocessable Entity` should be returned.

```
{
   "name": "SYNTAX_ERRORS",
   "trace_id": "123456789",
   "message": "The alert id format is not right.",
   "information_link": "http://developer.foo.com/apidoc/blah#SYNTAX_ERRORS"
}
```

<h2 id="error-declaration">Error Declaration In API Specification</h2>

It is important that documentation generation tools  recognize errors. Following section shows  example how one  could refer `error.json` in
an API specification confirming to OpenAPI.


```
"responses": {
          "200": {
            "description": "alert found and returned.",
            "schema": {
              "$ref": "error.json"
            }
          },
          "403": {
            "description": "Unauthorized request.  This error will occur if the SecurityContext header is not provided or does not include a alert_id."
          },
          "404": {
            "description": "The requested address does not exist.",
            "schema": {
              "$ref": "v1/schema/json/draft/error.json"
            }
          },
          "default": {
            "description": "Unexpected error response.",
            "schema": {
              "$ref": "v1/schema/json/draft/error.json"
            }
          }
      }

```
<h2 id="userguide-errors">Samples with Error Scenarios in Documentation</h2>

The User Guide of an API is a document that is exposed to API consumers. In addition to linking to samples showing successful execution for invocation on various methods exposed by the API, the API developer should also provide links to samples showing error scenarios. It is equally, or perhaps more, important to show the API consumers how an API would propagate errors in a machine-readable form in order to build applications that take necessary actions to handle errors gracefully and in a meaningful manner.
In conclusion, we reiterate the message we started with that non-human consumers of RESTful APIs need more help to take necessary actions to resolve an error in a machine-readable manner. Therefore, a representation of errors following the [schema](#error-schema) described here MUST be returned by APIs for any HTTP status code that falls into the ranges of 4xx and 5xx.

<h2 id="error-catalog">Error Catalog</h2>

Error handling [guidelines](#error-handling) described earlier show how to provide error related details responses at runtime. This section explains how to catalog the errors so they can be easily consumed in service runtime and for generating documentation.

An Error Catalog is a single JSON file that contains a collection of error specifications (or error metadata) for a namespace. Each error specification includes error name, error message, issue details and related links among other things. The error catalog supports multiple locales or languages. For a specific error catalog, there should be exactly one default version, known as the top-level catalog, which could be in English for example. There should be corresponding locale-specific catalogs, one for each additional supported locale, as needed.

<h3 id="reasons-to-catalog-errors">Reasons To Catalog Errors</h3>

Following are some reasons to catalog the errors for an API:

1. To _externalize_ hard-coded error message strings from the API implementation: Developers are good at writing code but not necessarily good at writing error messages. Often error messages written by a developer make a lot of assumptions about the context and audience. It is hard to change such error messages if these are embedded in implementation code. For example, to change a message from "Add Card refused due to compliance guidelines" to `Could not add card due to failure to comply with guideline %s`, a service developer has to make the change and also redeploy the service emitting that error.
2. To _localize_ the error strings: If error related strings such as `message`, `issue`, `actions`, among other things in `error.json` are externalized, it is easy for the documentation and an internationalization team to modify and localize these without any help from service developers and without requiring redeployment of the services.
3. To keep service's implementation and the API documentation in _sync_ with regards to errors: API consumers should be able to refer with confidencethe API's documentation for errors generated by the services at runtime. This helps to reduce the cost and the time spent supporting an API and increases `adoptability` of the API.


<h2 id="error-catalog-schema">Error Catalog Schema</h2>
         TBD

<h1 id="api-versioning">API Versioning</h1>

This section describes how to version APIs. It describes API's lifecycle states, enumerates versioning policy, describes backwards compatiblity related guidelines and describes an End-Of-Life policy.

<h2 id="api-lifecycle">API Lifecycle</h2>

Following is an example of states of an API's lifecycle.

|State | Description|
|---------|------------|
|PLANNED |API has been scheduled for development. API release plans have been established.|
|BETA|API is operational and is available to selected new subscribers in production for the purposes of testing, validating, and rolling out a new API.|
|LIVE | API is operational and is available to new subscribers in production. API version is fully supported.|
|DEPRECATED | API is operational and available at runtime to existing subscribers. API version is fully supported, including bug fixes addressed in backwards compatible way. API version is not available to new subscribers.|
|RETIRED | API is unpublished from production and no longer available at runtime to any subscribers. The footprint of all deployed applications entering this state must be completely removed from production and stage environments.|

<h2 id="api-versioning-policy">API Versioning Policy</h2>

API’s are versioned products and MUST adhere to the following versioning principles.

1. API specifications MUST follow the versioning scheme where where the `v` introduces the version, the major is an ordinal starting with `1` for the first LIVE release, and minor is an ordinal starting with `0` for the first minor release of any major release.
2. Every time there is an incremental change to an API, whether or not backward compatible, the API specification MUST be versioned. This allows the change to be labeled, documented, reviewed, discussed, published and communicated.
3. API endpoints MUST only reflect the major version.
4. API specification versions reflect interface changes and MAY be separate from service implementation versioning schemes.
5. A minor API version MUST maintain backward compatibility with all previous minor versions, within the same major version.
6. A major API version MAY maintain backward compatibility with a previous major version.

For a given functionality set, there MUST be only one API version in the `LIVE` state at any given time across all major and minor versions. This ensures that subscribers always understand which versioned API product they should be using. For example, v1.2 `RETIRED`, v1.3 `DEPRECATED`, or v2.0 `LIVE`.


<h2 id="backwards-compatibility">Backwards Compatibility</h2>

APIs SHOULD be designed in a forward and extensible way to maintain compatibility and avoid duplication of resources, functionality and excessive versioning.

APIs MUST adhere to the following principles to be considered backwards compatible:

1. All changes MUST be additive.
2. All changes MUST be optional.
3. Semantics MUST NOT change.
4. Query-parameters and request body parameters MUST be unordered.
5. Additional functionality for an existing resource MUST be implemented either:
    1. As an optional extension, or
    2. As an operation on a new child resource, or
    3. By altering a request body, while still accepting all the previous, existing request variations, if an existing operation (e.g., resource creation) cannot be reasonably extended.


<h3 id="backwards-incompatible-changes">Non-exhaustive List of Backwards Incompatible Changes</h3>

<b>URIs</b>

1. A resource URI MAY support additional query parameters but they CANNOT be mandatory.
2. There MUST NOT be any change in the behavior of the API for request URIs without the newly added query parameters.
3. A new parameter with a required constraint SHALL NOT be added to a request.
4. The semantics of an existing parameter, entire representation, or resource SHOULD NOT be changed.
5. A service MUST recognize a previously valid value for a parameter and SHOULD NOT throw an error when used.
6. There MUST NOT be any change in the HTTP status codes returned by the URIs.
7. There MUST NOT be any change in the HTTP verbs (e.g. `GET`, `POST`, `PUT` or `PATCH`) supported earlier by the URI. The URI MAY however support a new HTTP verb.
8. There MUST NOT be any change in the name and type of the request or response headers of an URI. Additional headers MAY be added, provided they’re optional.

<b>APIs only support media type `application/json`. The following applies for JSON representation stability.</b>

1. An existing property in a JSON object of an API response MUST continue to be returned with same name and JSON type (number, integer, string, array, object).
2. If the value of a response field is an array, then the type of the contents in the array MUST NOT change.
3. If the value of the response field is an object, then the compatibility policy MUST apply to the JSON object as a whole.
4. New properties MAY be added to a representation any time, but it SHOULD NOT alter the meaning of an existing property.
5. New properties that are added MUST NOT be mandatory.
6. Mandatory fields are guaranteed to be present in the response.
7. For primitive types, unless there is a constraint described in the API documentation (e.g. length of the string, possible values for an ENUM type), clients MUST not assume that the values are constrained in any way.
8. If the property of an object is a URI, then it MUST have the same stability mentioned as URIs.
9. For ENUM types, there MUST NOT be any change in already supported enumerated values or meaning of these values.


<h2 id="eol-policy">End of Life Policy</h2>

The End-of-Life (EOL) policy regulates how API versions move from the `LIVE` to the `RETIRED` state. It is designed to ensure a consistent and reasonable transition period for API customers who need to migrate from the old to the new API version while enabling a healthy process to retire technical debt.

<b>Minor API Version EOL</b>

Per versioning policy, minor API versions MUST be backwards compatible with preceding minor versions within the same major version. Thus, minor API versions are `RETIRED` immediately after a newer minor version of the same major version becomes `LIVE`. This change should have no impact on existing subscribers so there is no need to transition through a `DEPRECATED` state to facilitate client migration.

<b>Major API Version EOL</b>

Per versioning policy, major API versions MAY be backwards compatible with preceding major versions. As such, the following rules apply when retiring a major API version.

1. A major API version MUST NOT be in the `DEPRECATED` state until a replacement service is `LIVE` that provides a clear customer migration path for all functionality that will be retained moving forward. This SHOULD include documentation and, as appropriate, migration tools and sample code that provide customers what they need to make a clean migration.
2. The deprecated API version MUST be in the `DEPRECATED` state for a minimum period of time to give client customers adequate notice to migrate. Deprecation of API versions with external clients SHOULD be considered on a case-by-case basis and may require additional deprecation time and/or constraints to minimize impact to the business.
6. If a versioned API in `LIVE` or `DEPRECATED` state has no clients, it MAY move to the `RETIRED` state immediately.

<h3 id="eol-policy-replacement">End of Life Policy – Replacement Major API Version Introduction</h3>

Since a new major API version that results in deprecation of a pre-existing API version is a significant business investment decision, API owners MUST justify the new major version before beginning significant design and development work.  API owners SHOULD explore all possible alternatives to introducing a new major API version with the objective of minimizing the impact on customers before deciding to introduce a new version. Justification SHOULD include the following:

Business Case

1. Customer value delivered by new version that is not possible with existing version.
2. Cost-benefit analysis of deprecated version versus the new version.
3. Explanation of alternatives to introducing an new major version and why those were not chosen.
4. If a backwards incompatible change is required to address a critical security issue, items 1 and 2 (above) are not required.

API Design

1. A domain model of all resources in the new API version and how they compare to the domain model of the previous major API version.
2. Description of APIs operations and use cases they implement.
3. Definition of service level objectives for performance and availability that are equal or better to the major API version to be deprecated.

Migration Strategy

1. Number of existing customers impacted; internal, external, and partners.
2. Communication and support plan to inform existing customers of new version, value, and migration path.




<h1 id="deprecation">Deprecation</h1>

This document describes a solution to deprecate parts of an API as the API evolves. It is an extension to the API Versioning Policy.


<h3 id="deprecation-terms-used">Terms Used</h3>

The term *`API Element`* is used throughout this document to refer to the *things* that could be deprecated in an API. Examples of an `API Element` are: an endpoint, a query parameter, a path parameter, a property within a JSON Object schema, JSON Object schema of a type or a custom HTTP header among other things.

The term *`old API`* is used to indicate existing minor or major version of your API or an existing different API that your API supersedes.

The term *`new API`* is used to indicate a new minor or major version of your API or a new different API that supersedes the `old API`.

*`API definition`* is in the form of specification of an interface of a service following the [OpenAPI][11]. API's definition could be found in `swagger.json`.

<h2 id="deprecation-background">Background</h2>

When defining your API, you must make a lot of material decisions that have long lasting implications. The objective is to make a long-lived, durable, and reusable API. You are trying to get it "right". Practically speaking, however, you are not going to succeed every time. New requirements come in. Your understanding of the problem changes. What probably looked like a good decision at the time, may now limit your ability to elegantly extend your API to satisfy your new understanding. Lightweight decisions made in the past now seem somewhat heavy as the implications come into focus. Maintaining backward compatibility is a persistent challenge.

One option is to create a new major version of your API. This allows you to leave past decisions behind and start fresh. Unfortunately, it also means that all of your clients now need to migrate to new endpoints for any of the new work to deliver customer value. This is hard. Many clients will not move without good incentives. There is a lot of overhead in managing the customer migration. You also need to support two sets of interfaces for quite some time. The other consideration is that your API product may have multiple endpoints, but the breaking changes that you want to make only affect one. Making your customers migrate their applications for all the API endpoints just so you can “fix” one small part is a pretty heavyweight and expensive change. While pure and simple from a philosophical and engineering point of view, it is often unjustifiable from an ROI standpoint. The goal of this guideline is to find a middle ground that provides a more practical path forward when minor changes are needed, but which is still consistent, in spirit, with the [API Versioning Policy](#api-versioning-policy).

<h2 id="deprecation-requirements">Requirements</h2>

Here are the requirements for deprecation.

1. An API developer should be able to deprecate an `API Element` in a minor version of an API.
2. An API specification MUST highlight one or more deprecated elements of the API so the API consumers are aware.
3. An API server MUST inform client app(s) regarding deprecated elements present in request and/or response at runtime so that tools can recognize this, log warnings and highlight the usage of deprecated elements as needed.
4. Deprecated `API Elements` must remain supported for the life of the major version or until customers are no longer using them (the means to determine this are left to the discretion of the API owner since it's their customers who will ultimately be impacted).

<h2 id="deprecation-solution">Solution</h2>

The following describes how to address the requirements listed above. The solution involves addressing documentation related requirement using 
an atttribute provided by OpenAPI. An optional property named `deprecated = true` is used to mark an `API Element` as deprecated in the definition of the API.

<h3 id="deprecation-attribute">Attribute: deprecated</h4>

`deprecated = true` can be used to deprecate any kind of `API Element`. The attribue should be used inline precisely where the `API Element` is defined.
It is expected that the API documentation would highlight deprecated `API Elements` for  by `deprecated = true` in the API specification in accordance 
with [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.1.0.md) distinctly  and at the appropriate granularity.

<h4 id="deprecation-schema-annotation">Deprecated schema examples</h4>

We have provided specific YAML object types to use for deprecation of specific `API Elements`. This section lists schema for these types. The intent in providing schema for specific application of `deprecated` is to make it easy for API developers to annotate the `API definition` and for API tool(s) to highlight each deprecated `API Element` with appropriate details.

##### Common Schema Elements

```
openapi: 3.1.0
...
components:
  schemas:
    Service:
      type: object
      properties:
        location:
          type: string
          description: Location of the service
          example: '400 Street name, City State postcode, Country'
          deprecated: true    # <--------- Location attribute is getting deprecated

```

```
    /resource/findByTags:
      get:
        deprecated: true  # <--------- Endpoint is getting deprecated
```
<h1 id="references">References</h1>
