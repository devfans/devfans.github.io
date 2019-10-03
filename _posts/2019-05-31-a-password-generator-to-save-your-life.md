---
layout: post
title:  "A Password Generator to Save Your Life"
author: devfans
categories: [ Password, Attack, Credential ]
image: /images.pexels.com/photos/162622/facebook-login-office-laptop-business-162622.jpeg?cs=srgb&dl=communication-connection-data-162622.jpg&fm=jpg
tags: [featured]
---
Living in modern society, most of the users of online services have to bear the pain to remember the access to plenty of platforms including websites, services, apps, bank account, email, etc. The login credentials cannot be too simple for security reasons, or too complex to remember it clearly. I believe everyone could lose some of the credentials for cannot recall it clear if he's keeping more than five quite different access passphrases. And if you are trying to use similar passwords for multiple services, you are possible to mess up with them easily and using similar passwords is not a good practice for security reasons, since you even don't know it if some services you ever used are hacked and your credentials are posted to public pages someday.

#### Is there a perfect solution for this?

As far as I know, not yet. But, there're some small security facilities that can probably ease the pain to remember all the credential and reduce the risk of access leaking.  The one I am gonna introduce in this post is `https://www.passwizard.com`, a small static page I wrote to serve as a password generator.

#### How does a password generator save your life

The idea behind `passwizard` is quite simple indeed. You feed in a simple passphrase to it and specify which account on which platform, then it will generate the repeatable random password table for you to select. It has some features that really matter for your concern:

+ Repeatable. Yes, if it can not repeat the password table with the same input, then every user of it is fucked up.

+ Static and No-Tracking. It's a static web page hosted on github which mainly a canvas operated by js code in the page, and it does not make any communication to anything else for the password generating process. You don't need to worry about your personal credentials are stored somewhere and might be leaked someday. No one is tracking the service and I wrapped the input interface into a canvas in the page which means it has a bit less risk of leaking if your computer or browser are hacked or listened by some methods. 

+ Randomness. It has to be random and less related to some common patterns. The `www.passwizard.com` implemented some tricks to reach that, so a single character change in the input would cause an absolute different password table generated.

+ Complexity. With randomness, it's possible the password generated could be quite simple in chances, that would not be something we expect, so there're other tricks in the implementation to get rid of it. Maybe the algorithm is not perfect yet, but it's working well. Also, the input fields support every kind of characters you can type with your keyboard. If it's not, well I need to figure it out with your feedback then.

#### Using `https://www.passwizard.com`

Heret o explain the interface of passwizard(Navigations: `enter` and `mouse cursor`):

![Passwizard]({{ site.baseurl }}/assets/images/a-password-generator-to-save-your-life-0.png)

+ Go to the site and it'll show two blank input fields. The first one is the passphrase you want to use, it could be a sentence or anything you can get from your mind. Press `enter` key to go to the next input, and the first field would be automatically hidden if not focused or hovered.

+ The second input field is for the account. It could be the username or an email address, or anything for there's no limitation. But the intention is for you to specify your user account identity. Press `enter` to jump to the next page.

+ On the next page, you can specify the platform or service name in the input field, and yes without any limitation. You may notice every time you change a character, the password table below will be changed totally.

+ Click any type of the password in the table to select the one you want to use. It's a copyable. You can use them now.

+ You need to remember the passphrase (The first input) and which password type you choose which each platform. This way, you can keep a single private passphrase and use it for every account on every platform with knowing which type of password you chose for each platform. And, yes you need to mark the passphrase into your mind clearly, and that's the single thing you truly need to take care of.

#### Using PassWizard Wechat mini-program

![PasswizardMiniProgram]({{ site.baseurl }}/assets/images/wechat.mini.PNG)

Scan below QR to try it:
![PasswizardQR]({{ site.baseurl }}/assets/images/passwizard.qr.PNG)


#### NOTES

+ The domain name is still good till now, but without assurance to keep using it. It's for sure it has a stable access address: `https://passwizard.github.io` and it's hosted at `https://github.com/passwizard/passwizard.github.io`.

+ If you are interested in optimizing the implementation of it or polishing it with brilliant new ideas, let me know.

Still, I am not a native English speaker, just trying to get used with it.
