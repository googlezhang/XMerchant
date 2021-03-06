XMerchant
=========

XMerchant is a set of .Net class libraries, built against the .Net 4.0 Framework, for easing integration with various
online payments providers.

Current Status
--------------

The library currently supports a partial implementation of PayPal's Website Payments Standard gateway, and can handle
IPN notifications and create subscription requests. Encrypted Website Payments (EWP) is also supported.

Sample Code (ASP.Net MVC 2)
---------------------------

	public ActionResult TestPayPal()
	{
		var settings = new PayPalConfigSettings();
		var sub = new PaypalSubscriptionRequest(settings)
		{
			ItemName = "Test Subscription 1",
			ItemNumber = "TSUB1",
			CustomValue = "Banana",
		};
		var values = sub.GetValues();
		var encryptedValues = PayPalEncryptedWebsitePayments.Encrypt(sub.GetValues(), settings);

		using(var writer = new StringWriter())
		{
			using(var form = new PayPalFormWriter(writer).BeginForm(values, settings))
				form.Write(@"<input type=""submit"" value=""Subscribe with PayPal"" />");

			using(var form = new PayPalFormWriter(writer).BeginForm(encryptedValues, settings))
				form.Write(@"<input type=""submit"" value=""Subscribe with PayPal (Encrypted)"" />");

			return Content(writer.ToString());
		}
	}

Application Configuration File
------------------------------

If you want to store your PayPal preferences (account email, return url, etc.) in your application configuration file,
the `PayPalConfigSettings` class will automatically read them for you. Alternatively, to manually specify settings,
create an instance of `PayPalCustomSettings` and set the properties as needed. Here is a sample set of appSettings
values:

		<add key="PayPal.Account" value="mypaypal@mywebsite.com"/>
		<add key="PayPal.EWP.RecipientPublicCertPath" value="~/App_Data/paypal_cert_pem.txt"/>
		<add key="PayPal.EWP.SignerPfxPath" value="~/App_Data/paypal-cert.p12"/>
		<add key="PayPal.EWP.SignerPfxPassword" value="mypassword"/>
		<add key="PayPal.EWP.CertID" value="BSNEUEX6SGCQJ"/>
		<add key="PayPal.NotifyUrl" value="http://ipn.mywebsite.com"/>
		<add key="PayPal.ReturnUrl" value="/app/thanks"/>
		<add key="PayPal.CancelUrl" value="~/cancelled"/>
		<add key="PayPal.LogoUrl" value="~/images/logo.png"/>
		<add key="PayPal.TestMode" value="false"/>

You can ignore the EWP values if you're not using Encrypted Website Payments. The only required value is
`PayPal.Account`, without which PayPal won't know which account should receive the payment. See
[this post](http://forums.asp.net/p/1236969/2248178.aspx) for a bit more information on setting up your EWP values.

How to Process IPN Notifications (ASP.Net MVC 2)
------------------------------------------------

Make sure you specify in your initial PayPal transaction request that the return URL points at your IPN controller. The
code below has three important items: (1) The model binder must be registered statically, (2) The AuthenticateIPN
attribute should be on the IPN action (unless you want to test IPN notifications from an source other than PayPal) and
(3) your IPN action should accept a single parameter of type IPayPalTransaction which you can cast to the appropriate
type after checking the TransactionType property.

	public class PayPalController : Controller
	{
		static PayPalController()
		{
			PayPalModelBinder.Register();
		}

		[AuthenticateIPN]
		public void IPN(PayPalTransaction trans)
		{
			switch(trans.TransactionType)
			{
				// process here...
			}
		}
	}

Future Plans
------------
Implement support for:

* PayPal Website Payments Standard (All possible interactions)
* PayPal Website Payments Pro
* PayPal Payflow Gateway?
* Amazon Flexible Payments Service? (they only allow USA developers at this time)
* Google Checkout
* Authorize.net
* Recurly?
* Chargify?
* Spreedly?
* CheddarGetter?
* ... who else?

I'm very open to collaboration with others in order to build this library up, so if you want to fork it and notify me of
your additions, I'll be very happy to consider your code for inclusion in the trunk. It would be really cool if over
time this could become a comprehensive library of integrations with different payment gateways.
