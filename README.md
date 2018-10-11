# Tasks
- Account Balances view on iPad
- DDA
- Make a set of called data services configurable

# Diagrams

# Aims and goals
- To design and implement a mechanism that will enhance an existing OLS balances API with:
    - Support of new **channels** (configurable set of values, specified in Spring config or Enum):
        - web
        - iPad (CDX)
        - iPhone
        - Android phone
        - Android tablet
        - `other channels` (?)
    - Support of new **options** (configurable set of values, depends on amount of entities provided by current API):
        - amounts
        - groups
        - (see `Horizontal slicing options` section)
        - ...
    - Support of new **response formats** (should be transparent and should not affect the logic of services):
        - JSON (no actions, simple data representation with raw JSON)
        - `other formats` (?)

![queen](https://user-images.githubusercontent.com/42194815/46779737-72cba980-ccef-11e8-89bc-3d665c797e1c.jpg)

# Advantages of URI versioning (v2-based) approach
- Better granularity and flexibility (provided by channels and options)
- Better layering and codebase structure
- Better tests coverage
- No changes in existing functionality
- Less supervision from OLS guys
- Less effort
- Less testing (including manual)
- Flexibility
- Less latency and less network pressure
    
# Key concepts
- We will use URI versioning approach
    - This is the most commonly used and straightforward approach while versioning a REST API
```bash
https://hostname/v1/products https://hostname/v1/customers
https://hostname/v2/products https://hostname/v2/customers
```
- Refactoring in existing codebase -- **v1**
    - I will try to **extract** the *markup* concepts (e.g. *balancesTab* and *performanceTab*) into a separate layer of
    architecture, b/c such kind of logic should NOT be present at the business services layer
        - Look through the **Testing strategy** section to understand how to safely perform such a change
        - Look through the **Design patterns** section to understand a new *Facades* and *Builders* layers
- Newly introduced aspects -- **v2**
    - Smooth process without the worries about existing functionality
    - Technical reasons
        - REST API design is a long-term commitment towards the users of that API, REST API should only be up-versioned
        when significant or groundbreaking changes made in the API. Here is the list of some common points when we:
            - Remove API part
            - Architectural changes to the API design
            - Changes in the response format
        As a rule of thumb, big number change (e.g v1 to v2) indicates groundbreaking changes or **significant milestone**
        (our case) in the REST API design and features. This also indicates REST API consumer significant changes. All
        minor changes like new endpoints etc. are known as non-breaking changes. Use minor version increment (v1.0 to
        v1.1) to show these changes in the API.
    - Tests only for newly introduced pieces of functionality
    - Less supervision and discussions
    - Less QA (manual)
    - I was NOT involved in a development of **v1** API
    - I am NOT aware of all corener cases
    - I am NOT aware of business value of all data interconnections and dependencies
    - It is always risky to change existing codebase that is covered with tests only **partially**
    - Separate **v2** folder for v2-specific classes (? -- need to be aligned with best practices) 
- Caching
    - If the URI is versioned, then the cache will need to keep multiple copies of each Resource � one for every version of the API. This puts load on the cache and decreases the cache hit rate, since different clients will use different versions. Also, some cache invalidation mechanisms will no longer work.
- Split view and business types of logic
- Content negotiation (raw json, HAL/json)

# Best practices and examples
```java
ENV_URL = "/wma/account-balances/v1/balances"

GET <ENV_URL>/v2/investment/amounts
GET <ENV_URL>/v2/investment/groups

GET <ENV_URL>/v2/investment?channel=ipad
    - In this case the configuration for a particular channel contains an amount of options to be returned:
        - groups
        - amounts

GET <ENV_URL>/v2/investment?options=amounts,groups
    - In this case we directly specify fetched options 
```

# Horizontal slicing options (see `Best practices and examples` section as well)
- Available set of options (groups, amounts, etc. for each API endpoint) should be confirmed separately
- Could be analyzed by live responses per endpoint
- Could be aligned with valies the specify the sequence of properties in response
```java
@JsonPropertyOrder({"id", "lastUpdatedOn", "criteria", "_embedded", "groups", "_links" })
```
- Sailaja's approach (filtering)
- Should be extracted to enum and carefully validated when the request comes to controller (`@Valid`)

# Sequence
- Confirm the strategy inside the CDX team (Amrit, Mike, Alex, Attila, Sailaja)
- PoC for one of endpoints (**Investment balances** and Sailaja's fork)
- Talk to Mikhail Shkolnik and Oleg Belenkii about a plan and make everybody in sync
- v1-changes:
    - Split response builders and business services (layering)
        - This will allow us to simplify the logic of services significantly
        - And prepare a ground for v2-changes
    - Tests for introduced changes
- v2-changes:
    - Options and channels (new approach)
        - Investment balances
        - Credit card balances
        - Investment market values
        - (revisit all the controllers in com.ubs.wma.rest.balances.v1.api.controller)
        - ...
    - Content negotiation (raw JSON for now)

# Design patterns
- Fac'ade
    - an additional layer of architecture that will obtain data as objects from existing services and DAOs layer 
    this layer will be responsible for obtaining the options to return / channel type and interact with strategies layer to create a response
- Strategy
    - according to passed set of options the layer of strategies will combine the response from the results provided by services and DAOs
- Builder
    - the set of builders will be involved in content negotiation mechanism (?)
    - should also be used to split view and business logic

# Testing strategy
- No changes without tests
- Tests first
- ALL existing tests should work before and after the changes
- Before we make any kind of change to existing code base, every line of code should be covered with unit tests (if not, we cannot make sure we have not broken anything)
- Right after the change ALL existing tests should pass (including the set of unit and integration tests created on the previous step)
- Would be nice to have all possible kinds of tests:
    - Unit tests -- the most simple and the most powerful coverage
    - Integration tests -- less by amount but should test the layers of architecture (including end-to-end RestAssured scenarios and MockMVC tests) 

# Key OLS persons
- Mikhail Shkolnik
- Oleg Belenkii

# Useful links
- https://medium.com/@mwaysolutions/10-best-practices-for-better-restful-api-cbe81b06f291
- http://www.springboottutorial.com/spring-boot-versioning-for-rest-services
- https://medium.com/@javadevjournal/rest-api-versioning-guide-d2f42be45fb1s
- https://stackoverflow.com/questions/389169/best-practices-for-api-versioning
- https://www.baeldung.com/rest-versioning
