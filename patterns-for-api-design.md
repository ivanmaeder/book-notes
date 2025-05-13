[Back](/)

# Patterns for API Design: Simplifying Integration with Loosely-Coupled Message Exchanges
Olaf Zimmermann, Mirko Stocker, Daniel Lübke, Uwe Zdun, Cesare Pautasso (2023)

---

## TL;DR

## Notes
> In this book, we are not concerned with microservices infrastructures but with service rightsizing on the API level (in terms of endpoint granularity and operation/data coupling). 13

APIs as products: “They are supposed to have a dedicated business owner, a governance structure, a support system, and a roadmap.” 13

Finding the right balance:

- Exposing data vs hiding implementation
- Independent evolution vs “decrease the cost of bringing two systems together” 16
    - The “dual connector-separator role” 19
- General endpoints vs one per client
    - Different clients want different things, and they keep changing their minds
- Fine-grained vs coarse-grained endpoints
- Few operations with lots of data vs chatty interactions with little data
- Correct vs current data
- Stable vs fast-changing contracts

Measure along: performance, scalability, reliability, security, manageability, ease of use, … … …

Once an API is used, “it will become increasingly expensive to apply corrections and improvements and impossible to remove features without breaking some clients.” 18


