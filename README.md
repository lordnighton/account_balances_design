# Aims and goals
- To design and implement a mechanism that will enhance an existing OLS balances API with:
    - Support of new **channels** (configurable set of values, specified in Spring config):
        - web
        - iPad (CDX)
        - iPhone
        - Android phone
        - Android tablet
        - `other channels` (?)
    - Support of new **options** (configurable set of values, depends on amount of entities provided by current API):
        - accounts with editable nickname
        - future account benchmarcs
        - future available for withdraw
        - ...
    - Support of new **response formats** (should be transparent and should not affect the logic of services):
        - JSON (no actions, simple data representation with raw JSON)
        - `other formats` (?)
    
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
- Newly introduced aspects - **v2**
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
    - I was NOT involved in a development of **v1** API
    - I am NOT aware of all corener cases
    - I am NOT aware of business value of all data interconnections and dependencies
    - It is always risky to change existing codebase that is covered with tests only **partially**
- Caching
    - From the perspective of proxy caches in the middle, each approach has advantages and disadvantages: if the URI is versioned, then the cache will need to keep multiple copies of each Resource – one for every version of the API. This puts load on the cache and decreases the cache hit rate, since different clients will use different versions. Also, some cache invalidation mechanisms will no longer work. If the media type is the one that is versioned, then both the Client and the Service need to support the Vary HTTP header to indicate that there are multiple versions being cached.
    - From the perspective of client caching however, the solution that versions the media type involves slightly more work than the one where URIs contain the version identifier. This is because it’s simply easier to cache something when its key is an URL than a media type.
- Split view and business types of logic
- Content negotiation (json, HAL/json)

# Best practices and examples


# Sequence
- 

# Advantages
- Layering
- Better code coverage
- No changes in existing functionality
- Less supervision from OLS guys
- Less effort
- Less testing (including manual)
- Flexibility
- Less latency & less network load 

# Design patterns


# Testing strategy
- No changes without tests

# Useful links
- https://medium.com/@mwaysolutions/10-best-practices-for-better-restful-api-cbe81b06f291
- http://www.springboottutorial.com/spring-boot-versioning-for-rest-services
- https://medium.com/@javadevjournal/rest-api-versioning-guide-d2f42be45fb1s
- https://stackoverflow.com/questions/389169/best-practices-for-api-versioning
- https://www.baeldung.com/rest-versioning