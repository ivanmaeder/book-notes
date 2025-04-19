[Back](/)

# Software Architecture: The Hard Parts: Modern Trade-Off Analyses for Distributed Architectures
Neal Ford, Mark Richards, Pramod Sadalage, Zhamak Dehghani (2021)

---

## TL;DR
- Don’t compare all pros/cons: narrow down the list depending on your context, goals
- Try to become the objective arbiter of trade-offs
- Breadth of knowledge is more important than depth
- Fitness functions: “architecture as code” to measure and regression test architectural designs
    - Keep services roughly the same size
    - Flag too many dependencies
    - Flag disallowed dependencies
    - Flag new components for review
- Service integrators/disingetrators: size, cohesion, volatility (rates of change), scalability/throughput, fault tolerance, security, extensibility, database transactions, shared data, shared code
- Data ownership: start with who writes to the table; failing that,
    - If many writers, create a service that writes to the table
    - Otherwise, split the table
    - Or make a choice—assign it to a service
    - Or combine the services
- Distributed data access
    - Service A asks B for the data
    - Replicate column between databases
    - Replicate data via a distributed cache (read-only)
- Choreography and complex workflows that need to recover from errors don’t mix well

## Podcast interview with Neal Ford
[#120 - Software Architecture: From Fundamentals to the Hard Parts - Neal Ford](https://podcasts.apple.com/us/podcast/tech-lead-journal/id1523421550?i=1000599335450), Tech Lead Journal (Henry Suryawirawan)

Architecture:

- You can’t search for the answers online
- There are always trade-offs; it’s about finding the least worse option, based on business goals/priorities
- _Why_ is more important than _how_; document trade-offs

Skills:

- Breadth is more important than depth
  - Knowing about 5 different ways to solve a problem ✅
  - Knowing 1 specific solution really well ❌
- Conflict resolution

> Your database schema is one of your most intense implementation details of your architecture, and the more you let implementation details leak out into your architecture, the more brittle your architecture becomes. Meaning you make one change somewhere and it breaks a whole bunch of stuff.

## DDD
- **Domain**: broadest concept; your application or business may model a single domain
- **Subdomain**: part of a domain; examples might be: payments, customer management. Types:
    - **Core**: what makes your company unique, or its competitive advantage; invest the most here (deep domain expertise, frequent evolution or highly maintainable, high quality)
    - **Supporting**: necessary but not differentiator, not strategic (good enough, consider outsourcing or using off-the-shelf)
    - **Generic**: commoditised; use existing solutions (e.g., auth, logging, billing, CRM…)
- **Bounded context**: same language applies consistently; might cover a single subdomain, part of a subdomain or several subdomains (our src/domains/ folder is for bounded contexts). They contain:
    - **Entities**: have a distinct identity over time, even if their data changes (e.g., a customer)
    - **Value objects**: no identity, just a set of attributes; typically immutable—you replace, not modify (e.g., address: two identical addresses are treated as the same thing)
    - **Aggregates**: groups of entities and value objects; they have a root entity (“aggregate root”). E.g., an order (root) with items (child entities)
    - **Services**: operations that don’t naturally belong to an entity or value object, but they operate on entities and value objects
        - In DDD, domain services aren’t exactly the API layer: domain services are “pure” and have a thin application layer on top of them where the HTTP, auth, etc concerns live
        - Confusingly, “domain services” don’t mean “services for the whole domain”—domain services live inside a bounded context
    - **Repositories**: abstractions for accessing aggregates (e.g., `OrderRepository.save()` or `findById()`). Typically one repository per aggregate root

## Notes
#### Chapter 1. What happens when there are no “best practices”?
> Change is also difficult and risky in this large monolith. Whenever a change is made, it usually takes too long and something else usually breaks… resulting in all application functionality not being available anywhere from five minutes to two hours while the problem is identified and the application is restarted. 17

🤔 We don’t usually break other things, because of tests? Or failover? Or…?

Architectural decision records should provide justifications, describe trade-offs and list alternatives.

### Part 1. Pulling things apart
#### Chapter 2. Discerning coupling in software architecture

<!--
Definitions:

- **Static coupling**. Operational dependencies (database, operating system, frameworks, packages, Docker, event brokers…)
    - A service can in theory run with just the static coupling, but it likely needs the dynamic coupling at runtime to do anything useful
    - “Is this dependent of the architecture necessary to bootstrap this service?” 33
    - “… does not consider other quanta whose only coupling point is workflow communication with this quantum.” 401
- **Dynamic coupling**. How services communicate at runtime—communication dependencies (sync/async…)
    - Imagine two services with different scaling needs. Calls can’t be sync because one will be hung up waiting for the other, and the other may be swamped with calls
    - More on this later: sync/async communication, atomic/eventual consistency, orchestration/choreography
- **Architecture quantum**. Independently deployable; high functional cohesion, high static coupling, synchronous dynamic coupling
   - A microservice in a workflow
   - A bounded context in DDD
-->

Topologies of one (because of the static coupling of a single database or front-end, even if multiple services are deployed),

- **Layered monolith**. Horizontal technical layers; e.g., MVC
- **Modular monolith**. Vertical functional layers
- **Microkernel**. Large core surrounded by plug-in components
- **Service-based**. Like microservices, but a shared database “because the architects didn’t see the value in separation (or saw too many negative trade-offs).” 32
    - You can focus on the domains, not the database
    - “When migrating monolithic applications to microservices, consider moving to a service-based application first as a stepping stone to microservices.” 72
- **Microservices** with a tightly coupled _single_ front-end (reasoning: the UI won’t work without the services, and because they serve the same UI they probably won’t have different scaling requirements)

🤔 Because of Automat etc, aren’t we service-based, with a large shared library?

Topologies of many:

- **Microservices** with micro front-ends: each service has its own static dependencies, including its own database

#### Chapter 3. Architectural modularity
Elasticity: speed with which scale can change. Monolithic applications have a large MTTS.

> Elasticity relies on services having a very small mean time to startup (MTTS), which is achieved architecturally by having very small, fine-grained services. 57

| Architecture  | Scalability | Elasticity | Notes                         |
|---------------|------------:|-----------:|-------------------------------|
| Monolithic    |         ⭐️ |          ⭐️ | Application-level scalability |
| Service-based |     ⭐️⭐️⭐️ |        ⭐️⭐️ | Domain-level scalability      |
| Microservices | ⭐️⭐️⭐️⭐️⭐️ |  ⭐️⭐️⭐️⭐️⭐️ | Function-level scalability    |

🤔 Many other aspects are missing here: maintainability, debuggability, etc

#### Chapter 4. Architectural decomposition
Tactical forking: instead of extracting functionality to create domains, copy the entire application to each domain, then delete what’s not needed. It makes more sense in apps that are very badly structured. But the drawbacks are problematic: without extra effort you’ll end up with duplicate, inconsistent and leftover code.

🤔 It doesn’t seem like this as a serious option

#### Chapter 5. Component-based decomposition patterns
“Component-based decomposition… is a highly effective technique for breaking apart a monolithic application when the codebase is somewhat structured and grouped by namespaces (or directories).” 82

“Initially, these patterns are used together in sequence when moving a monolithic application to a distributed one, and then individually as maintenance is applied to the monolithic application during migration.” 82

Architectural fitness functions are set up to make sure we don’t deviate from the plan later on. Some of these are used for planning, or even evaluating whether the move is feasible:

- **Identify and size components**
    - List the number of statements per domain (or lines of code) and consider breaking up large domains into subdomains. Show per cent size. We’re looking for generally same-sized domains
    - Fitness function: flag too big components
- **Gather common domain components**
    - Differentiate between domain functionality (formatting, validation…) and infrastructure functionality which is common to all processes (logging, security…)
    - Fitness function: flag duplicate code
- **Flatten components**
    - Within functional grouping, everything should be a leaf, even shared code
    - Fitness function: no shared code in root directories
- **Determine component dependencies**
    - Visualise the dependencies between modules to understand how complex the project will be (ignore shared infrastructure modules)
    - Fitness function: flag too many dependencies (incoming, outgoing, or both)
    - Fitness function: disallow dependencies between certain modules
- **Create component domains**
    - They propose this structure: domain/subdomain/component
    - Fitness function: flag new components
- **Create domain services**
    - Once all domains are working and established, move them out to separate services, one at a time (start service-oriented—keep the shared database)

#### Chapter 6. Pulling apart operational data
Consider splitting the database when:

- Services are competing for resources (connections, throughput)
- You don’t want one database outage to bring everything down (fault tolerance)
- You don’t want to tie schema changes to multiple services (lockstep deployments)
- You want different database types

| Criteria            | Relational | Key-Value | Document | Column    | Graph  | Distributed SQL | Cloud Native | Time-Series |
|---------------------|-----------:|----------:|---------:|----------:|-------:|----------------:|-------------:|------------:|
| Ease of learning    |   ⭐️⭐️⭐️⭐️ |    ⭐️⭐️⭐️ |   ⭐️⭐️⭐️ |      ⭐️⭐️ |     ⭐️ |          ⭐️⭐️⭐️ |         ⭐️⭐️ |          ⭐️ |
| Ease of modeling    |     ⭐️⭐️⭐️ |        ⭐️ |   ⭐️⭐️⭐️ |        ⭐️ |   ⭐️⭐️ |          ⭐️⭐️⭐️ |         ⭐️⭐️ |        ⭐️⭐️ |
| Scalability         |       ⭐️⭐️ |  ⭐️⭐️⭐️⭐️ |     ⭐️⭐️ |  ⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️ |          ⭐️⭐️⭐️ |     ⭐️⭐️⭐️⭐️ |    ⭐️⭐️⭐️⭐️ |
| Partition tolerance |         ⭐️ |  ⭐️⭐️⭐️⭐️ |   ⭐️⭐️⭐️ |  ⭐️⭐️⭐️⭐️ | ⭐️⭐️⭐️ |          ⭐️⭐️⭐️ |       ⭐️⭐️⭐️ |        ⭐️⭐️ |
| Consistency         | ⭐️⭐️⭐️⭐️⭐️ |      ⭐️⭐️ |     ⭐️⭐️ |        ⭐️ | ⭐️⭐️⭐️ |            ⭐️⭐️ |       ⭐️⭐️⭐️ |      ⭐️⭐️⭐️ |
| Maturity            |   ⭐️⭐️⭐️⭐️ |    ⭐️⭐️⭐️ |   ⭐️⭐️⭐️ |      ⭐️⭐️ |   ⭐️⭐️ |            ⭐️⭐️ |         ⭐️⭐️ |        ⭐️⭐️ |
| Read/write priority |   Flexible |     80/20 |    80/20 |     20/80 |  80/20 |        Flexible |        80/20 |       60/40 |

⚠️ I think there’s a lot missing that could be important, depending on your use case: schema-less, aggregate queries, text search (where’s Elastic?), memory databases… Also, within each category, different products can have different characteristics.

| Database type   | Products                                   |
|-----------------|--------------------------------------------|
| Relational      | PostgreSQL, MySQL, Oracle, SQL Server      |
| Key-value       | Riak KV, Amazon DynamoDB, Redis, Memcached |
| Dcoument        | MongoDB, Couchbase, Amazon DocumentDB      |
| Column          | Cassandra, Scylla, Amazon SimpleDB         |
| Graph           | Neo4j, Infinite Graph, Tiger Graph         |
| Distributed SQL | VoltDB, ClustrixDB, SimpleStore            |
| Cloud native    | Snowflake, Datomic, Amazon Redshift        |
| Time-series     | InfluxDB, kdb+, Amazon Timestream          |

📘 [NoSQL Distilled: A Brief Guide to the Emerging World of Polyglot Persistence](https://www.amazon.com/NoSQL-Distilled-Emerging-Polyglot-Persistence/dp/0321826620) (Pramod Sadalage, Martin Fowler).

#### Chapter 7. Service granularity
> “… Let’s take our customer notification service as an example. We can notify our customers through SMS, email, and we even send out postal letters. So tell me everyone, one service or three services.”
>
> “Three,” immediately answered Taylor. “Each notification method is its own thing. That’s what microservices is all about.”
>
> “One,” answered Addison. “Notification itself is clearly a single responsibility.”
>
> “I’m not sure,” answered Austen. “I can see it both ways. Should we just toss a coin?”
>
> “This is exactly why we need help,” sighed Addison.
>
> “The key to getting service granularity right,” said Logan, “is to remove opinion and gut feeling, and use granularity disintegrators and integrators to objectively analyse the trade-offs and form solid justifications for whether or not to break apart a service.” 186

Disintegrators:

- Is the service doing too many unrelated things? Is the service too big?
    - Example: the customer service is doing unrelated things: profile, preferences and comments
    - Similar reasoning: the code doesn’t fit easily into one person’s head
- Different rates of change
    - Example: the weekly changes made to one subdomain cause tests to the whole domain
    - These things might not go together (if they were cohesive they would change at the same time)
- Different performance demands
    - Example: notification methods have different scaling profiles: SMS 200,000/minute, email 500/minute, letters 1/minute
- Is a subdomain bringing all other subdomains down?
    - Example: out of memory errors in the email code brings down the other subdomains
- Do some parts need different security levels?
    - Example: a wallet service (with credit card information) vs a profile service (less secure)
- Is the service always expanding to add new contexts?
    - Example: a payments service that does credit cards, gift cards and PayPal, needs to accommodate ApplePay, rewards points, etc—consider breaking each into its own service
    - Apply this driver once “a pattern can be established or confirmation of continued extensibility can be confirmed.” 196

<!--
🤔 They argue an interesting perspective: out of the three notification methods, imagine email has to be extracted because of fault tolerance. Now, what do you call a service that notifies via SMS and letters? Because of naming, they break them up! I see it, but would struggle to justify it, and I would settle for a potentially less ideal set of names: EmailNotifications, and for everything else, Notifications

> This naming problem relates back to the service scope and function granularity disintegrator—if a service is too hard to name because it’s doing multiple things, then consider breaking apart the service. 194
-->

Integrators:

- Shared database objects: foreign keys, triggers, procedures, views or JOINs
- Are ACID transactions required?
    - Example: registering a new user across different services (profile and password) breaks ACID guarantees
- Do services need to talk to each other?
    - Example: when dependencies go down and consequently take down upstream services, the fault tolerance disintegrator doesn’t make sense
    - Example: getting data from a bunch of different services is slower than if they were one (this may not matter in batch processing)
- Do they need to share code?
    - Example: five services all use a shared library (not infrastructure or cross-cutting code) which changes, causing five separate deploys
    - As a rule of thumb, if the shared code is >40% of the collective codebase, consider integrating
    - The problem is exacerbated by frequent changes to the shared lib
- Does the data need to stay together?
    - Example: two different services that rely on each other’s data, maybe there are FKs: you have to choose between a lot of chatter between the services, or keeping them together

> “In that case,” said Addison, “let’s analyse the trade-offs. Which is more important—isolating the assignment functionality for change control purposes, or combining assignment and routing into a single service for better performance, error-handling, and workflow control?”
>
> “Well,” said Taylor, “when you put it that way, obviously the single service. But I still want to isolate the assignment code.”
>
> “OK,” said Addison, “in that case, how about we make three distinct architectural components in the single service. We can delineate assignment, routing, and shared code with separate namespaces in the code. Would that help?”
>
> “Yeah,” said Taylor, “that would work. OK, you both win. Let’s go with a single service then.” 211

> Parker noticed how Austen handled the meeting by facilitating the conversation rather than controlling it. This was an important lesson as an architect in identifying, understanding, and negotiating trade-offs. 215

<!--
🤔 How do you figure out trade-offs for other types of decisions? Is there a way to create a standard list that could work in many instances?

- Security
- Consistency
- Sync/async
- … ?
-->

### Part 2. Putting things back together
“Attempting to divide a cohesive module would only result in increased coupling and decreased readability.” (Larry Constantine) 217

#### Chapter 8. Reuse patterns
- **Code replication**
    - Copy code into each service; after that, the code isn’t kept in sync
- **Shared library**
    - Keep the libraries small, to reduce upstream dependencies
    - Always version, for the same reason
    - Use a deprecation strategy
    - Don’t have upstream dependencies rely on the latest version—hot fixes may break those
    - Has compile-time checks
    - Use when the shared code changes infrequently
- **Shared service**
    - No compile-time checks
    - Versioning isn’t straightforward
    - Latency, scalability and fault tolerance concerns
    - Use when changes are frequent or in a polyglot environment
- **Sidecars and service mesh**
    - Sidecar is a helper process/container that runs along the main service (on Kubernetes, in the same pod)
    - You use it for infrastructure concerns like logging, networking, etc
    - Allows polyglot services
    - Service meshes insert sidecars automatically, declaratively
    - It’s a utility with a runtime, and runs next to each service (not centralised)

Code reuse isn’t always good. It creates brittleness: if reused code breaks, the change “could potentially impact every other domain, requiring coordination and testing to ensure that the change hasn’t ‘rippled’ throughout the architecture.” 243

> What architects failed to realise is that reuse has two important aspects; they got the first one correct: abstraction. The way architects and developers discover candidates for reuse is via abstraction. However, the second consideration is the one that determines utility and value: rate of change. 243

Reuse creates a dependency. If that shared component changes frequently, everyone who uses it feels it. So good reuse isn’t just about identifying common patterns (the abstraction bit), it’s about finding stable abstractions that don’t churn.

#### Chapter 9. Data ownership and distributed transactions
- **Single ownership scenario**. The service that writes to the table becomes the owner
    - Rule of thumb: if only one service writes to a table, that service owns the table
- **Common ownership**. Everyone or most services write to the table
    - Example: audits
    - Solution: create a service that’s called to write to the table
- **Joint ownership**. A few services write to the table
    - Solutions:
        - Split the table, sharing PKs. If distributed, CAP
        - Share table access, somehow making the table it’s own bounded context (“data domain”)
        - Delegate technique: assign ownership to one service. Which? “The first option, called primary domain priority, assigned table ownership to the service that most closely represents the primary domain of the data—in other words, the service that does most of the primary entity CRUD operations for the particular entity within the domain. The second option, called operational characteristics priority, assigned table ownership to the service needing higher operational architecture characteristics, such as performance, scalability, availability, and throughput.” 258
            - “Regardless of which service is assigned as the delegate (sole table owner), the delegate technique has some disadvantages, the biggest being service coupling and the need for interservice communication.” 260
        - Consolidate the services: turn them into a single service. Problems: increased testing scope, deployment risk, fault tolerance risk; can’t scale independently

📘 [Refactoring Databases](https://databaserefactoring.com/SplitTable.html) (Pramod Sadalage)

<!--
Eventual consistency patterns:

- Background synchronisation
- Orchestrated request-based (try doing everything synchronously)
- Event-based
-->

#### Chapter 10. Distributed data access
- **Interservice communication**. Service A asks service B for the data it needs; a network call
    - Pros: simple
    - Cons: slow (latency and throughput), no fault tolerance
- **Column schema replication**. Add service B data to the service A database, through queues, topics, event streaming…
    - Pros: performance, fault tolerance
    - Cons: consistency and ownership issues; complex data synchronisation
- **Replicated caching**. Replicated read-only cache replica on A (distributed cache)
    - Pros: fast access, scaleable, fault tolerance, data ownership preserved
    - Cons: limited data size, slower-changing data only
- **Data domain**. Discussed previously: A and B tables are contained in a separate data service which both can access; this is last resort
    - Cons: coupling

#### Chapter 11. Managing distributed workflows
“Orchestration is useful when an architect must model a complex workflow that includes more than just the single ‘happy path,’ but also alternate paths and error conditions.” 302

Centralisation makes error-handling and recovery easier, and state of the workflow is easily available. In contrast: potential bottleneck in terms of performance, single point of failure, limited scalability and coupling.

<!--
Semantic coupling: inherent in the requirements. “However clever an architect is, they cannot reduce the amount of semantic coupling, but their implementation choices may increase it.” 309
-->

Workflow state using choreography:

- **Front-controller pattern**. State is held by the first triggering service and each service reports as they complete
    - This is presented as a benefit: “Creates a pseudo-orchestrator within choreography” 312 🤔
    - Cons: adds workflow state to a domain, increases communication overhead
- **Stateless choreography**. “… build a workflow that queries the state of each domain to determine the most up-to-date order status.” 313
- **Stamp coupling**. Pass state along with the message passed between services. This gives services more context but it doesn’t really provide a place to query job status

Choreography is more scalable (performance, parallelism), fault tolerant (choreographer is a single point of failure, but isn’t it the case that the more moving parts, the higher risk of failure? 🤔), more decoupled. In contrast: no central state management (and much harder to debug!), error-handling is more difficult (services must have more knowledge… of each other, so coupling?), recoverability is more difficult (retries, etc).

> In the choreographed solution, removing the mediator forces higher levels of communication between services. This might be a perfectly suitable trade-off. For example, if an architect has a workflow that needs higher scale and typically has few error conditions, it might be worth trading the higher scale of choreography with the complexity of error handling. 315

> … as workflow complexity goes up, the need for an orchestrator rises proportionally… 315

They lean on orchestration because trying to choreograph complex workflows can make things worse: you smear the mess around (similar to the way you smear related functionality across boundaries in a layered architecture).

> Ultimately, the sweet spot for choreography lies with workflows that need responsiveness and scalability, and either don’t have complex error scenarios or they are infrequent. 316

#### Chapter 12. Transactional sagas
I’ve highlighted the patterns with the highest scores; they all use eventual consistency:

| Pattern/saga    | Communication | Consistency | Coordination | Coupling  | Complexity | Availability | Scale     |
|-----------------|---------------|-------------|--------------|-----------|------------|--------------|-----------|
| Epic            | Synchronous   | Atomic      | Orchestrated | Very high | Low        | Low          | Very low  |
| Phone Tag       | Synchronous   | Atomic      | Choreography | High      | High       | Low          | Low       |
| **Fairy Tale**  | Synchronous   | Eventual    | Orchestrated | High      | Very low   | Medium       | High      |
| **Time Travel** | Synchronous   | Eventual    | Choreography | Medium    | Low        | Medium       | High      |
| Fantasy Fiction | Async         | Atomic      | Orchestrated | High      | High       | Low          | Low       |
| Horror          | Async         | Atomic      | Choreography | Medium    | Very high  | Low          | Medium    |
| **Parallel**    | Async         | Eventual    | Orchestrated | Low       | Low        | High         | High      |
| **Anthology**   | Async         | Eventual    | Choreography | Very low  | High       | High         | Very high |

Complexity is the odd one out:

- With choreography (without a mediator), resolving errors becomes part of the responsibility of each service, so complexity increases
- Everything gets harder with async communication; the trade-off is higher performance and lower coupling

Simplifying a lot:

| In order to…         | Communication   | Consistency  | Coordination     | Pattern/saga       |
|----------------------|-----------------|--------------|------------------|--------------------|
| Reduce coupling      | **Async**       | **Eventual** | Choreography     | Anthology          |
| Reduce complexity    | **Synchronous** | **Eventual** | **Orchestrated** | Fairy Tale         |
| Improve availability | Async           | **Eventual** |                  | Parallel/Anthology |
| Improve scale        | Async           | **Eventual** | Choreography     | Anthology          |

Atomic consistency refers to distributed transactions. (They talk about compensating updates only, not multi-phase commits.)

> Distributed transactions are not something that can be simply “dropped into” a system. They cannot be downloaded or purchases using some sort of framework or product like ACID transaction managers—they must be designed, coded, and maintained by developers and architects. 356

Additional complications with compensating updates:

- Other services that may have already taken action before the compensating update is propagated, and there is no feasible way to undo that. These are called “side effects.” It could be a service downstream of the one invoked by the orchestrator, or it may happen by another incoming call to the service invoked by the orchestrator. “This scenario points to the importance of _isolation_ within a transaction, something that distributed transactions do not support.” 361
- A compensating update can fail!

> Alternatively, the mediator could insist that other services don’t accept calls during the course of a workflow, which guarantees a valid transaction but destroys performance and scalability. 363

Eventual consistency can be managed through state management. Consider a service that sends a survey at the end of a purchase. The orchestrator can mark this as _pending_ and retry later; failing that, ping someone to fix manually, or give up (some failures to send surveys may be tolerated).

#### Chapter 13. Contracts
“Many architects like strict contracts because they model the identical semantic behaviour of internal method calls. However, strict contracts create brittleness in integration architecture—something to avoid.” 367

They’re saying you should be able to e.g., add fields to an object passed between services and not cause everyone to change.

More: “Keeping contracts at a ‘need to know’ level strikes a balance between semantic coupling and necessary information without creating needles fragility in integration architecture.” 369

<!--
Loose is JSON, no schema.

Strict contacts provider guarantees (but tight coupling), are easier to verify at build time, better docs. A good use case is when “information is highly semantically coupled and likely to change together” 379

Lose contacts are highly decoupled, easier to evolve, needs something like consumer-driven contracts. (An interesting use case is for apps that need approval from app stores—they’re harder to change).

Stamp coupling = over-specify details that aren’t needed (return the “full object” even of consumer only needs some fields). “Over-specifying details in contracts is generally an anti-pattern…” 377

A valid use case is coordination in choreography (“the semantic coupling must go somewhere” 378)
-->

#### Chapter 14. Managing analytical data
- Data warehouse (ETL)
- Data lake (ELT)
- Data mesh

<!--
Data warehouse pattern (~ETL)

- Centralised consolidation of data, isolation, but right coupling between operational database and transform logic
- Makes more sense in monolithic applications

Data lake pattern (~ELT)

- Centralised, isolated, less up-front transformation, better suited to distributed systems, but puts the burden on the analytics team to massage the data, dealing with PII becomes more problematic

Data mesh pattern

- Data becomes another product of each domain; data stays distributed but is available through self-serve mechanisms
- You would serve the data from a separate, tightly coupled, source/quantum (not your operational/transactional database)
- Each of these can then feed a common aggregate analytics service
- Suited to microservices, and when analytical data doesn’t have to stay in sync at all times
-->

#### Chapter 15. Building your own trade-off analysis 
- Focus on business drivers
- Cross-team collaboration
- Trade-off analyses
- Architecture decision records

1. Find the coupling points in the architecture: “if someone changes X, will it possibly force Y to change?” 401
    - Static coupling
    - Dynamic coupling (communication, consistency and coordination)
2. Analyse the coupling points: “model the possible combinations in a lightweight way.” 402
    - “The goal of the analysis is to determine what forces the architect needs to study—in other words, which forces require trade-off analysis? For example, for our architecture quantum dynamic coupling analysis, we chose coupling, complexity, responsiveness/availability, and scale/elasticity as our primary trade-off concerns, in addition to analysing the three forces of communication, consistency, and coordination…” 402

Trade-off techniques:

- “It’s important for architects to be sure they are comparing the same things rather than wildly different ones.” 404. E.g., don’t compare simple message queues to enterprise service buses
- Don’t compare all pros/cons, choose the right things to balance depending on your own context/needs
    - Having fewer options to consider will also make decisions easier
    - “Finding the correct narrow context for decisions allows architects to think about less, in many cases simplifying design.” 407
- Consider working through the problems iteratively, “diagramming sample architectural solutions to play qualitative ‘what-if’ games to see how architecture dimensions impact one another.” 407
    - This is easier when you combine it with the idea below: narrow down on the important factors (vs trying to evaluate everything, regardless of your own context)
- Model relevant cases
    - Imagine trying to decide between a single payments service, or a service per provider
    - Adding a new provider is simpler with multiple services
    - Maintenance, because of shared code doesn’t have a clear winner
    - But if you need to introduce a complex workflow using multiple providers (maybe one is a rewards points service), then you need to choose between “performance and data consistency (a single payment service) or sensibility and agility (separate services).” 410. A or B, simple!
- Definitely simplify, consolidate and focus on outcomes for non-technical stakeholders
- Beware of evangelists. Even if something has always worked for someone, the contexts they’ve worked in may not match the current one. “We advise architects to avoid evangelising and to try to become the objective arbiter of trade-offs.” 415 Remember: “it’s not an argument if two sides don’t exist.” 415
