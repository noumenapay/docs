# docs


#### Card issued

- What are the required steps to get a card issued from scratch? 

A: KYC -> KYC Approved -> Apply for bank card -> Activate bank card

Open api access demonstration： [https://github.com/noumenapay/noumena-sdk-java](https://github.com/noumenapay/noumena-sdk-java)

- Which KYC Level is required to issue a card? What is the time schedule? How long it can take you to verify documents?

A: passport and passport piture.  about 24h.

- After successful verification of KYC documents we order a card for the use,How long it can take you to send the card? And can you notify us somehow that card has been sent?

A: After KYC successful, send card in 48h. User receives and activates the card in your APP.


 



#### API Submitting user's KYC data
url：/api/v1/customers/accounts
method：POST

- The parameter “acct_no” - Is a unique user’s ID on our side?   

A: Yes, It is your side user id.

- What is the meaning of the parameter “acct_name”? Which value should we submit in that parameter? 

A: Your side user name, depend you.

- The parameters “real_name” - Is the user's full name?  

A: Dont have this parameters

- What is the format of the parameter “birthday”?  

A: "1990-01-01"

- According to the API documentation the parameter “doc_type” can be “Passport”, “National ID card” and “Driving Licence” . Which values should we send for Residency Verification (level 2)? 

A：Only support Passport right now. Only need

- What is the meaning of the parameter “mix_doc” (Other clicked pictures)? What is required for?  

A: Photo with certificate in hand, Please fill in empty.

- What is “area_code”? Is it a mandatory parameter? And where can we find it?  

A: Has updated to "country_code".

- Which information must contain the parameter “address”, e.g country, postal code, city, address? 

A: The Detail the better, Bank card will send to this address.

- What is “kyc_info”, “mail_verification_code” and “mobile_verification_code”? Which information should we post in these parameters?

A: “kyc_info” is optional parameter, for other information. “mobile_verification_code”is filled by user, how send code to user's email please refers to "4.1. Sending Email verification code" in API doc.

- How can we use POST /api/v1/customers/accounts ? Which parameters we are required to submit for Level 1 KYC and for Level 2 KYC? And can we submit Level 1 KYC documents first, then order a card for the user and then after some period of time submit Level 2 KYC documents?

A: KYC should in one request. Example: https://github.com/noumenapay/noumena-sdk-java/blob/master/src/test/java/com/noumena/open/api/test/KycTest.java#L31



#### Submitting user's card information
url：/api/v1/debit-cards
method：POST
This API method allows us to order a card for users.

- There is only one input parameter “acct_no”. How can we select the required payment system: VISA or UnionPay?

A: You pay to Noumena in USDT, User how pay to you is depend you.

- A response contains the parameter “card_no”. Is it a masked or full PAN?

A: To prevent real card information from being exposed, the 'card_no' parameter is card id not card number. We return 'card_no' and 'card_number'.

- How to specify a shipping address for the card?

A: The address in KYC.

- Which delivery options are available: Express, Standard, etc? How can we select the required option?

A: Please ask our business colleagues

#### User activating bank card
url：/api/v1/debit-cards/status
method：PUT

- Will we receive card activation result synchronously or activation can take some time and we need to additionally query card activation result using the API GET /api/v1/debit-cards?acct_no={acct_no} ?

A:  After use receive card and activation. User input card number in your website and you call this API "2.2. User activating bank card", we will return if your user activation successful.

```text
url：/api/v1/debit-cards/status
method：PUT
```

- User triggers a card withdrawal password reset Email
url：/api/v1/debit-cards/deposit-pwd-emails?acct_no={acct_no}
method：POST
What is the purpose of a withdrawal password? Is it a 3ds password to be used for payment authorizations? 

A: For user reset the password when forgot.


- User deposit
url：/api/v1/deposit-transactions
method：POST
Card deposit process is not clear for us at the moment. As we were informed, each card will have a cryptocurrency address (e.g. BTC, USDT or ETH address), and to top up the card user must send crypto currency amount to that address. How can we use the API POST /api/v1/deposit-transactions in such flow?
How can we retrieve cryptocurrency addresses, assigned to the card?
When a user sends cryptocurrency to address provided, Your partners receive a cryptocurrency amount, convert it using a rate from coinmarketcap.com, and then send fiat amount to the user’s card. When can we know the finally converted fiat amount, which will be credited to the user’s card: when a cryptocurrency transaction has been registered in the network or when transaction has gained a certain amount of confirmations?
Can we somehow calculate fiat amount, which user will receive on his card, at the moment when user sends cryptocurrency amount?

A: This is we have done. We allocate cryptocurrency address (e.g. BTC, USDT) for you, "User deposit" is pay with your address, We calculate fiat amount and transfer fiat to user card, the query transaction details API is "GET /api/v1/deposit-transactions?acct_no={acct_no}".

- Query a deposit transaction status
url：/api/v1/deposit-transactions/{tx_id}/status
method：GET
{tx_id} - is a transaction ID on our side (can be value passed in the parameter “cust_tx_id” in the User deposit request POST /api/v1/deposit-transactions) ?

A: {tx_id} is not “cust_tx_id”, {tx_id} is in Noumena system when you deposit.

- Query all the deposit records
url：/api/v1/deposit-transactions
method：GET
Can we query all deposit records for all our cards using this api method?

A; Yes. you can do it.

- Query a particular user's deposit records
 url：/api/v1/deposit-transactions?acct_no={acct_no}
method：GET
How can we query card’s withdrawal transactions (POS, ATM, etc)? We need them to be displayed in the user's account, in transactions statement.
Can you send callbacks to our server on cards’ withdrawal transactions (POS, ATM, etc)?
Will users receive email / sms notifications about card’s withdrawal transactions (POS, ATM, etc)?

A: Get withdrawal transactions in "5.3 Check transaction records " API, notifications will support in future vertion.

```text
url：/api/v1/bank/transaction-record
method：POST
```
 
- Public API
POST /api/v1/emails/{email}/verification-codes
POST /api/v1/mobiles/{mobile}/verification-codes
PUT /api/v1/emails/{email}/verification-codes?code={code}&verification_id={verification_id}
PUT /api/v1/mobiles/{mobile}/verification-codes?code={code}&verification_id={verification_id}
Are SMS and Email verification required for our users? If yes, when should we query them, before submitting user's KYC data and card ordering or after?  

A: Mobile phone verification is not required. Email verification is optional. If you've done validation, you don't need to do it again in Noumena. How send code to user's email please refers to "4.1. Sending Email verification code" in API doc.

Card’s actual status retrieving. Card blocking / unblocking.
In the document I can’t find API for card blocking / unblocking and actual card status retrieving. Do you have them? We need to display actual card status in our account and allow users to block/unblock their cards if required. Also our support operators can block/unblock cards by users requests 

A: We will support blocking / unblocking card.

API to retrieve balance on a card
In the document I can’t find API for card balance retrieving. Do you have such an API? We need to display actual card balance in the user’s  account

A: In "5.2 Check account balance" API.

Do you have a test API environment? Can you please provide API URL and credentials?

A: Please ask our business colleagues.