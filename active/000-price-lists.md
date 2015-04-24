- Start Date: 2015-04-24
- RFC PR:
- Spree Issue:

# Abstract

I'd like to move the current "price has a currency" logic to a more
generic "price belongs to a price list", to enable much more flexibility
without drawbacks.

# Motivation

I'm doing this because i needed this feature on some different projects and
the current spree prices/currency logic was a limit.

Use cases (actual sites, 2 on production and 2 on development):

* a fashion store needs a different price for each zone (actually, some brands
have a specific price list for some countries and they want to follow this).
they like to use the same base currency (EUR) on all lists to keep things simple.
payment gateway will do the currency conversion.

* a B2B store has 3 different price lists. each agent is assigned to a price
list. currency is the same for all lists.

* a B2B + B2C store have different prices for public clients and internal
agents. Currency is the same.

* a furniture store wants to use two different prices, one for their home
country and one for the rest of europe. currency is always EUR.


# Detailed Design

I have some code on https://github.com/freego/spree_price_lists
It's probably not production ready, it was extracted from a project and it's
missing tests. it can also be improved in many ways.

I think this is similar to https://github.com/spree-contrib/spree_price_books,
but more simple and generic.
I saw some very specific logics there that are probably extracted from some
project.

What i've done there is to create a simple `PriceList` model, which can just
have a `name` and a `currency`.

On Spree, a variant has many prices. This is a very good start, but it's also
very limited in my opinion because of the `currency` stuff.

Inside Spree, `currency` is just a string and is passed as parameter on tons of
methods, expecially during the checkout.
I want to change this to pass a price list instead.

This should not change any behavior for simple stores, because you'll just have
a default price list with your preferred currency.

This should not have performance problems because it's just one addition query
that can probably be cached.

This should enable flexibility to easily add any kind of custom logic for price
list selection.

For example, on the mentioned fashion store i get the user location from the IP
address and i load the right price list on the fly.
On the B2B store, i get the price list from `current_user`.
This is just a matter of overriding a method, which by default gets the default
price list.

# Drawbacks

I don't see many drawbacks: we don't need to *add* much code, we just have to
move all the currency stuff to price lists.

# Alternatives

I can just keep improving the extension.
I'm totally for many extensions and a simple spree core, but in this case i think
needed changes are too "low level".
I need many decorators to overwrite many important methods, and it's only to
switch from currency to price list.
Those methods also change often: i basically rewrited everything between 2.3 and
2.4, and need some more work for 3.0.

Another alternative i tried was to keep spree as is and to add "fake" currencies,
like "EU1", "EU2", all with the same symbol €.
The main problem here is that fake currencies doesn't work in the real world,
you can't pass them the the payment gateway, for example.

# Unresolved Questions

To be better at english.
