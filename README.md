# Requirements and scope (27) -> solution design (9) -> implementation (3)
1. UBS Accounts view on iPad (**v1 -- phase #1**)
    - Preliminary discussions
        - **Talked to Ellie**. BnL we need to customize just a couple of CDX-specific things at UBS Accounts view
        (see `Requirements` section)
        - **Talked to Vitalii Bondur**. UI team is fine with `HAL+json` format so that there is no need to apply the
        content negotiation (JSON, XML, etc.).
    - Requirements
        - As an iPad user I need the totals for credit cards / mortgages (and other tiers) to be shown on UBS Accounts
        view
        - As an iPad user I need CDX-specific actions to be shown on UBS Accounts view for each account / group / tier
    - Most annoying aspects
        - [HAL+json](https://en.wikipedia.org/wiki/Hypertext_Application_Language) format - regular JSON + Hypermedia
        aspects. For now UI components know how to deal with this format and I even insist on using this format and the
        objects hierarchy, because it is really close to the table structure we show iPad users on UBS Accounts view.
        - **An amount of markup we send to client** (columns, links, etc). Incoming solution will allow to reduce an amount
        of info sent to CDX and suppress required these pieces at the `converters` layer.
        - **Tiers**. Finally we realized that there is a requirement to merge some groups at the `MTP-view`. Out of
        this task scope (separate UI service, see `MTP-view on iPad` section)
        - **An amount of time we spend to get the results**. For now we spend 1-2 minutes to obtain the results in OLS.
        Out of this task scope (see `Make a set of called data services configurable` section).
        - **Pagination**. Not supported by data services (?)
        - **Possible calculations on UI-side** (not critical now). If we want to show CDX-specific actions, we need some
        level of flexibility to be provided by `Account Balances UI service`. If we keep the UI service untouched, we
        have to implement this logic at UI-side.
    - Current design issues (technical part)
        - **Single responsibility principle** is broken at the level of business services. Business services are not
        cohesive (concentrated on a single task) now, but do caching, DAO-layer fetches, conversion of models obtained
        from DAO-layer into data objects exposed to API-clients and UI-specific data formatting (tabs, columns, etc).
        - **Less layers than required**. Controller->Services->DAO. Fixed (see the solution design and code examples).
        - **Cyclomatic complexity** of methods is `HUGE` (mostly because of the first two aspects).
        - **Less tests than required**. Mostly because of the first two aspects.
2. MTP-view on iPad (**v1 -- phase #2**)
    - Preliminary discussions
        - **Talked to Ellie**. BnL we need to customize just a couple of CDX-specific things. There is a requirement to
        merge some groups at the `MTP-view`. An amount of tiers we show to iPad users is less than in OLS.
    - Requirements
        - As an iPad user I need an amount of tiers to be reduced on MTP-view
        - As an iPad user I need some groups to be squashed on MTP-view
    - Most annoying aspects
        - The discussion of changes is still in progress and there is no clear plan yet
        - Separate UI service and that's why requires a dedicated solution design
3. Make a set of called data services configurable (**v1 -- phase #3**)
    - Preliminary discussions
        - **Talked to Sailaja, Alex and Mike**. Its enough to fetch accounts and withdraws only to render all needed
        data on UBS Accounts iPad view (already works in Sailaja's solution)
    - Requirements
        - As an Account Balances API client I need to obtain the results faster
    - Most annoying aspects
        - **Do we have 100% confidence?** that our partial fetch does not influence the returned data set (for now we
        are **90%** confident, but would be nice to cover ALL the corner cases with tests).
4. DDA (durable data API, **v2 -- phase #4**)
    - Preliminary discussions
        - **Talked to Amrit, Sailaja and Mike**. The support of DDA format is a nice-to-have (not only because of
        unclear roadmap, but also because of inconveniences with unclear UBS <-> DDA matching process)
    - Requirements
        - As an Account Balances API client I need the entities to be provided in DDA format
    - Most annoying aspects
        - **DDA format** is just a **set of recommendations**, an attempt to create a standard supported by all financial
        institutions in industry.
        - **Entities and properties**. When I compared the mortgages in UBS and DDA formats I realized that there are no
        matching fields at all :-) Credit cards -- only one match.
        - **Business analysis**. I am sure we have to talk with BAs and OLS team about matching procedure.
        - **Additional layer of UBS -> DDA converters**. An additional layer of converters from UBS format to DDA should
        be definitely implemented.
    - Good things
        - **Technical aspects**. One of the most important things is that our existing solutions already take into account
        the technical requirements / suggestions / recommendations explained in DDA document.
        - **There is a DDA XSD schema**. There is an XSD schema attached to DDA document that specifies what are the key
        entities, how the properties are called and what is the type of each property (including allowed values for Enums).
        - **There is a DDA Swagger YAML contract**. `fapi.yaml` was provided by OLS team (commit -- ?).

# Diagrams

# Aims and goals
- To design and implement a flexible mechanism that will enhance an existing OLS balances API with:
    - Support of new **channels** (configurable set of values, specified in Spring config or Enum):
        - web
        - iPad (CDX)
        - iPhone
        - Android phone
        - Android tablet
        - `other channels` (?)
    - Support of new **response formats** (should be transparent and should not affect the logic of services):
        - JSON (no actions, simple data representation with raw JSON)
        - DDA

![queen](https://user-images.githubusercontent.com/42194815/46779737-72cba980-ccef-11e8-89bc-3d665c797e1c.jpg)
    
# Key concepts
- We will use URI versioning approach
    - This is the most commonly used and straightforward approach while versioning a REST API
```bash
https://hostname/v1/products   VS   https://hostname/v2/products
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
    - I am NOT aware of all corner cases
    - I am NOT aware of business value of all data interconnections and dependencies
    - It is always risky to change existing codebase that is covered with tests only **partially**
    - Separate **v2** folder for v2-specific classes (? -- need to be aligned with best practices) 
- Caching
    - If the URI is versioned, then the cache will need to keep multiple copies of each Resource one for every version of the API. This puts load on the cache and decreases the cache hit rate, since different clients will use different versions. Also, some cache invalidation mechanisms will no longer work.

# Best practices and examples
```java
GET <ENV_URL>/wma/account-balances/v1/investment?channel=ipad
```

# Sequence
- Confirm the strategy inside the CDX team (Amrit, Mike, Alex, Attila, Sailaja)
- PoC for one of endpoints (**Investment balances** and Sailaja's fork)
- Talk to Mikhail Shkolnik and Oleg Belenkii about a plan and make everybody in sync
- v1-changes:
    - UBS Accounts view on iPad
        - Layering
        - Channels
        - Tests for introduced changes
    - MTP-view on iPad (+ testing)
    - Make a set of called data services configurable (+ testing)
- v2-changes:
    - Business analysis (including sync-up with OLS BAs)
    - DDA

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
