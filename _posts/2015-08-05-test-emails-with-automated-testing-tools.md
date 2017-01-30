---
layout: post
title: Test emails with automated testing tools
---

[Ghost Inspector](https://ghostinspector.com/) calls it UI tests, [Selenium](http://www.seleniumhq.org/) calls it web browser automated tests, [Capybara](https://github.com/jnicklas/capybara) calls it acceptance tests. Despite how they call it, if you use a testing tool, that browses web pages and you would like to test the emails being sent during the tests, but you don't know how, this post is for you.

Most people (so do I), don't test the emails being sent or rely on mocks:

```ruby
expect(MyContactMailer).to receive(:send_contact).with('…')

click_on 'Send email'
```

Mocking can be fine, if you assume that the [framework to send emails](http://guides.rubyonrails.org/action_mailer_basics.html) works, and also that the [email configuration](http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration) is correct.

Can you assume that with 100% of confidence? I can't!

## How I test

I use Capybara for acceptance tests. These tests visit pages, trigger clicks and check the page content based on the performed actions.

```ruby
visit '/products'

click_on '#add-product'

fill_in 'product-title', with: 'Conference table'

click_on '#save'

expect(page).to have_content 'Product saved.'
expect(page).to have_content 'Conference table'
```

But besides Capybara, I also use Ghost Inspector. The main differences on how I use them are:

- I don't check the page content with Ghost Inspector, I don't write assertions. Ghost Inspector only tries to click on everything it can. Its goal is to check if the application isn't broken.
- With Capybara I test the current version of the application code, whereas with Ghost Inspector, I test environments. Every time I deploy to production, I execute the production suite against production URLs, same for other environments.

^ Usually through a [Slack webhook](https://api.slack.com/outgoing-webhooks) combined with the [Ghost Inspector API](https://ghostinspector.com/api/):

![ghost exec prod](/assets/images/posts/ghost-exec-prod.png)

## How I test emails

Long story short, check the video below:

<video width="512" height="384" controls=""><source ng-src="/assets/images/posts/putsbox-reset-password.mp4" type="video/mp4" src="/assets/images/posts/putsbox-reset-password.mp4"></video>

I'm using [PutsBox](http://putsbox.com) to test emails for real. It's a mix of disposable email tools and HTTP request recorders tools such as [PutsReq](http://putsreq.com).

In the video above I'm testing the Reset Password feature from PutsBox itself. The test tries to reset password using a @putsbox.com email, **opens the email for real, clicks on the reset link**, resets the password and finishes the test. The **highlighted** part is how PutsBox can be used for email testing.

Try it now! Send an email to what-you-want@putsbox.com, wait for a few seconds to give time to the email arrival, then check http://putsbox.com/what-you-want/inspect. No sign up required, it's free and [open source](https://github.com/phstc/putsbox).

Getting back to the test recorded above… As the [Ghost Inspector test used to record the video is a little verbose](https://gist.github.com/phstc/403ccf7b34a8c99c31f1), let's check an equivalent Capybara version:


```ruby
visit '/'

click_on 'a[href="/users/sign_in"]'
click_on 'a[href="/users/password/new"]'

fill_in '#user_email', with: 'test-reset-password@putsbox.com'

click_on 'input[name="commit"]'

# waits for the email
sleep 60

# opens the email
visit 'http://preview.putsbox.com/p/test-reset-password/last'

# clicks on the reset link
click_on 'body > p:nth-child(3) > a'

# resets the password
fill_in '#user_password', with: 'password'

fill_in '#user_password_confirmation', with: 'password'

click_on 'input[name="commit"]'
```

Awesome, right?!

## Conclusion

You can test emails being sent during your tests for real or not, it depends on your needs.

But if you do so, I recommend you to use something like Ghost Inspector or similar. I personally don't like having the tests that I run locally making external HTTP requests.