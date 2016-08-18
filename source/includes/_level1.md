# Use cases

## Retrieve a quote

The external service will ask a quote from a specific country to another one and for a specific amount in order to receive all the possible combinations to transfer the money  (ways of paying, ways of receiving and how much you would receive for each combination)

<img src="images/usecase-quote.png" />




# Entry points 

## Quotes

### Get quote [GET]

Name	|	Description
----- | ----
Path	|	/quote
Method	|	GET
Authentication	|	Optional
Description	|	Get options for a corridor and an amount


#### 1. request

Name	|	Type	|	Requirement	|	Description
----- | ---- | ----- | ----
countryFrom	|	string	|	Mandatory	|	Sending country
currencyFrom	|	string	|	Mandatory	|	Sending currency
countryTo	|	string	|	Mandatory	|	Reception country
currencyTo	|	string	|	Mandatory	|	Reception currency
paymentAmount	|	decimal	|	Conditional	|	Sending amount requested (expressed in currencyFrom)
receptionAmount	|	decimal	|	Conditional	|	Receiving amount requested (expressed in currencyTo)

<aside class="notice">
Either fill paymentAmount or receptionAmount.
</aside>


#### 2. Response

```shell
curl "http://example.com/api/v1.0/quote?
countryFrom=FR&currencyFrom=EUR&countryTo=MX&currencyTo=MXN
&paymentAmount=100"
```

> The above command returns JSON structured like this:

```json
{ 
  "rate" : 21.2332,
  "mtos" : {
   "transferwise" : {
      "options": [    {
          "optionId": "666f6f2d6261722d71757578",
          "paymentMethod":"LOCALBANK",
          "receptionMethod":"LOCALBANK",
          "isEstimatedAmount":false,
          "min":10,
          "max":1000000000,
          "optionDetails" :
              {
                  "paymentAmount":100,
                  "receptionAmount":1982.27,  
                  "rate":20.02299,
                  "quoteExpiration":12569537329,  
                  "fee":1,
                  "score": 8.663,
                  "time":{"b":true,"d":1,"receiveBy":"2016-08-12T15:03:47.070Z","fromToday":3}
              }
      } ]
     },
    "westernunion": {
      "options": [ {
          "optionId": "666f6f2d6261722d71757578",
          "paymentMethod":"LOCALBANK",
          "receptionMethod":"CASH",
          "isEstimatedAmount":false,
          "min":10,
          "max":1000000000,
          "optionDetails" :
              {
                  "paymentAmount":100,
                  "receptionAmount":1684.45,  
                  "rate":18.02299,
                  "quoteExpiration":12569537329,  
                  "fee":1,
                  "score": 5.1213,
                  "time":{"b":true,"d":1,"receiveBy":"2016-08-12T15:03:47.070Z","fromToday":3}
              }
         } ]
      }
  },
  "banks":{
      "averageBank" : {
         "options": [ {
          "paymentMethod":"LOCALBANK",
          "receptionMethod":"CASH",
          "isEstimatedAmount":false,
          "min":10,
          "max":1000000000,
          "optionDetails" :
              {
                  "paymentAmount":100,
                  "receptionAmount":1361.83,  
                  "rate":17.02299,
                  "quoteExpiration":12569537329,  
                  "fee":20,
                  "score": 3,
                  "time":{"b":true,"d":5,"receiveBy":"2016-08-16T15:03:47.070Z","fromToday":7}
              }
         } ]
        }
  },
 "errors": {
   "xendpay":"mto's data not reachable"
 }
}
```

Name | Type | Requirement | Description
----- | ---- | ----- | ----
mtos | MTOs JSON object | Optional | See below
banks | Banks JSON Object | Mandatory | An array containing one object
rate | decimal | Mandatory | The exchange rate
errors | Errors JSON Object | Optional | See below

* MTOs JSON object

Name | Type | Requirement | Description
----- | ---- | ----- | ----
transferwise | Solution JSON object | Optional | See below
westernunion | Solution JSON object | Optional | See below
... | ... | ... | ...

* Banks JSON object

For now, we only have an average bank information. We will add more solution in future versions

Name | Type | Requirement | Description
----- | ---- | ----- | ----
options | Array | Mandatory | Array of options provided

* Options

Name | Type | Requirement | Description
----- | ---- | ----- | ----
optionId	|	string		| Mandatory	|	Unique option id
paymentMethod	|	string		| Mandatory	|	Payment Option code(credit card, wire transfer…) -- see dictionnary
receptionMethod	|	string		| Mandatory	|	Reception Option code (Cash, wire transfer…) -- see dictionnary
isEstimatedAmount	|	boolean		| Mandatory	|	true, if the receptionAmount is estimated
isCalculated    |   boolean |   Optional    |   True if the Payment Amount or Reception Amount have been calculated. This can result is some differences compared to the MTO website
 | | | Example : Some MTO don’t give us their API but ways to calculate their rates
integrated  |   boolean |   Optional    |   True if Moneytis can take in charge the transaction
min	|	decimal		| Mandatory	|	Minimum paymentAmount for a transaction (expressed in currencyFrom)
max	|	decimal		| Mandatory	|	Maximum paymentAmount for a transaction (expressed in currencyFrom)
urgent	|	boolean		| Optional	|	True if the option is a “urgent" feature
who	|	integer		| Optional	|	1. if this solution is only available to send money to the user itself
 | | | 2. if this solution is only available to send to someone who is not the sender (ex : sending to an email)
 | | | 0. Available for both (by default)"
redirect_url    |   string  |   Mandatory   |   The url to redirect the user to the money transfer operator
optionDetails	|	Object		| Optional	|	All the main information for the option (fees, amounts). This object can be null if the amount asked is outside the limits (min - max)









* optionDetails						

Name | Type | Requirement | Description
----- | ---- | ----- | ----						
paymentAmount	|	decimal		| Mandatory	|	Amount to pay (expressed in currencyFrom)
receptionAmount	|	decimal		| Mandatory	|	Reception amount (expressed in currencyTo)
paymentAmountUser   |   decimal |   Optional    |   If the user is redirected to the MTO’s website, this variable document the amount to enter at the beginning of the process to pay “paymentAmount" at the end.
 | | | It has been added as some MTO’s add fees at the end of the process.
rate	|	decimal		| Mandatory	|	Exchange rate used by the mto
quoteExpiration	|	dateTime		| Mandatory	|	The time at which the quote will be considered expired
fee	|	decimal		| Mandatory	|	Fixed fees applied to the transaction
 | | | This fee is calculated with the formula : (paymentAmount - fee)*rate = receptionAmount
 | | | (expressed in currencyFrom)
time	|	object		| Mandatory	|	An estimation of the time required to receive the money-- see below
score    |   decimal    |   Optional    |   Score (between 0 and 10) given to the option regarding the amounts, time, and MTO’s easiness of use


* Time

Name | Type | Requirement | Description
----- | ---- | ----- | ----
d	|	integer	|	Mandatory	|	Number of days estimated between the transaction creation and the delivery
b	|	boolean	|	Mandatory	|	true if the days are business days
receiveBy | date  | Mandatory | Estimated date of the delivery
fromToday | integer | Mandatory | Number of days estimated between the transaction creation and the delivery with the business taking in count


* Errors JSON Object

Name | Type | Requirement | Description
----- | ---- | ----- | ----
xendpay   |   string |   Optional   |   Human readable message explaining the error
... | ... | ... | ...

