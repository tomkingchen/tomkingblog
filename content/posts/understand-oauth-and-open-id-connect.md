---
title: 'Understand OAuth and Open ID Connect'
date: 2018-11-17T19:39:00.000-08:00
draft: false
url: /2018/11/understand-oauth-and-open-id-connect.html
---

IT world is always full of buzz words. “Digital Transformation”, “Automation”, “Blockchain”, “AI”, “Machine Learning”, etc, etc… We like to talk about them all the time, to show that we are not out of touch, we are up to date. Although I have to admit some of those words are used so often yet so few people have the really proper understanding of the actual technologies themselves. I believe OAuth is one of them. It’s a technology we use everyday, yet if you asked a lot of people why we need it and when to use, I bet not many will be able to explain (well). I hope this article can help you understand the technology better. After read it, hopefully you will at least have a basic idea of what OAuth is.

First, instead of providing a Wikipedia type definition for the term, let’s look at what particular technical issue OAuth helps us solve.

Like many other IT technologies, OAuth become widely adopted largely thanks for the prosperity of Smart Phones. It solves a key security issue came up while we use mobile apps on those phones.

**The Challenge 1**

Imaging this, a start up company created a mobile app which hosts cat videos (bad example, I know). Let’s call it CatTube… To help the app take off, you added a feature to send invitation to all Email contacts of a new subscriber.

To make the case a bit easier to understand, let’s assume the new user: Jack has a Gmail account. Now in order to access Jack’s Email contact list, CatTube need to somehow access Jack’s Gmail account. How can we do that?

Well, the easiest (dumbest) way is obviously to get Jack’s username and password for Gmail, and then CatTube will use this credential to log into Jack’s mailbox and retrieve his contact list. I know, this sounds ridiculous nowadays, right? But this is exactly what people were doing back then. Not only there is no technical guarantee that the credential will not be abused by the app provider, but what if their database got compromised, which never happens right? (check out [https://haveibeenpwned.com/](https://haveibeenpwned.com/ "https://haveibeenpwned.com/"))

So how can we delegate rights for CatTube, so it only retrieves the contact lists from Jack’s Gmail account and do nothing more or less?

**The Solution**

To solve this problem, we need a secure way to authorize CatTube’s delegation rights. This is where OAuth comes in. Let’s look at the below **OAuth Authorization Flow**.

[![image](https://lh3.googleusercontent.com/-MqDx1yevhOQ/W_D1gEeHWcI/AAAAAAAAKYQ/bmhVu0edv0kiq8oTRdefDFXWfLwl4RfXQCHMYCw/image_thumb?imgmax=800 "image")](https://lh3.googleusercontent.com/-v5SDMnnUO14/W_D1fEe_LeI/AAAAAAAAKYM/T8_wG-QDpiQmq2Bs0GawzTw3CfXOI4odwCHMYCw/s1600-h/image%255B2%255D)

A user click a button in CatTube app to allow it to access his/her Gmail contact list. After the button is clicked, the browser/mobile app send a request to Google servers holds the user accounts and asking for an authorization code for read contacts.

The initial request from the user contains following information

\- What is the type this request for: Response type: Code –> request an authorization code

\- What are the rights this request want: Scope

\- Where should the authorization code to send back to: Redirect URI

Google servers then respond by asking user to put his/her Google credential. Once the credential is verified, user will receive a confirmation prompt to confirm the delegation. After user clicked yes, Google generates the authorization code based on the request details and sends the **Authorization Code** to CatTube (through Redirect URI _CatTube.com/callback_).

After obtain the **Authorization Code**, CatTube **backend servers** send the **Authorization Code** back to Google to exchange for an **Access Code**. This is the actual code allows CatTube to do things. Google issues CatTube with the Access Code upon receiving the correct authorization code. CatTube **backend servers** then call upon Gmail API with this Access Code to retrieve the user Gmail contacts.

You may ask, why don’t we just issue the Access Code directly to the user, rather than going through these extra steps with 2 codes method (Authorization code and Access code).

Well, this is to prevent unauthorized access even when non-legit user got the authorization code.

If you look the diagram closely, you may notice I use two types of lines. The dot lines in the diagram represents communication between Google servers and CatTube servers. The solid line represents sessions between the user browsers/mobile app and servers.

To ensure Access Code will never be captured by the browser, it is always transferred in the backend. In case the authorization code is compromised, the unauthorized party will not be able to get the Access Code, as Google will only issue the code with CatTube servers.

[![image](https://lh3.googleusercontent.com/-1OGlG34TGkM/W_EvqMOg0XI/AAAAAAAAKZQ/L0dEH5bIhUEXd-sFjotxJEK-ZlLaDYKgACHMYCw/image_thumb%255B2%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-LxtDkq5eDa8/W_EvouDuhZI/AAAAAAAAKZM/ntTRUyFW_KcNfycxGdu5zqPBMDVXtUBmQCHMYCw/s1600-h/image%255B8%255D)

What about those apps do not have backend, like Angular, React? You can use a different type of OAuth Flow, called **Implicit**, which is for pure front channel authorization. With Implicit flow, the app set **Response type** as **token** when send request to the authorization server. The authorization server will then provide the Access Code (the token) directly through user browser session.

**The Challenge 2**

From the above user case, we can clearly see OAuth is a very good way to handle delegated authorization for applications. But what if we want to log into an application with another identity provider (like Facebook or Google)? In the above example, we got access to Jack’s Gmail contacts, but CatTube does not know or care who the user is. What if Jack wants to log back into CatTube with his Google account?

**The Solution**

OAuth originally is an authorization protocol, while it thrives in space of Web app delegated authorization, it is not designed for authentication purpose. To solve the second challenge, we need an authentication protocol designed for web apps. OpenID Connect is here to help us with that. It is an extension of OAuth 2.0, which means it is built on top of OAuth protocol.

To authenticate users, OpenID Connect use the 2 attributes below to capture user information.

\- ID Token: user information represent user id

\- UserInfo: provide more user information

Because OpenID Connect is built on top of OAuth, it uses the OAuth flow. The only difference is, the initial request will have **Scope** set as OpenID. Then in return, upon the Authorization server receives the authorization code, it will return with a ID Token as well.

Below is the OpenID Connect flow

[![image](https://lh3.googleusercontent.com/-kGVaaRf_Ct4/W_EygJ08viI/AAAAAAAAKZw/Z-hW4dqc5Tk-4JZ0FE6w-Tl0BEOyGxkiwCHMYCw/image_thumb%255B3%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-WkBybb47Mj8/W_EyfCgnD0I/AAAAAAAAKZs/FRkTTbnyTOMXkT28O6xXDp1yzS9rYN9-QCHMYCw/s1600-h/image%255B11%255D)

So here you go, that’s a quick run down of the basic concept of OAuth and OpenID Connect.

This article is mostly based on Nate Barbettini’s explanation of OAuth 2.0 in his [Youtube clip](https://www.youtube.com/watch?v=996OiexHze0&t=1098s). Hopefully, you find it is useful.