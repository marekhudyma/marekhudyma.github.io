bardzo kontrowersyjny temat 

# Public API
Always version your API.
* can iterate
* always support at least 2 versions

There are mixed opinions around whether an API version should be included in the URL or in a header.


I'm a big fan of the approach that Stripe has taken to API versioning - the URL has a major version number (v1), but the API has date based sub-versions which can be chosen using a custom HTTP request header. In this case, the major version provides structural stability of the API as a whole while the sub-versions accounts for smaller changes (field deprecations, endpoint changes, etc).

curl https://api.stripe.com/v1/charges \
  -u sk_test_4eC39HqLyjWDarjtT1zdp7dc: \
  -H "Stripe-Version: 2019-05-16"
  
https://stripe.com/docs/api/versioning


An API is never going to be completely stable. Change is inevitable. What's important is how that change is managed. Well documented and announced multi-month deprecation schedules can be an acceptable practice for many APIs. It comes down to what is reasonable given the industry and possible consumers of the API.



---
https://medium.com/hashmapinc/rest-good-practices-for-api-design-881439796dc9
---
