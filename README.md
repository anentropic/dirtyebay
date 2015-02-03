# dirtyebay

An eBay API client which respects (XSD) schema[*](caveat) and talks SOAP, but doesn't use Suds.

With a clean and simple interface and low memory usage.

```python
>>> from dirtyebay import ShoppingAPI
>>> client = ShoppingAPI()
>>> r = client.GetSingleItem(ItemID='321021906488', IncludeSelector='ItemSpecifics')
>>> r.Item.ItemID
321647696835
```

## Getting started

1. First you need to get yourself some developer keys [from eBay here](https://developer.ebay.com/DevZone/account/)
2. You need to make an `ebay.conf` file in the root of your project (or `export DIRTYEBAY_CONFIG_PATH=<path to conf>` in your shell)
3. Easiest way is to copy `ebay.conf-example` from this repo and fill in the blanks. `site_id` is the [code of the eBay site](http://developer.ebay.com/DevZone/XML/docs/WebHelp/FieldDifferences-Site_IDs.html) your profile is on.
4. Make an `ebay.sandbox.conf` file in the root of your project (or `export DIRTYEBAY_SANDBOX_CONFIG_PATH=<path to conf>` in your shell) if you want to use the ebay api sandbox

That's it. Well, you need to read the docs (eg [here's the Trading API](http://developer.ebay.com/DevZone/XML/docs/WebHelp/wwhelp/wwhimpl/js/html/wwhelp.htm?href=Overview-.html)) to see how to make the calls you want. 

All of the API classes take a `sandbox` kwarg. This is mostly only useful on the Trading API where your calls can have real effects! This will cause dirtyebay to use your sandbox conf file and the eBay sandbox API endpoints.

```python
from dirtyebay import TradingAPI
client = TradingAPI(sandbox=True)
client.GetItem(ItemID="321021906488")
```

### Why the name?

I previously made this: https://github.com/anentropic/ebaysuds

I thought it was nice that eBay provided a WSDL and Python had Suds and by putting the two together you get a very nice interface where responses are intelligently deserialized into appropriate python objects etc.

However the eBay TradingAPI WSDL is enormous (> 5MB) and the resulting memory usage of Suds was unacceptable (> 360MB). So ... no Suds => dirtyebay

There is an official eBay Python SDK _which you should probably just use instead_:  
https://github.com/timotheus/ebaysdk-python

They have a pretty nice interface and avoid using Suds and SOAP so memory usage is fine (~ 20MB on comparable calls). The trade-off is they don't make any use of the schema so deserialization is (neccessarily) best-effort and a bit crufty.


### Why make this then?

eBay provide an XSD for some services, and anyway a WSDL file typically contains a whole XSD schema embedded in it (along with descriptions of endpoints and possible calls)... so I started playing around with that and found to my surprise that using `lxml.objectify` in conjunction with the XSD schema[*](caveat) gives you all the nice deserialization like Suds did, but with < 10% the memory usage (~ 30-35MB). And it's pretty fast too.

This seemed like a good compromise. Also, I'd already made some stuff on top of `ebaysuds`, so I decided to do a rewrite that preserves the interface but uses XSD schema instead of Suds+WSDL hell. 

_<a name="caveat"></a>* despite the promise of schema-fied goodness, currently this is stymied by an [apparent bug in lxml](https://bugs.launchpad.net/lxml/+bug/1416853)._
