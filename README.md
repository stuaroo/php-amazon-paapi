php-amazon-paapi
======================

PHP class for Amazon's Product Advertising API allows you perform the following calls:

* ItemLookup: retrieve item information by Amazon ASIN
* CartCreate: create a virtual shopping cart using OfferId (from an ItemLookup) and Qty
* CartAdd: add items to an existing cart
* CartModify: change quantity of items in an existing cart
* CartGet: retrieve the contents of an existing cart

Each function will return a properly signed URL using your API public key, private key, and associate tag.  You can then use cURL, file_get_contents or whatever to retrieve the XML response.

## ItemLookup
The type of information returned by the ItemLookup operation depends on the Response Groups (RGs) specified. The default RGs are Offers and Images, which will return price and availability information as well as URLs to product images.  You can override the defaults by providing a comma-delimited string of RGs as the second parameter to the itemLookup() method.

```
$Amazon = new AmazonProductApi();
$items = array('0452284694','0072230665','0812550706');
```

Using default RGs:
```
$queryUrl = $Amazon->itemLookup($items);
```

Custom RGs:
```
$queryUrl = $Amazon->itemLookup($items, 'ItemAttributes,Images');
```

Once you have your signed URL, you can retrieve the response and load it into an object:
```
$request = file_get_contents($queryUrl);
$response = simplexml_load_string($request);
```

and display the info you retrieved:
```
foreach ( $response->Items->Item as $item ) 
{
    echo $item->ASIN;
    echo $item->Offers->Offer->OfferListing->Price->FormattedPrice;
    echo $item->ItemAttributes->Title;
    echo $item->LargeImage->URL;
    // etc.
}
```

## Cart Operations
When using the Offers RG with the ItemLookup operation, you can begin creating virtual shopping carts for Amazon.com.  You will use the OfferId, which is returned for each Offer on an ItemLookup, and a quantity to create a signed URL to add, delete, or retrieve a particular cart.  

For example, `cartCreate()` accepts an array of OfferIds and an array of Quantities.  This signed URL will then generate a cart and return cart identifiers like CartId, HMAC, and a PurchaseURL in the response.

```
$cartRequest = $Amazon->cartCreate($offerId, $qty);
$request = file_get_contents($cartRequest);
$response = simplexml_load_string($request);

$purchaseUrl = $response->Cart->PurchaseURL;
$cartId = $response->Cart->CartId;
// etc.
```

## More Resources
* [Product Advertising API Scratchpad](http://associates-amazon.s3.amazonaws.com/scratchpad/index.html) is great for testing which elements are returned by each RG
* [Product Advertising API Quick Reference Card](http://s3.amazonaws.com/awsdocs/Associates/2011-08-01/prod-adv-api-qrc-2011-08-01.pdf) lists required/optional parameters as well as relevant RGs for each operation
* [Amazon's Official Product Advertising API Developer Guide](http://docs.aws.amazon.com/AWSECommerceService/latest/DG/Welcome.html) in case you get lost
* or [Amazon Product Advertising API on APIhub](http://www.apihub.com/amazon/api/amazon-product-advertising-api/docs/getting-started) if you don't like the official docs and want to learn elsewhere