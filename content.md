# Email 📬

[Lecture Video](https://youtu.be/NIFtMCNlBss)

If you want to follow along:

- [Create a new blank rails application](https://github.com/new?template_name=rails-7-template&template_owner=appdev-projects)[](https://github.com/new?template_name=rails-7-template&template_owner=appdev-projects)
- Add Devise for User authentication
  - Add 'devise' to your `Gemfile`:

      ```ruby
        gem "devise"
      ```

  - `bundle install`
  - `rails g devise:install`

  - Generate `User` with devise: `rails g devise user`
    - Generate a `Task` scaffold: `rails g scaffold task user:references content:string`
    - Make the root route the tasks index page: `root "tasks#index"`.
    - Force users to sign in in `ApplicationController`: `before_action :authenticate_user!`

- Add [letter_opener](https://github.com/ryanb/letter_opener) for simulating sending emails
  - Add "letter_opener" to your Gemfile in the `development` group. (we only need it in development environment)

  ```ruby
  group :development do
    gem "letter_opener"
  end
  ```

  - `bundle install`
  - In `config/environments/development.rb`, add the following within the block:

  ```
  config.action_mailer.delivery_method = :letter_opener
  config.action_mailer.perform_deliveries = true
  ```

- Sign up for an account in your app.
- Sign out and go through the “Forgot password” flow. It should open a new tab with your email. 📧
- Caveats:
  - You'll need to purchase your own [Domain Name 🌐](https://dpi.instructure.com/courses/294/assignments/2085?wrap=1 "Domain Names 🌐") and sign up for an SMTP (simple mail transfer protocol) service to send real emails in production.
    - [Postmark](https://postmarkapp.com/) (my preferred and local to Chicago)
    - [Sendgrid](https://sendgrid.com/)
    - [Mailgun](https://mailgun.com/)
    - [Amazon Simple Email Service](https://aws.amazon.com/ses/)
  - Devise includes a bunch of mailers, but the really interesting part is creating your own with [ActionMailer](https://guides.rubyonrails.org/action_mailer_basics.html), which is sort of like controllers but for emails (SMTP) instead of web requests (HTTP).
  - A relatively new feature is [ActionMailbox](https://guides.rubyonrails.org/action_mailbox_basics.html), which allows you to easily parse incoming emails, like when you get GitHub notifications, email a receipt to Expensify or forward a flight ticket to TripIt at a custom email address that they assign to you. Lots of cool possibilities.
  - You can customize Devise mailers by running the view generator and adding your own styling

    `rails generate devise:views`  

Further Reading:

- [Choosing between plain text and html email](https://chamaileon.io/resources/choosing-between-plain-text-html-email/)
