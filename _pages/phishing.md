---
permalink: /portfolio/phishing/
author: Rick Theeuwes

defaults:
  # _pages
  - scope:
      path: ""
      type: pages
    values:
      layout: single
      author_profile: true
toc: true
title: Phishing test
---

In December Merlijn Vermeer and I hosted a phising test at an unnamed company.

## Preparation

We got in contact with the company via school in order to plan a test. We met up with the Security officer of the company and planned to execute a phishing test and a pentest. He would get in touch with the third party who hosts the system we would test for permission. As this went on we start preparing for the phishing test. After the phishing test we did not have any more time for the pentest, so that was unfortunately scrapped.

## The test

For the phishing test we setup `GoPhish` on a VPS. In GoPish we made a campaign and imported all the information of the employees we needed for the test. We got this information from the officer. The test was executed on approximately 60% of the employees. In here we build a mail template. The story is that the employee needs to change their password before it expires. We then catch the username in GoPhish, nothing is done with the password, it isn't even send back to us.

The mail we send to the employees looks like this:

![mail](https://cyber.merlijnvermeer.nl/images/companymail.png)

The mail has some clear indications that it is a phishing mail, such as the fact that the mail has no markdown, or a logo of the company. The mail has been sent from a domain with the same name as the company, but ending in `.ml` instead of `.nl`, and the link in the mail looks a bit suspicious.

The landing page that the victims go to looks like this:

![page](https://cyber.merlijnvermeer.nl/images/landingpage.png)

The page looks exactly like the real login page. This can be easily done using GoPhish's functions. After the users logged in, they were referred to the real login page of the company.

As an SMTP server, we first tried to use postfix this worked but some of the emails went into spam. This was mostly on office 365 outlook which the company is also using. To fix this we tried to get a real SMTP server. We have used Mailgun for a project we did last semester and know this could be a great solution. We added the needed DNS records and configured the SMTP. This way as far as we know for now all the emails ended up in the inbox instead of spam. We can send 20K emails, which is more than enough.

## Results

The mails were distributed over a week time, this caused some users to spread the mail to warn others that it is a phishing mail. Nonetheless, we got quite some clicks and credentials entered.

![result](https://cyber.merlijnvermeer.nl/images/phishingendresults.png)