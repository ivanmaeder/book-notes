[Back](/)

# Building Microservices: Designing Fine-Grained Systems
Sam Newman (2021)

---

## TL;DR
- 🚨

## Notes
### Chapter 1. What are Microservices?
Emerged from DDD, continuous delivery, on-demand virtualisation, infrastructure automation, systems at scale.

Key concepts:

- Independently deployable
- Model a business domain
- Separate databases/state
- Small enough that they can fit in your head 

Advantages of microservices:

- Technology heterogeneity
- Robustness, because of one service goes down or fails it shouldn’t bring down others. E.g., Analytics made a change in the warehouse database and it affected Client Ops—those teams should have separate databases
- Isolate issues, resilience
- Independently scale
- Easier/nimbler deploys
- Composability
- Flexibility in terms of human resource assignment
- Replaceability

Pain points:

- Running a tonne of services on localhost
- Risk of technology overload
- Higher costs (tech and people)
- Reporting, because the data isn’t in one central location anymore
- Monitoring and troubleshooting ⚠️ 
- Security
- Testing
- Latency
- Data consistency

### Chapter 2. How to Model Microservices
> In essence, microservices are just another form of modular decomposition, albeit one that has network-based interaction… 36

Modular decomposition gives us:

- Information hiding
  - Can work on the internals independently
  - Makes things easier to understand and work with
  - Reduce the number of dependencies between modules
- High cohesion
  - Keep related code together so that changes are localised (vs a change that requires many changes in many different parts of a system)
- Loose coupling
  - Changing one module/service (in this discussion we’re working with the assumption that boundaries have already been defined) shouldn’t cause a change to another. Continual contract changes would kill independent deployability

Types of coupling, from loose too tight:

- Domain: where 2 services interact with each other. “A microservice that needs to talk to many downstream services might point to a situation in which too much logic has been centralised.” 42
  - Temporal coupling: when two services have to be running at the same time
- Pass-through coupling: this manifests when a service is doing some work for a downstream service it doesn’t call directly. Instead make the intermediary do the work (or, less preferably, explicitly invoke the service from the original service—make the contract explicit; or have the intermediary completely ignorant of the data sent to the final service so that if it has to change the changes don’t affect the intermediary). Remember we want to avoid deploying in lock-step
- Common coupling: services access shared data, like a database. This is more problematic the more the data structure changes, or the data is read and written to by the services
- Content coupling: a service has access to read or change internal state of another service (e.g., a service accessed another’s database). “You’ve lost the ability to define what is shared (and therefore cannot be changed easily) and what is hidden. Information hiding has gone out the window.” 51

Domain-driven design (DDD),

- Ubiquitous language: use the language of your users
- Aggregate: a domain concept like an order or invoice. They typically have an identity and a life cycle and can be modelled as state machines. Aggregates should be managed within services, not across services. Aggregates are often multiple objects (e.g., an order without line items doesn’t make much sense, and an order with its line items has a life cycle)
  - “A single microservice will handle the lifecycle and data storage of one or more different types of aggregates.” 53
  - He then goes on to talk about how your service can say no to state transitions. I think this is related to information hiding: if another service can reach in and change the state to something I think is illegal, I’ve failed at information hiding
  - Relationships between aggregates “could be managed by the same microservice or by different microservices.” 53
- Bounded context: an organisation boundary. Contains various aggregates, not all may be shared. Some aggregates may be stored separately by different bounded contexts but there should still be a single source of truth (a single owner). Bounded contexts can contain other bounded contexts
  - “Bounded contexts hide implementation detail.” 56
  - “As with aggregates, bounded contexts may have relationships with other bounded contexts—when mapped to services, these dependencies become inter-service dependencies.” 56
  - “Our domain is the whole business…” 56
  - Examples—“they both have an explicit interface to the outside world… and they have details that only they need to know about…” 56
    - Finance department
      - Interface: pay slips…
      - Details: calculators…
  - Warehouse
      - Interface: inventory reports…
      - Details: forklift trucks…
    - Shared information: the warehouse needs to know where stock is within the warehouse, the finance department needs to know how much stock we have. Finance might refer to the same thing as “assets.” “We store information about the stock item in both locations, but the information is different.” 58
  - Bounded contexts are powerful because they present “a clear boundary to the wider system while hiding internal complexity that is able to change without impacting other parts of the system… whether we realise it or not, we are also adopting information hiding…” 62

<!--
  - What if we modelled debts differently to placements?
    - A debt has a life cycle that starts with the credit and a balance that changes over time, and certain events like being charged off, payments being made, etc
    - A placement’s life cycle is when we’re asked to collect on the debt. There may be multiple placements for a single debt. When customers send us debts they’re asking us to collect on the debt (to create a new placement, except when they send us duplicates)
    - We could go further: placement request which is kind of an order from the client
  - For borrowers, we can model the person, but the person has different representations depending on the client. There are rules around where we can share borrower information between clients
-->

Both aggregates and bounded contexts can serve as service boundaries; early on you will typically have larger services, so bounded contexts. But never split aggregates into multiple services

Splitting services into smaller services can probably be hidden from users

Tip: instead of storing another service’s aggregate’s FK in your database, you could be more explicit and e.g., store:

- `/customer/141` (a URI)
- `january:customer:141`

💡 Isn’t this the same old advice? Common language, model entities on real-world concepts, etc?

Event storming:

- Get everyone together, business and tech
- Identify events e.g., order placed, payment received
- List the commands/decisions that make these things happen
- Then group everything into aggregates
- The group into bounded contexts (typically, the company’s organisational structure)

Other ways to think about service boundaries:

- Frequency of changes of different parts of the system (keep the parts that change the most separate from those that change less). Volatility (there’s a tool that can help figure this out on p74)
- Keep services that handle some types of data separate, e.g., PII
- Group by tech stack
- Group by organisational boundaries (Conway’s law)

