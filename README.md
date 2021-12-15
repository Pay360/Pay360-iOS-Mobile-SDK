# Pay360 iOS SDK

## Summary

This is iOS mobile SDK for Pay360 to integrate Pay360 to iOS mobile app.
It is avaibale for native iOS application(Objective-C/Swift) and other hybrid mobile framework for example React Native, Xamarin, etc.

## Prerequisites for running/building the project

- iOS 9+
- XCode 10+
- Apple Pay Credentials & Certificates

## Getting Started

Once you have a Bearer token from the Pay360 authentication service you can start to build a payment to process.

A payment consist of all information required to make payment :-

- Transaction - Details regarding the transaction (eg. Amount, Currency)
- Card - Card details
- Billing Address - Address of card holder
- Customer - Details of card holder
- Financial Service - Extra details of card holder
- Credentials - Merchant details required by Pay360

Once a Payment object has been populated you can then start the payment process.

There are currently 4 flows:

- processGuestPayment - Process Card Details & Stores Card Details if merchant ref supplied.
- requestVerify - Validates Card Details & Stores Card Details if merchant ref supplied.
- processTokenPayment - Processes a payment with the card token.
- processGooglePayResponse - Processes Google Pay payment, requires Google Pay

Examples of all flows can be seen in the sample app.

## Making a Payment

1. First need to create a Transaction containing all the details of the value of the payment.

```
  let transaction = PPOTransaction()
      transaction.currency = "GBP"
      transaction.amount = 100
      transaction.transactionDescription = "Sample Transaction"
      transaction.merchantRef = "YOUR MERCHANT REFERENCE"
      transaction.isRecurring = "false"
      transaction.isDeferred = "false"
```

2. Create the Card Details.

```
let card = PPOCard()
card.pan = is3DS ? "9903000000005131" : "9903000000000017"
card.cvv = "123"
card.expiry = "0117"
card.cardHolderName = "John Smith"
```

If making a payment with a card token then only the CV2 will required populating as all other card details are stored within Pay360.

```

let card = PPOCard()
card.cvv = "123"

```

3. Add Billing Address details. These are always required and must link to the card address.

```
    let billingAddress = PPOBillingAddress()
    billingAddress.line1 = "YOUR ADDRESS LINE 1"
    billingAddress.line2 = "YOUR ADDRESS LINE 2"
    billingAddress.line3 = "YOUR ADDRESS LINE 3"
    billingAddress.line4 = "YOUR ADDRESS LINE 4"
    billingAddress.city = "YOUR CITY"
    billingAddress.region = "YOUR REGION"
    billingAddress.postcode = "YOUR POSTCODE"
    billingAddress.countryCode = "GBR"

```

4. Add extra card holder details

```
    let financialService = PPOFinancialService()
    financialService.dateOfBirth = "12041979"
    financialService.surname = "YOUR SURNAME"
    financialService.accountNumber = "YOUR FINANCE ACCOUNT"
    financialService.postCode = "YOUR FINANCE POSTCODE"

```

5. Add the customer.

```

let customer = PPOCustomer()
customer.email = "YOUR EMAIL ADDRESS"
customer.dateOfBirth = "120479"
customer.telephone = "YOUR  TELEPHONE"

```

Include the merchant ref if you would card details stored and a card token returned.

```
customer.merchantRef = "YOUR CUSTOMER REFERENCE"
```

6. Add Credentials. This must include your client token from the MITE API and the installation id provided to you by Pay360.

```

    let credentials = PPOCredentials(clientToken: clientToken, installationId: self.installationId, environment: PPOEnvironmentTesting)

```

7. Build Payment ready for processing.

Once all payment detais have been taking you can build the payment object,
setting all values to the objects built.

```
let payment = PPOPayment.init(self)
payment.credentials = credential
payment.transaction = transaction
payment.card = card
payment.address = billingAddress
payment.customer = customer

```

The constructor of the payment takes in two arguements:

- `PPOPaymentDelegate` - You must implement this delegate class and pass as the first argument.
- `Context` - Android Application Context

8. Taking Payment

To start the payment process for a Guest Card can now be started by calling:

`payment.processGuestPayment()`

The SDK will then contact Pay360, display a 3DS window if required then proceed to attempt the payment calling the relevant complition method (failure/success) on completion.

```
self!.payment!.processGuestPayment { (data: PPOPaymentResponse) in
      print("\(data.processing.status) \(data.transaction.transactionId)");
  }  failure: { (error) in
    print(error.localizedDescription)
  }
```

- Make Payment with Token

To make a payment with the token, set the token to the payment and use the `processTokenPayment` method on payment:

```
payment.setNewCardToken("token");

```

Please note: The CV2 must also be included in the card details.

## Apple Pay

1. To use Apple Pay ensure your ViewController implements the PKPaymentAuthorizationViewControllerDelegate. The didAuthorizePayment delegate method is fired when the user has successfully authorised with Apple.

   On successfully authenticating with Pay360 we set the Apple payment and the Pay360 credentials and fire off the request to complete the payment with Pay360.

   Please note you need to manually dismiss the PKPaymentAuthorizationViewController.

```

func paymentAuthorizationViewController(\_ controller: PKPaymentAuthorizationViewController, didAuthorizePayment payment: PKPayment, handler completion: @escaping (PKPaymentAuthorizationResult) -> Void) {
    self.authentication(clientType: PPOClientGuest) { [weak self] credential, error in
        if (error != nil) {
            // handle
            return;
        }
        self!.payment!.setNewPassKit(payment);
        self!.payment!.setNewCredentials(credential!);
        self!.payment!.processApplePay { (data: PPOPaymentResponse) in
        controller.dismiss(animated: true) {
            print(data)
        }
        } failure: { (error) in
            print(error)
        }
    }
}

```

2. To start the Apple Pay flow select which networks to support and create a `PKPaymentRequest`.
   Initilise the ApplePay screen, set the delegate to the current ViewController and present the screen.
   The rest of the flow is handled in the delegate set in Step 1.

```

@IBAction func applePay () {
self.payment = self.buildPayment(PPOApplePayRequest, false);

    let paymentNetworks : [PKPaymentNetwork] = [.JCB, .discover, .visa, .masterCard, .amex];

    let paymentRequest = self.payment!.toPKPaymentRequest(self.merchantId, supportedNetworks: paymentNetworks);

    let applePayAuthController = PKPaymentAuthorizationViewController.init(paymentRequest: paymentRequest);

    applePayAuthController?.delegate = self;
    self.present(applePayAuthController!, animated: true, completion: nil);

}

```

```

```
