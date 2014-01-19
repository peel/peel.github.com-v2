---
layout: post
title: Grammar Nazi? Best practices for REST API design
category: articles 
date: '2012-08-24 19:53:55'
---

A few best practices regarding designing REST APIs I try to follow when re/designing web grammar.
<p class="alertBox notice">Notice &#8211; Will update w/ repo code.</p>


###Fundamentals

#### Idempotency
A service should return the same value upon every invocation w/ the same payload; use tokens (i.e. in REST POST URI)

#### Self-documentation
Service naming, method naming should be self-documenting. Follow REST convention when designing

#### RESTfulness
Keep POST params in body, asynchronously fetch and return media and follow the rule:
    
    Create   POST     /Customers
    Fetch    GET      /Customers/ID123
    Modify   PUT/POST /Customers/ID123
    Remove   DELETE   /Customers/ID123
    
#### Versioning
 To provide flexibility keep version number in RESTful service URIs, i.e.
 `GET /api/v1/Resource`
 
#### Statefulness
As keeping state is non trivial it is generally a good practice to avoid making a user to keep state. Instead if possible make your service stateful
    
#### Predictability
Once again a naming convention; keep names understandable;
    
#### Responsiveness
When a response is available immediately, use a synchronous response.  
Otherwise, consider an asynchronous webhook rather than polling.

<!--- more start -->
---

###RESTful API grammar design
#### Use nouns not verbs
Keep verbs out of your base URLs. As in JAX-WS':

    /getCustomer
    /getCustomerData

use two base URLs per resource

    /customers
    /customers/1234
    ...
    
use HTTP verbs to operate on data

    | Resource        | POST           | GET               | PUT           | DELETE        |
    | --------------- | -------------- | ----------------- | ------------- | ------------- |
    | /customers      | new customer   | list customers    | bulk update   | delete all    |
    | /customers/123  | invalid        | get 1234's data   | update/fault  | delete 1234   |                       

#### Plurality and concreteness
make API's nouns plural and concrete

####Simplify associations
Customers have relationship w/ PBAs who have relationships w/ 
To avoid complex structures like ie.

    GET    /customers/1234/accounts/1234/owner/1234
it is generaly a good practice to hide the comples structures behind the '?'. It makes simple for developers to use the base URL w/ additional attibutes:
    
    GET    /customers?status=Gold

#### Status codes and error handling
It is generally a good practice to keep a limited subset of status codes and a verbose description of the status along w/ a link to a detailed description (Status enum does a great job here). When designing from ground up start by using 3 basic status codes: 200, 400, 500 to eventually grow your list to 5-7 codes.

#### API Versioning
API version (in URL) should be bumped up when the change modifies expected response, minor version updates should be put in a header.
One version back support is obligatory.

#### Pagination
Use count and start for pagination i.e.
    
    GET    /customers?start=10&count=10
also it is valueable to keep a default values for calls w/o parameters.

#### Non-resource scenarios
When designing a non-resource scenario it is a recommended approach to use verbs instead of nouns, i.e.
    
    GET    /convert?from=PLN&to=EUR&amount=1000

#### Polyformat services
As of now the established format is a verbose XML. The XML is being used in such components as Oracle Service Bus and Oracle BPM. Thus it is recommended to use this format as default. For more advanced, not based on the legacy XML format, applications it is recommended to provide a terse JSON representation. 
JAXB supports JSON via Jettision project. JSON schema is also available.
It is recommended to let JSON data representation to be accessed via
`GET    /customers.json`
The above convention is based on conneg w/ JAX-RS that treats the `.{type}` as acceptable format.  
For atrubutes naming a JavaScript conventions should be followed:

- CamelCase
- Upper/lowercase depending on object type

#### Search
For global search, use:
`GET     /search?q=customerId`

For scoped search, use:
`GET     /customers/1234/accounts?=debit`

#### Provide exemplary client code in repository
When using plain old JAX-WS SOAP web services we tend to generate clients for other apps to consume and place them in our local Nexus repository. In case of REST services you need to handle client-generation by yourself /even though RESTeasy does have a feature of code generation/. That's why we always place a sample code in the repo along with the service code.

#### API facade design pattern
API facade is a good design pattern to avoid unnecessary complexity on API consumer side.
<!-- more end -->
