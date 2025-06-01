[Back](/)

# Patterns for API Design: Simplifying Integration with Loosely-Coupled Message Exchanges
Olaf Zimmermann, Mirko Stocker, Daniel Lübke, Uwe Zdun, Cesare Pautasso (2023)

---

## TL;DR

## Notes
> In this book, we are not concerned with microservices infrastructures but with service rightsizing on the API level (in terms of endpoint granularity and operation/data coupling). 13

> Our focus in this book is on remote APIs connecting systems and their parts. 30

APIs as products: “They are supposed to have a dedicated business owner, a governance structure, a support system, and a roadmap.” 13

Finding the right balance:

- Exposing data vs hiding implementation
- Independent evolution vs “decrease the cost of bringing two systems together” 16
  - The “dual connector-separator role” 19
- General endpoints vs one per client
  - Different clients want different things, and they keep changing their minds
- Fine-grained vs coarse-grained endpoints
  - For IoT, they say it’s better to batch transmissions (presumably because they have very small payloads and it’s mostly overhead)
- Few operations with lots of data vs chatty interactions with little data
- Correct vs current data
- Stable vs fast-changing contracts
  - “… continuous evolution fot eh API design has a negativy impact on the stability of the API design. Important design considerations that are hard to get right in the first place are involved, on reason being the uncertainty about (future) uses that API providers and API clients often experience.” 77-8
- Nested vs flat structures
- Public vs private
- Data- vs action-oriented
  - Small, granular, chatty interactions give clients control; “but action-oriented capabilities can promote qualities such as consistency, compatibility, and evolvability.” 60
  - “A truly stateless Processing Resource can be hard to achieve in reality. API security and request/response data privacy can lead to the need for maintaining state…” 61
  - “Highly data-centric approaches tend to lead to create, read, update, delete (CRUD) APIs, which can negatively affect coupling.” 61

Measure along: performance, scalability, reliability, security, manageability, ease of use, … … …

Once an API is used, “it will become increasingly expensive to apply corrections and improvements and impossible to remove features without breaking some clients.” 18

Types of conversation and message (this seems arbitrary and incomplete),

1. Request/response = command message
2. One-way = document
3. Notification = event
4. Callbacks (single request, multiple replies)

💡 Is it a good rule of thumb to first try to design a processing resource, then if it must be an information holder resource, abstract away the database (e.g., don’t expose debt status) and if that’s not easy/obvious, don’t worry?

Patterns

- Visibility
  - **Public**: anyone can access
  - **Community**: deploy in an access restricted location, e.g., intranet
  - **Solution-internal**: restricted access
- Integration types
  - **Front-end integration**: front-end calls an API)
  - **Back-end integration**: back-end services use each other’s APIs
- Documentation
  - **Description**: describe request/response messages, errors, side-effects, sequences, conditions, invariants…
- Roles
  - **Processing resource**: action-oriented (process commands, activities)
  - **Information holder resource**: data-oriented (storage)
    - **Operational data holder**: frequent operations on “data that is rather short-lived, changes often during dailiy business operations, and has many ongoing relations” 63
    - **Master data holder**: operations on long-living and frequently referenced data; often shared by organisational units
    - **Reference data holder**: immutable data that can be cached
- Responsibilities
  - **State creation operation**: write-only
  - **Retrieval operation**: read-only
  - **State transition operation**: combine input and state to update/delete
  - **Computation function**: stateless, no side-effects
- Message formats
  - **Atomic parameter**
  - **Atomic parameter list**
  - **Parameter tree**
  - **Parameter forest**
  - “Note that in addition to the more conceptual considerations called out in this section, many technology decisions also have to be made. This includes supported communication protocols (HTTP, HTTPS, AMQP, FTP, SMTP, and so on) and message exchange formats such as JSON, SOAP or plain XML, ASN.1, Protocol Buffers, Apache Avro schemas, or Apache Thrift,. API query languages such as GraphQL can also be introduced.” 78

