---
title: "Dirt Cheap Recurring Payments with Stripe and AWS Lambda"
---

If you run a small SaaS business, you need a recurring billing solution, and there are a lot of options out there. There's [Chargify](https://www.chargify.com/pricing/) (from $149/month), [Chargebee](https://www.chargebee.com/pricing/) (from $99/month), [Recurly](https://recurly.com/pricing/) (from $99/month), [MoonClerk](https://www.moonclerk.com/pricing/) (from $15/month), etc. All of these platforms have features beyond recurring payments, which increases their value props -- but what if all you want is a dirt cheap way to charge your customers each month? Here's how you can do it nearly for free by using Stripe Checkout and AWS Lambda. As a bonus, you can keep your site entirely static aside from the Lambda call (i.e. this is a "serverless" payments solution) -- which means that you can host it at zero cost on Github pages (or any other free static site server).

## Stripe Checkout

Stripe's [documentation for Checkout](https://stripe.com/docs/checkout/tutorial) is great, and if you're not familiar with it already, you may want to check it out for context. We're going to focus on just the steps necessary to set up a subscription plan.

On the client side, set up a Checkout form that looks like this. This form will post data to our lambda instance, which will ultimately create the subscription.

```html
<form action="PATH_TO_YOUR_LAMBDA_INSTANCE (will look something like this: https://123123123123.execute-api.us-west-2.amazonaws.com/production" method="POST">
  <script
    src="https://checkout.stripe.com/checkout.js" class="stripe-button"
    data-key="pk_live_YOUR_STRIPE_KEY"
    data-image="images/marketplace.png"
    data-name="Emma's Farm CSA"
    data-description="Subscription for 1 weekly box"
    data-amount="2000"
    data-label="Sign Up Now for $20/month!">
  </script>
</form>
```

There's no difference from a typical Checkout form; the only thing to note is that we're pointing it at an AWS Lambda instance (on which, more below).

## AWS Lambda

Now that Checkout on the client side is doing our job, all we have to do is set up little code on Lambda. While there are simpler ways to do it, I like to use [Zappa](https://github.com/Miserlou/Zappa) for this so that I can rely on Flask. If you go that route, your code will look like this:

```python
import os
import boto
from flask import Flask, request, redirect
import stripe

ENCRYPTED_STRIPE_KEY = os.environ['ENCRYPTED_STRIPE_KEY']
# Decrypt code should run once and variables stored outside of the function
# handler so that these are decrypted once per container
DECRYPTED = boto.client('kms') decrypt(CiphertextBlob=b64decode(ENCRYPTED_STRIPE_KEY))['Plaintext']
stripe.api_key = DECRYPTED

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def handler():
    try:
        customer = stripe.Customer.create(
            email=request.form.get('stripeEmail'),
            source=request.form.get('stripeToken')
        )

        stripe.Subscription.create(
            customer=customer.id,
            plan="base-plan"
        )

        return redirect("https://www.your-site/success-message")

    except Exception as e:
        # Don't forget to log your errors before doing this!
        return redirect("https://www.your-site/error-message")

if __name__ == "__main__":
    app.run()
```

Most of this is just the usual server-side code you'd write to create a subscription in Stripe (if your backend were Flask). The only part that's different is the key storage:

```python
ENCRYPTED_STRIPE_KEY = os.environ['ENCRYPTED_STRIPE_KEY']
# Decrypt code should run once and variables stored outside of the function
# handler so that these are decrypted once per container
DECRYPTED = boto.client('kms') decrypt(CiphertextBlob=b64decode(ENCRYPTED_STRIPE_KEY))['Plaintext']
stripe.api_key = DECRYPTED
```

All that's going on here is:

* We [create an environment variable](http://docs.aws.amazon.com/lambda/latest/dg/env_variables.html) in the AWS console called `ENCRYPTED_STRIPE_KEY`, and set it's value to our private Stripe key.
* We tell AWS (still using the console) to encrypt it.
* On the server side (the code above), we decrypt using Amazon's KMS (Key Management Service).

## Stripe Console

The last thing we have to do is to [create a new subscription](https://stripe.com/docs/subscriptions/quickstart) in Stripe's management console. To match with the code above, we call it `base-plan`.

## End Result

And that's it! You now have a simple system that will bill whomever completes your form on a recurring basis (whatever terms you set up for `base-plan`). The total cost for all this is:

* The 2.9% + \$0.30 that Stripe charges per transaction (but most recurring billing providers would charge you this on top of your monthly fees anyway).
* The cost of your lambda instance ([probably zero](https://aws.amazon.com/lambda/pricing/), since your first million requests per month are free).
* Your static hosting costs (also probably zero, if you use a free static server like Github pages).

Why is this useful? Let's say that you have a subscription service that mostly relies on other APIs. For example, maybe you want to create a paid-subscriber email list through MailChimp. Since MailChimp has an API, all you'd need to do is add a few lines of code to our Lambda snippet above to sign the user up for your mailing list. Consequently, you could run your business almost for free (free static hosting, free lambda, free tier of MailChimp). For a simple
subscription-model business, there's a tremendous amount of infra value that you can get for no cost.
