---
layout: post
title: Privacy trick - Using a VoIP number for Whatsapp
tags: twilio privacy whatsapp
excerpt: Twilio allows to rent any phone number in the world for $1/month, which can be used as your Whatsapp number.
---

# Problem
Phone numbers are public addresses and must be shared, for others to reach you. So they become a permanent record of 
sort, against your name. While having another phone number, for Whatsapp is possible, this has some side effects:
1. Getting a new number means a visit to the telecom store.
2. A SIM slot in the device is lost forever (OR) The SIM needs to be kept safely elsewhere between usages.
3. The new number will still be an Indian number (+91 prefixed).

[Twilio](https://twilio.com) provides a VoIP (Voice over IP) phone number, which could be rented from anywhere in the 
world, at a price of $1/month (in most geographies), without the above side effects. 

# Solution

## Pre-requisites
* A credit card that is usable internationally. 
* A functional local number to connect the rented number with.
* A functional email address, which will be the user name for the twilio account.

## Creating a functional Twilio account
1. A trial account creation is simple - Just click the "Sign up" button in [Twilio](https://twilio.com) and sign up
using the Email Id and the password. 
2. Twilio mandates 2FA (2nd Factor Authentication) for accounts after the trial period. The 2FA is done either via SMS
to the functional local number or via the [Authy](https://authy.com/) application.
3. Once done, you can pre-pay a certain amount using your credit card, depending upon how long you intend to use the 
VoIP number ($12 / year / VoIP number).
4. Projects are basic units in Twilio before any service provisioning happens. So we have to create a project, before 
renting a number.

## Renting a number
1. In the project dashboard, either click the "..." button (OR) go directly to the [Buy a phone number page](https://www.twilio.com/console/phone-numbers/search)
2. Choose the country in which you want the phone number and the capabilities you need on that phone number. For Whatsapp 
to work, the minimum capabilities needed are Voice and SMS.
3. While rental rates are normally $1 / month / number, in the United States, United Kingdom and in EU Countries, they are much
higher in other countries.
4. Once a number is chosen and bought, monthly rentals are automatically deducted from the pre-paid account.

## Receiving SMS and Voice calls
1. Unlike a phone number bought from a telecom store, incoming SMS and voice calls on twilio numbers are nominally
charged. Hence usage limits must be adhered to. 
2. The content of all incoming voice calls and SMS messages are stored for a very long time and can be accessed via 
the [Numbers dashboard](https://www.twilio.com/console/phone-numbers/incoming).

## Using the twilio number in Whatsapp
To use Whatsapp on the provisioned twilio number, the following steps are sufficient:
* Keep the [SMS Log page](https://www.twilio.com/console/sms/logs) open in your browser. 
* Download Whatsapp and input the provisioned number.
* Whatsapp will send the verification SMS to this number, which would show up in the SMS log page.
* Inputting the verification code in the phone, will make it your Whatsapp number.

# Caveats
* Please avoid using the above approach, if you find handling the extra complexity overwhelming. The complexity is only
worth, if you want an extra layer of privacy for your conversations and want to keep them isolated from your regular 
phone number. 
* Twilio numbers are de-provisioned from your account automatically, if you run out of $ and you can only get them
back, within a month by paying the rental. So using them as your primary Whatsapp number implies that you are paying a
life-long rental of $12 / year. 

