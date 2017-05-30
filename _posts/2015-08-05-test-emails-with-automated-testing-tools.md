---
layout: post
title: Test emails with automated testing tools
---

Usually, for automated tests that can send emails, because of a step in the test, we either stub or just don't check the emails being sent. We don't go to Gmail or any other email client for confirming (`assert`/`expect`) that.

Not testing emails, like for real, in most cases can be fine. For example, if you are using [Amazon SES](https://aws.amazon.com/ses/) or similar, you just trust them, and usually, the configuration you have in place for sending emails does not change often. If it worked once, it's most likely it will continue to work (famous last words).

But if the email delivery is a critical part of your application, and you do want to make sure it's working, this post is for you.

## Say Hello to PutsBox

Let me introduce you [PutsBox](http://putsbox.com), it's a mix of disposable email tools and HTTP request recorders tools such as [PutsReq](http://putsreq.com).

### How does this piece of joy work?

Send an email to `whatever-recipient-you-want-to@putsbox.com`, give it a few seconds (don't blame PutsBox, email delivery services may take a few seconds) then check `https://preview.putsbox.com/p/whatever-recipient-you-want-to/last`, voalá, it will respond with your email content. You can also append `.json` to that URL to get the JSON representation.

No sign up required, PutsBox is free and [open source](https://github.com/phstc/putsbox).

## Gimme an example!

Sure thing! Here's an example with [Capybara](https://github.com/jnicklas/capybara).

```ruby
visit '/'

click_on 'a[href="/users/sign_in"]'
click_on 'a[href="/users/password/new"]'

fill_in '#user_email', with: 'test-reset-password@putsbox.com'

click_on 'input[name="commit"]'

# Waits for the email. Email delivery services may take a few seconds
sleep 60

# Opens the email
visit 'http://preview.putsbox.com/p/test-reset-password/last'

# Clicks on the reset link (sent in the body of your email)
click_on '#reset-password'

# Resets the password
fill_in '#user_password', with: 'password'

fill_in '#user_password_confirmation', with: 'password'

click_on 'input[name="commit"]'
```

Cool, isn't?