“… shared ownership of microservices is a fraught affair.” 66

Splitting teams along horizontal layers is a terrible idea; they're much too coupled.

### Chapter 3. Splitting the Monolith
What’s the real goal? Microservices is _not_ the end goal. Are there other, easier ways to achieve these goals?

Chip away at the problem. This way you can learn as you go, and if you make a mistake it’s not a big one.

> You won’t appreciate the true horror, pain, and suffering that a microservice architecture can bring until you are running in production. 73

Knowing why will also help prioritise what parts to split out first.

Approaches:

- Code first. Typically easier than database first. If you mess this up it’s easier to revert and stop there
- Database first
- Strangler fig
- Parallel run
- Feature toggle

Remember:

- Data you could JOIN before is now in separate databases
- Integrity isn’t provided by the database
- Transactions across services need to be managed

It’s handy to flush data from multiple services to a reporting database.

### Chapter 4. Microservice Communication Styles
- In-process communication
- Vs inter-process
  - Networked (slow!). Reconsider lots of small calls, the amount of data passed around (not just transportation, but serialisation and deserialisation. Don’t abstract this away: callers should know
  - Compatibility: refactoring makes it easy to change both in-process caller and callee at the same time, not so here
  - Additional error types: network timeouts, downstream services unavailable, disconnections, no responses (did the requests even arrive?)

🚨 Image from 94

> It’s common for a single microservice to implement more than one form of collaboration. 95

Synchronous blocking:

- Typically because you either need a response to keep going, or want some sort of guarantee and retry if it fails
- The callee needs to be up to receive and respond; the caller needs to be up to receive the response
- Callee is constrained to do the work quickly (i.e., not wait forever for another downstream service). The longer the chain of calls, the more problematic this pattern becomes
  - Higher risk of something in the chain causing problems
  - Resource contention: all these network connections remaining open (running out of connections, or network congestion)
- If the callee is under load and slow, retries could exacerbate the issue

Asynchronous non-blocking:

- Both services don’t have to be running at the same time
- If processing in the downstream service takes a long time, no problem
- Additional complexity though

(JavaScript promises are asynchronous and blocking: “synchronous style” 100)

🚨 Asynchronous option 1: shared data store:

- Couples the services. The upstream service should keep its data private and have the ability to modify its store without affecting others
- “Whether this pattern really shines is in enabling interoperability between processes that might have restrictions on what technology they can use.” 104
- “Another major sweet spot for this pattern is in sharing large volumes of data. If you need to send a multigigabyte file to a filesystem or lost a few million rows into a database, then this pattern is the way to go.” 104

Event-driven:

- “… inversion of responsibility.” 109. Does this mean it’s the downstream service’s responsibility to “retry” if it fails or wasn’t around when the event was emitted?
- Strong when loose coupling is important
- “Personally, I find myself gravitating toward event-driven almost as a default.” 115
- Implementation
  - Message brokers (e.g., RabbitMQ) receive and forward. “In general, I’m a fan.” 110. Also: “… keep your middleware dumb, and keep the smarts in the endpoints.” 111
  - HTTP, e.g., Atom. “… HTTP OS not good at low latency… and we must still deal with the fact that the consumers need to keep track of what messages they have seen and manage their own polling schedule.” 111
- Payloads
  - If you expect the downstream service to query data from the upstream service…
    - That’s an additional dependency
    - It could also lead to a heavy workload if there are a lot of downstream services 
  - “The alternative, which I prefer, or to put everything into an event that you would be happy otherwise sharing via an API.” 113
    - This can also improve auditability
    - But size limits: Kafka 1MB, RabbitMQ 512MB
    - Also, the data is shared indiscriminately: e.g., PII and this data becomes part of the contract (but this would be the same with the other approach, no?)

Terminology: “The message is the medium; the event is the payload.” 110

> Our workers kept dying. And dying. And dying.
> 
> Eventually, we tracked down the problem. A bug had crept in whereby a certain type of pricing request would cause a worker to crash. We were using a transacted queue: as the worker died, its lock on the request timed out, and the pricing request was put back on the queue—only for another worker to pick it up and die…
>
> Aside from the big itself, we’d failed to specify a maximum retry limit for the job on the queue. We fixed the bug itself, and also configured a maximum retry. But we also realised we needed a way to view, and potentially replay, these bad messages. We ended up having to implement a message hospital (or dead letter queue), where messages got sent if they failed. We also created a UI to view those messages and retry them if needed. These sorts of problems aren’t immediately obvious if you are only familiar with synchronous point-to-point communication.
>
> The associated complexity with event-driven architecture and asynchronous programming in general leads me to believe that you should be cautious in how eagerly you start adopting these ideas. Ensure you have good monitoring in place, and strongly consider the use of correlation IDs, which allow you to trade requests across process boundaries…

📘 Enterprise Integration Patterns (Gregor Hohpe Bobby Woolf)

### Chapter 5 🚨
When choosing a communications technology, you’re aiming to:

- Make backward compatibility easy
- Explicit interfaces
- Technology-agnostic
- Easy: e.g., providing a library or easy tech that will make writing client code easy

Remote procedure call (RPC),

- Makes a remote call look local
- Easy client-side code generation
- But: technology coupling (e.g., Java RMI) and sometimes brittleness (e.g., when removing fields)
- And remember local calls are not remote calls:
  - Compared to local calls, marshalling/unmarshalling is costly
  - Network calls even more
  - Service boundaries have to be better designed (you can’t just refactor)
  - Networks can be much more unreliable
- gRPC: “I actually quite like RPC, and the more modern implementations, such as gRPC, are excellent…. gRPC would be at the top of my list.” 127
- Java RMI: brittle, limited technology choices
- SOAP: heavyweight
- Recommendations: don’t abstract the network away

> It’s [gRPC] high on my list whenever I’m in situations where I have a good deal of control over both the client and server ends of the spectrum. If you’re having to support a wide variety of other applications that might need to talk to your micro services, the need to compile client-side code against a server-side schema can be problematic. In that case, some form of REST over HTTP API would likely be a better fit. 127

Representational state transfer (REST),

- Check out the Richardson Maturity Model (https://martinfowler.com/articles/richardsonMaturityModel.html)
- Take advantage of HTTP ecosystem. E.g., security, caching proxies
- But: generating client code isn’t automatic
  - OpenAPI/Swagger can do this for various languages! The documentation spec may not be as explicit as needed though
- Also not as lean as binary protocols
- HATEOAS: premature optimization; “I haven’t seen much evidence that the additional work to implement this style of REST delivers worthwhile benefits in the long run, nor can I recall in the last few years talking to any teams implementing a micro service architecture that can speak to the value of using hates.” 132

> … although you can construct asynchronous interaction protocols over the top of REST-based APIs, that’s not really a great fit compared to the alternatives for general microservice-to-microservice communication. 132

Recommendation: on the perimeter sync and async, internally sync.

GraphQL:

- “… it makes it possible for a client-side device to define queries that can avoid the need to make multiple requests to retrieve the same information.” 133
- But: there can be server-side load issues: “An expensive SQL statement can cause significant problems for a database and potentially have a large impact on the wider system. The same problem applies with GraphQL. The difference is that with SQL we at least have tools like query planners for our databases… whereas a similar problem with GraphQL can be harder to track down.” 133-4
- Also caching is more complex
- Also not as good for writes? “This leads to situations in which teams are using GraphQL for read but REST for writes.” 134
- Recommendation: “Just because you are using GraphQL, don’t slip into thinking of your micro services as little more than an API on a database—it’s essential that your GraphQL API isn’t coupled to the underlying datastore of your micro services.” 134
- Great for supporting GUIs, mobile devices
- “Fundamentally, GraphQL is a call aggregation and filtering mechanism, so in the context of a micro service architecture it would be used to aggregate calls over multiple downstream micro services. AS such, it’s not something that would replace general microservice-to-microservice communication.” 135

Message brokers:

- Intermediaries, middleware, manage communication between processes
- “A message could contain a request, a response, or an event.” 135
- Queues: point-to-point
- Topics: multiple consumers can subscribe and each subscriber receives a copy
  - Consumer group: only one instance of the group receives the message
- “Topics are a good fit for event-based collaboration, whereas queues would be more appropriate for request/response communication.” 136
- Guaranteed delivery: the downstream service will hold onto messages until they’re delivered
  - Compare to e.g., REST where the caller has to figure out what to do if the downstream service isn’t available
  - To reduce data loss, brokers are typically distributed
  - “… all brokers have restrictions as to how they need to be run to deliver the promise of guaranteed delivery. If you plan to run your own broker, make sure you read the documentation carefully.” 138

- Some brokers can deliver messages in order, or almost/mostly in order
- Some can do transactions on write or read transactionality 🚨
- Some promise (but don’t always deliver?) exactly once delivery. “This is a complex topic …” last line of 138 🚨

Kafka lets you query data inside topics—interesting.

📘 Designing Event-Driven Systems (Ben Stopford)

Schemas catch structural changes; tests catch semantic changes. Without schemas, the burden of testing is higher. This is similar to the static v dynamic languages debate. He recognises some benefit to dynamically typed languages, but not going schemaless for services.

Tips for avoiding breaking changes:

- Add things, don’t remove things
- As a client, be tolerant. E.g., don’t use tech that breaks if new fields are added; with something like XPath we could even work around changes in the location of fields
- Choose tech that makes backward compatibility easy
- Define an explicit interface???????????? Is this saying ahead of time “we might change blah”???????? 🚨
  - For event-based endpoints: asyncapi.com, cloudevents.io
- Catch errors early, before changes are deployed ⭐️ 
  - Protolock, json-schema-diff-validator, openapi-diff… but these won’t check backwards compatibility
  - These can: Confluent Schema Registry (Kafka)
  - Pact is a contract-driven testing tool
  - Obviously, testing

Tips for managing breaking changes:

- Lockstep deploy. Forces services to deploy together which is exactly wish you’re trying to avoid. Leads to a “distributed monolith” 152
- Coexist multiple versions: “I am not a fan of this idea personally, and understand why Netflix uses it rarely.”
  - Bug fixes are needed in 2 places
  - State persisted by both services needs to be consistent or handled some other way
- Emulate the old interface
  - This is fairly ingenious! You create new outwardly facing versions of the endpoint, but internally they’re all using the same code, or calling into the same service. Then once nobody is using the old interface you drop it
  - The phrase for migrating users to be interfaces (regardless of what you do internally) is expand and contact

Teams should agree on:

- How to communicate that an interface needs to change
- How to collaborate to agree on the change
- Who needs to update the consumer code
- How long do consumers have to migrate

General tip: ask consumers to identify their service with e.g., user-agent. Helpful to know who’s using your service and how. In the current context, to know who’s still using the old version.

Extreme solutions to getting people to migrate from an old version:

- Make it slow
- Turn it off

Be careful with shared code/libraries: there will be different versions on different services.

Service discovery, API gateways (externally-facing, like a reverse proxy) and service meshes (between services). Use the last two for shared, common cross cutting functionality (logging, rate limiting—“microservice-agnostic behaviour” 163. Plus for meshes: mutual TLS, correlation IDs, service discovery, load balancing…)

Remember: dumb pipes, smart endpoints. The functionality should not be “specific to any one microservice.” 168

Generic config, _no_ business functionality should be leaked.

Service mesh: “It’s up there with selecting a message broker or cloud provider in terms of how seriously I’d take it.” 168

But options are limited without Kubernetes. Also they mostly work for REST and other HTTP interfaces, not so much message brokers.

🚨 The documentation tools mentioned in the last pages of the chapter

### Chapter 6. Workflow
Distributed transactions:

- E.g., two-phase commit: ask systems to “vote” (voting phase) and only continue if all systems agree; typically they will lock resources to provide guarantees that the transactions can happen
- But things are not so simple. E.g., what if there are technical glitches after the voting phase?
- Instead:
  - Keep the tables in the same database
  - Use a distributed RDBMS like Google Spanner

Sagas:

- Compensating actions are taken place if there are errors that can’t be continued
- As each step is executed, compensating actions are recorded. E.g., if there’s a failure halfway through booking a hotel and flight: the continuation (retry) is to book the flight; the compensation is to cancel the flight

Sagas can be orchestrated or choreographed:

- He prefers choreography over orchestration: “… in my experience, the extra complexity associated with tracking the progress of a saga is almost always outweighed by the benefits associated with having a more loosely coupled architecture.” 195
- However: “… I am very relaxed in the use of orchestrated sagas when one team owns implementation of the entire saga.” 195

💡 In Architecture: The Hard Parts, they seem to strongly prefer orchestration

Also, don’t forget about re-ordering steps to avoid these problems. E.g., don’t email the customer until payment is taken, or don’t bill the customer until the order ships.

💡 The example he uses for choreographed sagas is interesting because instead of taking payment based on the “Order placed” event, he takes payment after the “Stock reserved” event (192). This simplifies things in one way—we won’t take payment until we confirm the product is in stock—but makes implementation less intuitive: now the payments service needs to know that it shouldn’t listen for “Order placed” events, only “Stock reserved” events

Pat Helland:

> In such systems, the larger it gets, the more likely the system is going to be down. When flying an airplane that needs all of its engines to work, adding an engine reduces the availability of the airplane.

### Chapter 7. Build
Trunk-based development, and how it’s really the correct way to do continuous integration.

Continuous delivery: every commit is a release candidate.

Continuous deployment: goes beyond that—every commit is deployed.

Using multiple repositories (typically per service or per team) sounds painful, because when you make changes across repositories they need to be coordinated and because of the workflow of managing multiple repositories (apparently nowadays, IDEs can handle this well though). Note that working across boundaries should be the exception, not the rule! If you’re working across boundaries too much there’s too much coupling and you should consider merging the services.

A monorepo allows for atomic commits across project boundaries. However! It doesn’t give you atomic rollout, so you still need to coordinate the deployment. He says monorepos work well for very large tech companies that can build tooling around it (Google, Microsoft, Facebook, Twitter, Uber) and small companies (10-20 engineers). It’s hard for middle-sized organisations. He feels that one repo per service is the right approach.

Code re-use can be easier with a monorepo.

Levels of code ownership:

- Strong: changes can only be made by the owners
- Weak: change can be made by anyone, but owners review
- Collective: anyone can do anything

### Chapter 8. Deployment
Principles of microservice (only microservices?) deployment:

- Microservices run on independent resources—whatever load or impact they have on the hardware, it doesn’t impact other services
- It’s all automated
- Infra as code
- Zero-downtime deploys
  - Kubernetes can do rolling upgrades (old instances are kept alive as new ones replace them) but is overkill if that’s all you want
  - Blue-green is easier
  - Long-lived connections can be problematic
- Automated state management: if an instance goes down or traffic increases, the platform should start new instances
  - Kubernetes, auto scaling groups

At the beginning, deploying everything to a single machine is typical, but it has problems:

- Which service is using all that CPU or RAM?
- How do you handle contradictory dependencies? E.g., different versions of Python? Docker—what about different versions of Docker or Linux libs?

Isolation levels, from stronger to weaker isolation, and from higher cost/time to lower: physical machine, virtual machine, container

“… for the vast majority of workloads, containers are ‘good enough,’ which is in large part why they are such a popular choice and why they tend to be my default choice in most situations.” 231

Gilt and the FT have 3 services per engineer!

Deployment options:

- Physical machine
- Virtual machine (virtual private servers—VPS—, EC2) improves utilisation and reduces costs. Each VM has its own share of hardware resources, an OS
  - Type 1 virtualisation: VMs run directly on hardware, not on an OS
  - Type 2: EC2, VMware. The hypervisor simulates the hardware for the guest VMs
- Container (Docker)
  - Containers don’t have a hypervisor, they can run their own OS, but they use the same kernel as the host OS. Containers perform better (without the hypervisor) are much faster to provision than VMs (seconds v minutes)
  - Consider better isolation if “you are running code written by others and are concerned about a malicious party trying to bypass container-level isolation” 244
- Application container (Java application servers, .NET)
- Platform as a service (PaaS), Heroku, Google App Engine, AWS Beanstalk, Netlify
  - Autoscaling isn’t great in practice—they work for the average application, but not specific use cases
- Function as a service (FaaS), AWS Labmda, Azure Functions
  - Databases, queued etc can also be serverless
  - AWS Lambda functions can be expensive if it’s being executed all the time
  - Beware of the scale of server less overwhelming other parts of your system that don’t scale so well. This can happen in other places too! ⚠️ 
  - If you deploy different serverless functions (e.g., one function per aggregate) but they’re still part a bigger API (or from there outside world,a single service), make it transparent for users

He recognises that Kubernetes isn’t way to use asks points to Knative (not stable at time of writing) or OpenFaaS as possible alternatives. But also: “Expect Kubernetes in your future.” 274

Progressive delivery:

- Blue-green deployments
- Feature flags. Can be used for canary releases, enable/disable a problematic feature, or at a specific time
- Canary releases 
- Parallel runs. Functionality runs in two places for comparison

Deploy doesn’t have to be the same as release.

### Chapter 9. Testing
> Previously, testing was predominantly carried out before the software got to production. Increasingly, though, we look at testing our applications once they arrive in production… 275

- Acceptance testing (business-facing)
  - Did we build the right thing?
  - Automated
- Unit testing (developer-facing)
  - Did we build it right
  - Automated
- Exploratory testing
  - How can I break the system?
  - Manual
- Property testing
  - Response time, scalability, security…

> If you currently carry out large amounts of manual testing, I would suggest you address that before proceeding too far down the path of micro services… 277

Manual testing should be exploratory (or impossible/difficult to automate).

Unit tests are at the bottom of the pyramid because they’re fast to run, easy you write, less brittle, and it’s easy to pinpoint errors. These benefits disappear as you go up the pyramid; that’s replaced by tests that ensure the system as a whole work.

They use the term “service tests” for end-to-end within an isolated service.

Typically you’ll have an order of magnitude fewer tests as you move up the pyramid:

- 10 end-to-end
- 100 service
- 1,000 unit

In an inverted pyramid:

- Build times slow down
- Builds can be broken for a long time
- Feedback loops are longer

(Because: brittleness, tests take longer to run, and it’s harder to pinpoint errors)

In service tests, you stub out or mock other services:

> … we create a stub microservice that responds with canned responses to known requests from the micro service under test. For example, I might tell my stubbed Loyalty micro service that when asked for the balance of customer 123, it should return 15,000. A variation on this is to use a mock instead of a stub.
>
> When using a mock, I actually go further and make sure the call was made. If the expected call is not made, the test fails. Implementing this approach requires more smarts in the fake collaborators that we create, and if overused it can cause tests to become brittle. As noted, however, a stub doesn’t care if it is called 0, 1, or many times. 283

I didn’t quite understand why stubs are preferred over mocks, so I asked ChatGPT:

- Mocks are implemented separately; a single stub class can stand in for a whole service, making it much easier to keep up to date and reason about
- Mocks can lead to over-specification which makes tests fragile. E.g., imagine adding an extra call parameter that doesn’t change the result. A mock `assert_called_with` will fail (false negative), but a stub won’t
- A mock might tell you that the call is wrong (`assert_called_with`), but a stub will return the wrong thing which generally makes it easier to figure out why the call is wrong to begin with

Back to the book: “The balance between stubbing and mocking calls is a delicate one and is just as fraught in service tests as in unit tests. In general, though, I use stubs far more than mocks for service tests.” 283-4

Implementing stubs:

- Implement yourself, with Apache, Nginx…
- https://mbtest.org/ (does this work?)
- Locally, you should only run your own services and stub out the rest

Real end-to-end tests (across multiple divide services) are often flaky/brittle (a service may not be up, a network connection may fail, data conflicts, service version mismatch, third-party dependencies, timing issues, test environment drift… these are all from ChatGPT). It’s also harder to diagnose where the failure occurred when these tests fail.

- Which version of the other services do we run? The one in production, or the one queued up to deploy?
- What if the downstream service uses other downstream services. Which of those do we run? Aren’t we duplicating tests by doing this?

(As a solution, consider a shared end-to-end tests CI stage, but still, brittleness)

> When we detect flaky tests, it is essential that we do or best to remove them. Otherwise we start to lose faith in a test suite that “always fails like that.” 287

Who writes end-to-end tests?

- Start with the team that owns the upstream service under test. But that can be problematic when other teams’ services can break the tests
- Having a dedicated test team is a disaster (“… developing the software becomes increasingly distant from the tests for its code. Cycle times increase, as service owners end up waiting for the test team to write end-to-end tests… Because another team writes these tests, the team that wrote the service is les involved with, and therefore less likely to know, how to run and fix these tests.” 289)
- Try to avoid end-to-end tests; or “Teams are free to check in to this suite, but ownership of the health of the suite has to be shared between the teams developing the services themselves… I think this approach is essential, and yet I have seen it done only rarely, and never without issues. Ultimately, I am convinced that at a certain level of organisational scale, you need to move away from cross-team end-to-end tests for this reason.“ 289

> I have seen [end-to-end tests] take up to a day to run, if not longer, and on one project I worked on, a full regression suite took six weeks! 289

Test speed is super important: “While a broken integration test stage is being fixed, more changes from upstream teams can pile in. Aside from the fact that this can make fixing the build harder, it means the scope of changes to be deployed increases. The ideal way to handle this is to not let people check in if the end-to-end tests are failing, but given a long test suite team, that is often impractical… The larger the scope of a deployment and the higher the risk of a release, the more likely we are to break something. So we want to make sure we can release small, well-tested changes frequently.“ 290

> Ideally, if you want your teams to be able to develop and test in an independent fashion, they should have their own test environments too. 291

Explicit schemas between services can catch structural issues and reduce the need for end-to-end tests. For semantic testing, consider:

- A contract test is written by the consumer (upstream) service, and it describes “how it expects an external service will behave.” 292
  - These tests can be run by the consumer against their service stubs, confirming the tests and stubs are in sync
- Downstream services run these tests in their own test suite (because it’s the downstream service that needs to be running to test the contracts)
- Then when a change is made by the producer and a test fails, it’s clear which of the upstream consumers are affected
  - “At this point, you can either fix the problem or else start the discussion about introducing a breaking change…” 293
- Producers and consumers should collaborate to write the contract tests
- https://pact.io/ (and https://github.com/pact-foundation/pact_broker)

Most teams end up removing the need for end-to-end tests; but use them like training wheels and only remove them once they’re confident they don’t need them anymore 296

Testing before hitting production increasingly has diminishing returns. The environment on production isn’t the same. Consider:

- Pinging services
- Smoke tests
- Canary releases
- Synthetic tests

> Sometimes expending the same effort on getting better at fixing problems when they occur can be significantly more beneficial than adding more automated functional tests. In the web operations world, this is often referred to as the trade-off between optimising for mean time between failures (MTBF) and optimising for mean time to repair (MTTR). 299

- Fast rollbacks
- Good monitoring

Non-functional tests should also follow the pyramid. E.g., if you discover a load test issue through an end-to-end test, implement a narrower-scoped load test, “Other [non-functional tests] fit faster tests quite easily.” 300

- Performance. “Due to the time it takes to run performance tests, it isn’t always feasible to run them on every check-in. It is a common practice to run a subset every day, and a larger set every week. Whatever approach you pick, make sure you run tests as regularly as you can. The longer you go without running performance tests, the harder it can be to track down the culprit. Performance problems are especially difficult to resolve…” 300

- Accessibility
- Load time
- Latency
- Security
- Etc

Bibliography:

- https://www.youtube.com/watch?v=vq8o_AFfHhE
- https://martinfowler.com/articles/nonDeterminism.html
- https://www.martinfowler.com/articles/practical-test-pyramid.html
- https://www.martinfowler.com/bliki/TestDouble.html
- https://martinfowler.com/articles/enterpriseREST.html
- https://medium.com/capital-one-tech/moving-one-of-capital-ones-largest-customer-facing-apps-to-aws-668d797af6fc
- Agile Testing (Lisa Cirspin, Janet Gregory)
- Learning Chaos Engineering (Russ Miles)
- Growing Object-Oriented Software, Guided by Tests (Steve Freeman, Nat Pryce)
- Succeeding with Agile: Software Development Using Scrum (Mike Cohn)
- The Challenger Launch Decision: Risky Technology, Culture and Deviance at NASA (Diane Vaughan)
- Accelerate: The Science of Building and Scaling High Performing Technology Organisations (Nicole Forsgren, Jez Humble, Gene Kim)

### Chapter 10. From Monitoring to Observability
Re the increased complexity of microservices: “In no situation is this increased complexity more evident than when it comes to understanding the behaviour of our systems in a production environment.” 305

> You won’t truly appreciate the potential pain, during, ash’s anguish caused by a microservices architecture until you have it running in production and serving real traffic. 305

From a tweet: “We replaced our monolith with micro services so that every outage could be more like a murder mystery.” 306

- Metrics (aggregation)
- Events
- Logs (aggregation)
- Traces (flow of calls across services)
- Error budgets, SLAs, SLOs etc
- Alerts

Log aggregation as a prerequisite for microservices. Also start using correlation IDs at the beginning—it’s harder to put them in later.

Use consistent formatting that will later make retrieving/filtering by e.g., customer ID, correlation ID, easy. Consider JSON.

To determine the chronology, NTP can help but it’s not foolproof. Use a counter or traces.

Cardinality and choosing the right tool for metrics (321-3)

For traces he recommends a tool that supports the OpenTelemetry API (and OpenTracing?).

Error budgets: you can afford more risk when you haven’t used the budget.

Alerts should be:

- Relevant
- Unique
- Timely
- Prioritised
- Understandable
- Diagnostic
- Advisory
- Focusing

Semantic monitoring isn’t about specific no errors, but figuring out at a high level, if things are working the way they should be. E.g.,

- Can new users register?
- Are we selling $x/hour at peak times?
- Are we shipping at a normal rate?

Production testing/monitoring:

- Synthetic transactions: execute use flows, can do this regularly
- A/B testing
- Canary release
- Parallel run
- Smoke tests
- Chaos engineering

Logging and monitoring should be standard for all services. Same log format, all metrics together.

As apps evolve they naturally evolve from passive/responsive monitoring to observability/alerts.

### Chapter 11. Security
Greater attack surface (more machines, more data transferred over the network), but more flexibility.

You need to address all these aspects:

- Identify
- Protect
- Detect
- Respond
- Recover

Credential theft appears in 80% of hacking cases, including brute force attacks.

- Temporary credentials
- Rotate credentials (automatically)
- Restrict access (principle of least privilege)
- Have the ability to revoke access
- Don’t share credentials across services, or even across instances
- Patch/upgrade the OS, container images, libraries…
- Set up automatic alerts for vulnerabilities
- Back up databases, logs
- Have the ability to restore systems (do this as part of CI/CD to limit exposure)
- Don’t implicitly trust everything inside the perimeter; a bad actor can wreak havoc if they gain access to any vulnerable parts. Instead: zero trust, or act as if there were no perimeter (permimeterless)

Implicit trust = “any calls to a service made from inside our perimeter are implicitly trusted” 366

This is the most common! Problems though:

- If an attacker penetrates the network, they can intercept anything
- Change the data
- Act as a service

Zero trust: assume you're working in a compromised environment (no perimeter)

- You can accept connections over the internet!

Mixed. E.g., zero trust for PII systems. Example of a medical company, where the more secret services can access less secret services, but not the other way around:

- Public data (can be shared with anyone; public domain)
- Private (user must be logged-in. E.g., insurance plan of a user)
- Secret (available to non-users in very specific situations—health data)

To simplify, he draws on an example where services were categorised based on the data they handle, into different levels of security. Less secret levels could access more public levels, or in its own level.

Data in transit:

- Server identity (avoid malicious parties impersonating services) can be verified with HTTPS, but that can be problematic if you want reverse proxies to cache data (they can’t see the data)
- Client identity (auth) is trickier; service meshes help
- Data should be encrypted
- To avoid data manipulation, consider HMAC (hash-bad message authentication code)

Data at rest:

- Encrypt, don’t roll your own, subscribe to mailing/advisory lists
- Use salted password hashing
- Remember to scrub PII from logs
- Be frugal with data collection (collect less, delete more)

Storing private keys:

- Separate security appliance
- Separate key vault

Don’t forget to encrypt backups

To minimise risk, a session token is used against the SSO gateway (centralized authentication), and the gateway generated a short-lived JWT on every request (decentralised authorisation; you want each service to authorise the user individually).

### Chapter 12. Resiliency
- Robust: accommodate expected/known problems. Not just software: what if the on-call engineer isn’t available?
- Rebound: recover from incidents, unknown errors. E.g., have backups, incident playbooks
- Sustained adaptability: challenge ourselves to continually adapt to ensure future resiliency

> Having the bandwidth to really examine such surprises and extract the key learnings requires time, energy, and people—all things that will reduce the resources available to you to deliver feature in the short term. 391

> To work toward sustained adaptability means that you are looking to discover what you don’t know. 391

> Taken more broadly, the ability to deliver resiliency is a property not of the software itself but of the people building and running the system. 391

https://oreil.ly/aYIjx

Issue of downstream service failing and making the caller hang and deny further requests (there was a request thread pool getting used up, timeouts weren’t configured correctly).

> What was worse 397 🚨

> We ended up implementing three fixes to avoid this happening again: getting our time-outs right, implementing bulkheads to separate out different connection pools, and implementing a circuit breaker to avoid sending calls to an unhealthy system in the first place. 397

When setting timeouts:

- Use a default
- Log all timeouts, and adjust
- Consider dynamic settings based on how much time is left before we want to give up on the overall request

Bulkheads isolate parts of a system so if one part fails, only that part fails. Circuit breakers can be seen “as an automatic mechanism to seal a bulkhead…” 401

> Given the perils 401 🚨

Load shedding is… 🚨

408 middle of the page “It is essential” 🚨

CAP theorem:

- Consistency: same answer no matter who you ask
- Availability: every request receives a response
- Partition tolerance: system is able to handle breaks in communication between parts

AP system: database replicas can’t communicate: we keep the system running, there’s a partition, but consistency is lost (eventual consistency).

CP system: we shutdown because we can’t guarantee consistency with a partition (this is really hard, do not try building it yourself).

CA system: not relevant (if there’s no partition it’s not a distributed system).

Chaos engineering notes starting 414 and tools on 415 🚨

Asking the same questions 418 🚨

### Chapter 13. Scaling
- Vertical scaling
- Horizontal duplication
  - Service requests through a load balancer
  - Spin up additional workers
  - Read replicas (can also use a load balancer)
- Data partitioning
  - Some databases support sharding natively!
  - “Data partitioning scales really nicely for transactional workloads. If your system is write-constrained, for example, data partitioning can deliver huge improvements.” 428
  - Geographical partitioning: useful to roll out changes when least disruptive, also data sharing rules
  - Problem: querying across shards
- Functional decomposition (scale each service independently)

### Chapter 14. User interfaces
Drive to have front-end teams:

- Scarcity of specialists. But this used to happen with database work which is now not specialised: embed specialists in your team and the knowledge will be shared. Consider spreading specialists across teams, or putting them in enabling teams that “consult” internally
- Consistent look and feel. Consider whether this is really that important
- Technical challenges: it’s hard to break up UIs, especially some SPAs

Monolithic front-end is a single unit. The state is in the UI, and synced to the back-end as needed. Hard to split this kind of pattern across multiple teams.

Micro front-ends can be worked on independently, either by splitting along widgets or by pages.

> Sometimes the capabilities offered by a microservice do not fit neatly into a widget or a page. … The more cross-cutting a form of interaction is, the less likely this model will fit, and the more likely it is that we'll fall back to just making API calls. 465

Page-based decomposition: like traditional web sites (not SPAs).

Widget-based decomposition:

- You need a “container” app and an assembly layer
- Iframes have sizing issues, and communication between iframes is difficult
- Components can communicate over events
- Consider using web components
- Main concern is payload size of every widget is it’s own bundle
- Don’t forget mobile vs desktop, accessibility

Central aggregating gateway:

- “Gateway” receives one request from the UI and makes separate requests to the services to get all the information (the UI doesn’t make multiple calls). Reduce bandwidth, improve throughput
- Problem: who owns it? It can be a source of contention between teams
- Beware of bloat when you need a bunch of slightly different gateway endpoints (e.g., the mobile version is slightly different to the desktop version)

Back-end for front-end (BFF)

- Like an aggregating gateway but for a specific UI (not across the whole app)
- The BFF will route calls to other services if needed
- Can also think of it as the UI split into two parts: on the client device, and on the server side (BFF)
- Prefers one BFF per type of device (Android and iOS would use different BFFs even if the functionality is similar), partly because the front-ends for each device will typically be owned by different teams. “Conway’s law wins again.” 482
- Not as common for desktop devices, because they’re more powerful. But a BFF could e.g., do server-side rendering, with a reverse proxy cache
- Use when you have a significant amount of aggregation in the back-end
- It’s also a pattern that can be used to group functionality for third parties. I guess it’s just adding a layer that combines internal calls

> As I have said before, I am fairly relaxed about duplicated code across microservices.
Which is to say that, while in a single microservice boundary I will typically do whatever I can to refactor out duplication into suitable abstractions, I don't have the same reaction when confronted by duplication across microservices. This is mostly because I am often more worried about the potential for the extraction of shared code to lead to tight coupling between services. 485

> Avoiding the trap of putting too much behavior into any intermediate layers is a tricky balancing act. 489

Aggregate layers can be implemented with GraphQL.

### Chapter 15. Organizational structures
Stream-aligned teams; “loosely coupled organisation” 491

From the Accelerate book (492), teams should be able to make “large-scale changes to the design of their system…” 492

- Without outside permission
- Without others having to change their system
- Without coordinating with other teams
- Deploy, release and test regardless of others
- Deploy during business hours without downtime

Historically, loosely coupled organisations create more modular systems.

To deliver faster, work needs to happen in parallel.

> The biggest cost to working efficiently at scale in software delivery is the need for coordination. 497

From John Rossman, Think Like Amazon:

> The Two-Pizza Team is autonomous. Interaction with other teams is limited, and when it does occur, it is well documented, and interfaces are clearly defined… One of the primary goals is to lower the communications overhead in organisations, including the number of meetings, coordination points, planning, testing, or releases. Teams that are more independent move faster. 497

Collective ownership reduces the options (vs strong ownership). There’s more coordination:

> With teams—and people—moving more frequently from microservice to microservice, we require a higher degree of consistency about how things are done. 501

Cross-cutting concerns (databases, languages, Kubernetes…) can be handled by enabling teams, architects, communities of practice… instead of stream-aligned teams figuring things out separately

The “paved road”: if you want everyone to do X, make it easy to do X. Also communicate why X.

> If a team has a lot of inbound pull requests, it could be a sign that you really have a microservice that is being shared by multiple teams. 513

> Coming up with a vision for how things should be done without considering how your current staff will feel about it and without considering what capabilities they have is likely to lead to a bad place.
> … Understand your staff’s appetite for change. Don’t push them too fast! Maybe you can have a separate team handle frontline support or deployment for a short period, giving your developers time to adjust to new practices. 523

> Conway’s law highlights the perils of trying to enforce a system design that doesn’t match the organisation. 524

### Chapter 16. The Evolutionary Architect
Architect as town planner in Sim City: you decide the zones, but not what gets built in them. In software our zones are “microservice boundaries, or perhaps coarse-grained groups of microservices. As architects, we need to worry less about what happens inside a zone and more about what happens between the zones.” 531

Architects should work closely with implementation teams: to help people understand the vision, “but also to change the plan when reality challenges that vision.” 533

> Architecture is what happens, not what is planned. 534

Grady Booch:

> In the beginning, the architecture… is a statement of vision. In the end… the architecture… is a reflection of the billions upon billions of small and large, intentional and accidental design decisions made along the way. 534

Habitabilty! “… enables programmers coming to the code later in its life to understand its construction and intentions and to change it comfortably and confidently.” 535 (Richard Gabriel)

> I cannot emphasise how important it is for the architect to actually spend time with the teams building the system! 535

Define principles and practices, based on strategic goals. Principles apply more broadly and hardly change. Practices may differ amongst teams. Example:

- Strategic goals
  - Enable scalable business
  - Support entry into new markets
  - Support innovation in existing markets
- Architectural principles
  - Reduce inertia: make choices that favour rapid feedback, with reduced dependencies across teams
  - Eliminate accidental complexity: aggressively retire and replace unnecessarily complex components, so we can focus on the essential complexity
  - Consistent interfaces and data flows: eliminate duplication of data; create clear systems of record
  - No solve bullets: off-the-shelf solutions deliver early value but create intention and accidental complexity 
- Design and delivery practices
  - Standard REST/HTTP
  - Encapsulate legacy
  - Eliminate integration databases
  - Consolidate and cleanse data
  - Published integration model
  - Small independent services
  - Continuous deployment
  - Minimal customisation of COTS/SaaS

📘 Building Evolutionary Architectures (Neal Ford, Rebecca Parsons, Patrick Kua, Pramod Sadalage)

Fitness functions to measure how close to ideal architecture you’re getting.

### Afterword: Bringing it All Together
> You should expose only the bare minimum over your service interfaces to satisfy your consumers. The less you expose, the easier it is to ensure the changes you make are going to be backward compatible. 551

> Internal databases should not be directly exposed to external consumers, as this causes too much coupling between the two, which undermines independent deployability. In general, avoid situations in which multiple microservices all access the same database. 552

> Each microservice should have its own build, its own CI pipeline. 555

> You won’t get all of these decisions right, I can guarantee that. So, knowing you are going to get some things wrong, what are your options? Well, I would suggest finding ways to make each decision small in scope; that way, if you get one wrong, you impact only a small part of your system. Learn to embrace the concept of evolutionary architecture, in which your system bends and flexes and changes over time as you learn new things. Think not of big-bang rewrites, but instead of a series of changes made to your system over time to keep it supple. 562

## Notes From the First Edition
Ask first what a microservice needs to do, don’t start with what data it needs.

Synchronous communication:

- Easier to reason about, simpler
- Better visibility/observability

Asynchronous:

- Great for long-running work
- Highly decoupled
- Leads to spreading logic instead of centralised

RPC looks like a local functional call, which sounds like a good idea, but it's not: a remote call is often very different to an in-process call—you still probably want to think about the network: latency, connection errors, etc (all the extra things that can go wrong). RPC may also require lock-step deploys to keep the client and server stubs in sync.

Shared code (DRY) can sometimes be problematic with microservices. Not just logic, but entities too.

It's also problematic for the microservice owners to write the client library: the risk is that logic seeps into the client library.

Another thing to think about: the example of asking an email service to send an email. Imagine it’s queued up, and by the time it goes to send the message, the customer’s email has changed. Instead consider passing in links so that the email service can query the info it needs.

Splitting things up. Gives the example of a financial report that used to do a JOIN on the product table, but now those things will be in separate databases. This is fine: make an API call from the financial service to the other service.

> Typically concerns around performance are now raised. I have a fairly easy answer to those: how fast does your system need to be? And how fast is it now? If you can test its current performance and know what good performance looks like, then you should feel confident in making a change. Sometimes making one thing slower in exchange for other things is the right thing to do, especially if slower is still perfectly acceptable. 85

What about something like a COUNTRY table? Duplicating the table may be a problem: data could become out of sync. So consider putting the data in a property file and somehow try to keep that in sync (his recommendation). Otherwise put the static data into a service.

Another example: two services that update customer (order) data: finance saves payments and refunds, warehouse updates dispatch information. Solution: split out a customer service used by both.

Transactions:

- Keep trying failed (eventual consistency)
- Undo the committed transactions (compensating transaction). Complicated when there are >2 transactional boundaries
- Distributed transactions (transaction manager). E.g., two-phase commit. Don't create your own

Pattern: send a request, get a 202; continue polling until a 201 (created) is received.

Reporting is the only exception where a service reaches into the database of another service. But the dump from the source of the data to the warehouse should be maintained by the owners of the source data. The alternatives are:

- Publish APIs to return data in large batches
- Empty events which the reporting service subscribes to
- Don’t report centrally
