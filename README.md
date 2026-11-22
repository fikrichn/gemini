Markdown
# google-ultra

Google One has a hidden 12-month free trial offer for AI Ultra that isn't displayed on any page—it only exists in the return values of the Google Play Billing Library (`BillingClient`). This script chains together "query offer → open Play checkout → confirm $0 → activate plan" into a single command.

The principle is to borrow Google One's own process to invoke `BillingClient`, requiring no Xposed and no APK modifications.

## What You Get

You get **Ultra 5x, 20TB, 12 months for $0**. Cancel before it expires and you won't be charged. Once activated, the Play subscriptions page will say "Starting 2027-11-17: US$99.99/month"—that is the price after the free trial ends, not an immediate charge.

The 30TB (20x) plan also shows an offer when queried, but Play runs an additional eligibility check during checkout, and most accounts will get "You're not eligible for the free trial." This script will not click/subscribe to that plan.

## Prerequisites

A rooted Android phone connected to your computer via USB. Magisk, KernelSU, and APatch all work, as do emulators.

Your computer needs Python 3.9 or higher, plus the following package:

```bash
pip3 install frida
Get adb from the Android SDK platform-tools. Once installed, running adb devices should show your phone. If you haven't installed it before, run brew install android-platform-tools on Mac, or on Windows download the platform-tools package, extract it, and add that folder to your PATH.

On your phone, enable Developer options → USB debugging, plug it in, and tap Allow on the prompt that pops up.

Usage
Run an environment self-check first. It will verify the device, root access, frida-server, push the billing module, and conveniently disable any proxies or Clash instances blocking Play's network:

Bash
python3 ultra0.py --check
Once everything shows green, you're ready to run your accounts.

Put your accounts into a text file, one account per line, with fields separated by four hyphens:

email----password----recovery email----2FA key----card number----expiration date----CVC----cardholder name----postal code
Trailing fields can be left blank. Newly purchased accounts usually don't have 2FA, so you can leave those empty:

abc123@gmail.com----mypassword----
With a card included:

abc123@gmail.com----mypassword--------4242424242424242----10/29----326----ZHANG SAN----97201
If the address is left blank, it defaults to 123 Main St, Portland, OR 97201 (Oregon has no sales tax).

Run:

Bash
python3 ultra0.py accounts.txt
You can also run a single account directly:

Bash
python3 ultra0.py --account "abc@gmail.com:password:2FA key"
To verify the result, open the subscriptions page to check:

Play Store → profile picture → Payments & subscriptions → Subscriptions
How It Works Under the Hood
Adds the account to the phone (adds it if not present, skips if already added).

Switches the active Play Store account. After switching, it reads back from the profile menu to verify—if it doesn't match, it exits immediately to avoid running on the wrong account.

Launches Google One, attaches via Frida, and uses DexClassLoader to load the billing module.

Calls queryProductDetailsAsync for g1.play.20tb, retrieving 5 offers.

Selects the offer with the longest free trial period (eft-12month, P1Y) and invokes launchBillingFlow on the main thread.

Monitors the checkout sheet. It only clicks subscribe if the price is 0 or says "12-month free trial"; if it sees "not eligible for the free trial," it closes it and never clicks subscribe.

Play occasionally prompts for password re-entry, which the script fills in automatically.

Opens the subscriptions page to verify and saves the results to ultra0-report/.

Troubleshooting Failures
Account not eligible. Play will say "You're not eligible for the free trial, though you can still subscribe." An account essentially only gets one shot: once a trial checkout is initiated—even if unconfirmed—the offer disappears. Therefore, do not test this on your main personal account.

No payment method. A card is required to subscribe. Play's add-card screen is a WebView that adb cannot auto-fill, so the script will pause and prompt you to finish adding the card on your phone, then press Enter. You only need to link a card manually once per device; afterward, all accounts on that device share the same card.

Play account failed to switch. The script will print whichever account is actually logged in and exit. Switch accounts manually in the Play Store, then re-run with --no-switch so it only verifies without attempting to switch again:

Bash
python3 ultra0.py accounts.txt --no-switch
Checkout screen doesn't appear. Run it one more time. If it fails twice in a row, check your device's network: run settings get global http_proxy—if it isn't null, clear it. VPNs like Clash hijack DNS, which can also prevent Play from connecting.

Parameters
Bash
--plan 5x      # only activate 20TB
--plan 20x     # only query 30TB (most accounts cannot claim the free trial)
--plan auto    # default; queries both plans and picks the longest free trial
--no-add       # account is already on the phone; skip the add-account step
--no-switch    # do not auto-switch accounts; only verify the current account
Notes
Remember to cancel the subscription in Google Play after testing, otherwise you will be charged $99.99 when the trial expires.

The script runs strictly locally on your machine; your emails, passwords, and card details are never sent over the network or uploaded anywhere. Report files are saved in ultra0-report/, containing offer details and step-by-step execution results.

This code simply leverages the fact that the client can fetch the trial offer within Play's standard billing flow—it does not modify any server-side data.

Discussion
If you run into any issues, join the Telegram group to ask: https://t.me/VQD_SHARE (@VQD_SHARE).
