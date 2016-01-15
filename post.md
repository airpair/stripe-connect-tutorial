# Summary
An essential building block of a successful marketplace is to process payments on behalf of others. Moreover, you want your users to list products and get paid instantly through your market. In this setting, you take a commission on each transaction. [Stripe Connect](https://stripe.com/connect) is an ideal solution to make this happen.

The process to setup such a framework is far from simple. While Stripe is probably the easiest solution out there, there are a lot of elements and pitfalls to tackle before you can have a working function.

I have therefore written this tutorial to give you a jump start and guide you through the complete setup. My estimate is that a novice developer would need about 1 hour to complete all the steps.

You can unlock the complete tutorial by [downloading the guide from this template](https://www.noodl.io/market/product/P201601151251248/). The package contains all you need to enable your users to accept money on behalf of others, while you subsequently collect fees in the process. The source code is provided in *NodeJS* and comes with an extensive documentation. This post can be therefore seen as a preview as it leaves out the code examples and the source code.

*Important: this tutorial is a follow up on [Stripe Charge](https://www.noodl.io/market/product/P201512041604740/), which comes with an elaborative documentation on how to setup Stripe and your servers. If you are not already familiar with processing payments using Stripe, then please complete [these instructions ](https://www.noodl.io/market/product/P201512041604740/) before proceeding with this tutorial.*

# Getting Started
The first step is to Register your platform on Stripe, which you can do by following the url:
`https://dashboard.stripe.com/account/applications/settings`, and pressing on **Connect** at the top. In this tutorial we will be working with a **Standalone Account**.

## Sidenote on Standalone Accounts versus Managed Accounts
From the docs: *A standalone Stripe account is a normal Stripe account that is controlled directly by the account holder (i.e. your platform’s user). A user with  standalone account will have a relationship with Stripe, be able to log in to the dashboard, can process charges on their own, and can disconnect their account from your platform.*.

This is definitely the recommended option if you are just starting out as it will significantly decrease your development process. To use the alternative, a **Managed Account**, you will need to implement many interactions, such as collecting all the information that Stripe needs. You might be interested in this option if you wish to make Stripe completely invisible to the account holder, but that is a tutorial on its own.

## Redirect URIs and Constants
An essential field in registering your Standalone Account is the Redirect URIs (define as `REDIRECT_URI`), which we will talk about later. This is essentially the field that Stripe uses to handle the callbacks after your user has authenticated their Stripe Application.

If you have setup Stripe before using [Stripe Charge](https://www.noodl.io/market/product/P201512041604740/), then you are already familiar with the setup. In that case, you can use your existing `SERVER_SIDE_URL` and add `/oauth/callback` at the end. If you have previously hosted your server-side on Heroku, then the final `REDIRECT_URI` would look something like:

```
[your-heroku-app-name].herokuapp.com/oauth/callback
```
[Add this url](https://dashboard.stripe.com/account/applications/settings) to both the development as production Redirect URIs.

If you have not setup the server-side before, then please follow the instructions in the section **Preparing the Server-Side** from the [Stripe Charge](https://www.noodl.io/market/product/P201512041604740/) template.

# Extending our Server-Side
Now that we have activated Stripe Connect, we need to extend our server-side code to cope with authenticating users and asking them for permission to connect their Stripe Account to your platform.

The first step is to add the `CLIENT_ID` (use development or production depending on yourother settings) to your server-side code, which you host on your `SERVER_SIDE_URL`. Basically add the following line of code at the top of your page:

```
var CLIENT_ID   = 'ca_<YOUR-CLIENT-ID>';
```

## Retrieving the Authorization Credentials
Next is to extend our server-side to retrieve the Authorization Credentials, which are basically the parameters that are used when your user is receiving a payment on behalf of your platform. Let's start by defining two new urls to authorize the user and to retrieve a token for retrieving the authorization credentials:
```
var TOKEN_URI     = 'https://connect.stripe.com/oauth/token';
var AUTHORIZE_URI = 'https://connect.stripe.com/oauth/authorize';
```

We proceed by creating an authorization route which can be called from the client-side, by adding this line of code:
```
// Code available in the guide after downloading this package
```

Then let's extend our routes by adding the callback (i.e. `REDIRECT_URI`) which will capture the `AUTHORIZATION_CODE`, which gives our platform the permission to connect the users Stripe Account and retrieve the Authorization Credentials. We do this as follows:
```
// Code available in the guide after downloading this package
```

## Store the Authorization Credentials in your Database
The data of interest in this part is the object `SCData`, the **Authorization Credentials**, on which we apply `JSON.parse()` such that we can **store it in our noSQL database**. In the complete source code I give an example of how to do this securely with Firebase, as it also requires retrieving a Firebase Authentication Token.

Now that our server-side is pre-configured, push the latest changes to your server such that we can consume it from the client-side.

# Client-side setup
It's time to implement our changes on the client-side. This is just a matter of redirecting the user to an appropriate route which we have defined on the server side. Let's define the following:
```
// Code available in the guide after downloading this package
```

Define this url in your **controller** as follows:

```
// Code available in the guide after downloading this package
```

This can be now called from your view by creating a link:
```
// Code available in the guide after downloading this package
```

## Firebase Token
Read this part only if you are using Firebase as your back-end. In order to store the Authorized Credentials of the user securely, it is recommended and required that these credentials are written to your database on the server. This means that your user somehow needs to be authenticated to your database provider on your server as well. When using Firebase, a good option is to generate authentication tokens from the client-side and pass these in `STRIPE_URL_AUTHORIZE`, which is called when the users attempts to connect their account to Stripe Connect. We can do this by extending the url as follows:

```
// Code available in the guide after downloading this package
```
Here `AuthData` is the object retrieved from monitoring the authentication state of the user.
In our setting, we generate `fbAuthToken` when the user enters a certain view, for which he needs to be authenticated already. I recommend doing this in the Account Settings of the user, where you also put the **Connect to Stripe button**. Note that I added the `ng-show=...` caption to make sure that the link is only shown when the token is actually loaded.

```
// Code available in the guide after downloading this package
```

with `generateFBAuthToken()` defined as a function in the service `FirebaseCheckout`:

```
// Code available in the guide after downloading this package
```

Here we are making a call to the url `STRIPE_FIREBASE_GEN_TOKEN` which is another route on our server side. I have included this part of the code in the full source code.

Now that you have succesfully generated a token, you can use this parameter on your client-side to authenticate the users and thus write the Authorized Credentials, stored in the object `SCData` (see above), to your database. I recommend writing it to a node as follows: `<YOUR-FIREBASE-URL>.firebaseio.com/stripe_connect_data/$userId`, thus separating it from an existing node that is open for public such as `<YOUR-FIREBASE-URL>.firebaseio.com/users`.

For safety, extend your Firebase security rules as follows:
```
// Code available in the guide after downloading this package
```

As it is now, everyone can read the `SCData`. We recommend therefore storing only fields that are essential (parsing it on the server-side), such as the destination account id. You could also consider encrypting this data or only providing access during sessions. This is to be covered in another tutorial.

# Processing Payments
We return again to our server-side as we need to do a final thing to make it possible to actually process the payments made on behalf of your users, and thus to charge a commission (or 'admission fee').

In the Stripe Charge tutorial we processed payments through the route `/charge` which we accessed by sending `$http POST` requests to `STRIPE_URL + "/charge"`. Let's rename this route to `/charge/nodestination` and then add the following line of code to handle payments with an admission fee:

```
// Code available in the guide after downloading this package
```

As you see we have two additional parameters, the `noodlioApplicationFee` expressed in cents, and `stripeDestinationAccountId`.The destination account id is one of the fields from the object `SCData` which you stored in the node `stripe_connect_data/$userId` before.

On the client-side, we need to change the way that we charge our clients in the controllers and services. A payment flow becomes then as follows:

- validate the credit card details and the `stripeToken` using Stripe Checkout (see Stripe Charge tutorial)
- retrieve the `stripeDestinationAccountId` from `stripe_connect_data/$userId`
- send a `$http POST` request with the additional parameters to the updated `STRIPE_URL + "/charge"`

This corresponds to the following changes in our services:
```
// Code available in the guide after downloading this package
```
```
// Code available in the guide after downloading this package
```

The only missing parameter in this example is `stripeToken`. You can get the source code of this function in the [Stripe Charge](https://www.noodl.io/market/product/P201512041604740/) starter in the file `services.js`.

# NodeJS Source code
Available for download [here](https://www.noodl.io/market/product/P201601151251248/)

# Questions or Feedback
That is all, you have now a working marketplace with Stripe Connect. If you have questions or issues, feel free to drop an email to noodlio@seipel-ibisevic.com, write/read your comments on [Noodlio](https://www.noodl.io/market/product/P201601151251248/), or on this website.
