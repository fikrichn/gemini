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
