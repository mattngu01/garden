
https://luzkan.github.io/smells/flag-argument

take the function `book(customer: Customer, is_premium: bool`. This seems very straightforward definition, but reading this call is confusing: `book("Matthew", true)`. You can't inherently tell unless you hover over for the definition.

This condition could also possibly split up into two functions instead: `premiumBook` and `regularBook`. This keeps the context of the method at the same level without having to read documentation / the function definition.