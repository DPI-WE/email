# Email in Rails Applications 📬
In this lesson, we'll explore how to incorporate email functionality into your Rails applications. This includes sending emails, simulating email sending in development, and understanding the basics of email protocols. We'll also touch on email customization, including views and controllers, and provide tips on avoiding spam filters. Let's dive in!

## Getting Started
Before we proceed, ensure you have a Rails environment ready for hands-on practice. If not, create a new blank Rails application and follow the steps outlined below to set up user authentication with Devise and email simulation with [Letter Opener](https://github.com/ryanb/letter_opener).

## Set Up User Authentication with Devise

Add Devise to Your Gemfile:

```ruby
gem "devise"
```
Run `bundle install` to install the gem, and then `rails g devise:install` to set up Devise in your application.

Use Devise to generate a User model with `rails g devise user`.

## Add a Task Model
Create a Task scaffold to associate tasks with users:

```ruby
rails g scaffold task user:references content:string
```

Make tasks index page the root route and ensure users are authenticated before accessing the application:

```ruby
# config/routes.rb
root "tasks#index"
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :authenticate_user!
end
```

## Simulating Email Sending with Letter Opener
To simulate email sending in development, add [Letter Opener](https://github.com/ryanb/letter_opener) to your Gemfile under the development group:

```ruby
# Gemfile
group :development do
  gem "letter_opener"
end
```

Then, configure your development environment to use Letter Opener:

```ruby
# config/environments/development.rb
config.action_mailer.delivery_method = :letter_opener
config.action_mailer.perform_deliveries = true
```

### Forgot Password Email
Now we're ready to test out our "forgot password" flow.

1. Sign up for an account in your app.
2. Sign out and go through the “Forgot password” flow.
3. It should open a new tab with your email. 📧

<!-- TODO: add screenshot/gif of forgot password email pop up -->

<aside>
You can use `rails g devise:views` to customize devise views and mailers.
</aside>

### Creating a Task Mailer
Now, we'll need to create a mailer for tasks. Rails mailers work similarly to controllers, but they're used for sending emails instead of responding to HTTP requests.

#### Generate a Mailer

```bash
rails g mailer TaskMailer task_created
```

This command generates a TaskMailer with an email action named task_created. Rails will automatically create views for this mailer under `app/views/task_mailer`.

#### Define the Task Created Email

Open the generated mailer file `app/mailers/task_mailer.rb` and define the `task_created` action. This method should accept a task as an argument and use it to set up email content.

```ruby
# app/mailers/task_mailer.rb
class TaskMailer < ApplicationMailer
  def task_created(task)
    @task = task
    @user = task.user
    mail(to: @user.email, subject: 'New Task Created')
  end
end
```

#### Create the Email View:
Next, set up the email view for the task_created action. Open `app/views/task_mailer/task_created.html.erb` (or create it if it doesn't exist) and add the following content:

```html
<!-- app/views/task_mailer/task_created.html.erb -->
<h1>New Task Created</h1>
<p>Hello <%= @user.email %>,</p>
<p>A new task has been created: <%= @task.content %></p>
```

#### Send an Email After Task Creation
Now, integrate the mailer with your `Task` model to send an email after a new task is created using the `after_create` callback. Open your Task model file `app/models/task.rb` and add the `after_create` callback to trigger the TaskMailer.

```ruby
# app/models/task.rb
class Task < ApplicationRecord
  belongs_to :user
  after_create :send_task_created_email

  private

  def send_task_created_email
    TaskMailer.task_created(self).deliver_now
  end
end
```

This callback calls the `send_task_created_email` method, which in turn uses `TaskMailer` to send the email asynchronously with `deliver_now`. 

<aside>
If you have setup background jobs, you can use the `deliver_later` method to move the email sending logic to a queue. This ensures that the email sending process doesn't block or slow down the web request.
</aside>

#### Test Your Task Created Email
With everything in place, you're now ready to test the email functionality.

1. Start your Rails server and sign in to your application.
2. Create a new task.
3. Check your development environment's default mail viewer (Letter Opener's web interface, if you're using it) to see the email generated for the new task.

This process integrates email notifications seamlessly into your application workflow, enhancing the user experience by keeping them informed about new tasks. Remember, while testing in development with Letter Opener is straightforward, you'll need to configure your production environment to use a real email service provider for sending emails to actual users.

## Sending Real Emails in Production
For sending real emails in a production environment, you'll need a [domain name](https://learn.firstdraft.com/lessons/313-rails-domain-names) and an SMTP service. Consider services like [Postmark](https://postmarkapp.com/), [Sendgrid](https://sendgrid.com/), [Mailgun](https://mailgun.com/), or [Amazon Simple Email Service](https://aws.amazon.com/ses/). Each service offers different features and pricing, so choose one that best fits your needs.

## Deep Dive into ActionMailer and ActionMailbox
[ActionMailer](https://guides.rubyonrails.org/action_mailer_basics.html) and [ActionMailbox](https://guides.rubyonrails.org/action_mailbox_basics.html) are powerful tools in Rails that handle outbound and inbound emails, respectively.

**ActionMailer**: Think of it as a controller for emails. It allows you to create mailer classes and views that manage the content and delivery of emails.

**ActionMailbox**: This newer feature parses incoming emails, enabling functionalities such as processing replies or handling attachments automatically, like when you get GitHub notifications, email a receipt to Expensify or forward a flight ticket to TripIt at a custom email address that they assign to you. Lots of cool possibilities.

## Best Practices

### Avoiding Spam Filters
To ensure your emails reach the inbox:

- Use a reputable email sending service.
- Keep your email lists clean.
- Avoid spammy content.
- Follow best practices for email authentication (SPF, DKIM, and DMARC).

### How Sender Reputation Works

Internet and email service providers monitor and evaluate the behavior of emails coming from specific domains and IP addresses. They use sophisticated algorithms to assign a reputation score based on observed sending habits and the responses of email recipients. The reputation score is dynamic, meaning it can change based on recent sending activities. It's also unique to each internet and email service provider, so an email sender might have different reputation scores with Gmail, Yahoo, Outlook, and others. A good sender reputation means emails are more likely to be delivered to the intended recipient's inbox. A poor reputation can lead to emails being filtered into spam folders or blocked outright.

### Factors Affecting Sender Reputation

Sender reputation is essentially the trustworthiness of your email sending practices in the eyes of internet and email service providers. Maintaining a good sender reputation is crucial for ensuring your emails reach your audience effectively. Here are some factors that affect sender reputation:

- **Volume of Emails**: A sudden spike in email volume can trigger red flags for internet service providers, as it's often indicative of spamming activity.

- **Bounce Rates**: A high rate of email bounces, especially hard bounces (permanent delivery failures), negatively impacts your reputation. It suggests poor list hygiene or outdated email lists.

- **Spam Complaints**: If recipients frequently mark your emails as spam, it severely damages your reputation. Keeping spam complaints low is crucial for maintaining a good standing.

- **Engagement Rates**: Internet service providers also look at positive engagement metrics, such as open rates, reply rates, and forwarding rates. High engagement can improve your sender reputation.

- **Spam Trap Hits**: Sending emails to spam trap addresses (email addresses created specifically to catch spammers) can significantly harm your reputation. It indicates that you're either purchasing email lists or not maintaining list hygiene.

- **Authentication Records**: Implementing email authentication protocols like SPF, DKIM, and DMARC helps build trust with internet service providers and can positively influence your sender reputation.

- **Content Quality**: The relevance and quality of your email content play a role. Content that triggers spam filters or is consistently low-quality can hurt your reputation.

### Managing and Improving Sender Reputation

- **Monitor Your Metrics**: Keep an eye on delivery rates, bounce rates, spam complaints, and engagement metrics. Many email marketing platforms provide these analytics.

- **Maintain List Hygiene**: Regularly clean your email list to remove inactive or invalid addresses and manage hard bounces effectively.

- **Follow Best Practices**: Ensure your emails provide value, seek permission before sending (opt-in), and make it easy for recipients to unsubscribe.

- **Warm Up Your IP**: If you're using a new IP address for sending emails, gradually increase your sending volume to build a positive reputation over time.

- **Use Email Authentication**: Implement SPF, DKIM, and DMARC to authenticate your emails and protect against spoofing and phishing.

### Security (SPF, DKIM, and DMARC)

SPF, DKIM, and DMARC are email authentication methods that help protect your domain from being used for email spoofing, phishing attacks, and other forms of abuse. Implementing these standards is crucial for ensuring your emails are delivered reliably and for maintaining your domain's reputation. Let's delve into each of these methods:

#### SPF (Sender Policy Framework)

SPF is an email authentication method that allows domain owners to specify which mail servers are authorized to send emails on behalf of their domain. When an email is received, the receiving mail server checks the DNS records of the sender's domain for an SPF record. This record lists the authorized sending IP addresses. If the email's source IP matches one in the SPF record, it's considered a legitimate email. Otherwise, it may be marked as spam or rejected. SPF helps to prevent email spoofing by ensuring that only authorized servers can send emails from your domain. It reduces the chance of your domain being blacklisted due to spam or phishing activities conducted by unauthorized senders.

Example SPF record

|Type|Host|Value|
|----|----|-----|
|TXT Record|@|v=spf1 include:_spf.google.com ~all|

#### DKIM (DomainKeys Identified Mail)

DKIM provides a way to validate a domain name identity that is associated with a message through cryptographic authentication. DKIM uses a pair of cryptographic keys, one private and one public. The sending server attaches a digital signature to the email headers using the private key. The receiving server then retrieves the public key from the sender's DNS records and uses it to verify the signature. If the signature is valid, it confirms the email was not tampered with in transit and that it indeed comes from the specified domain. DKIM ensures the integrity and authenticity of an email, significantly reducing the risk of email being altered or forged. It helps improve email deliverability and protects the domain's reputation.

Example DKIM record

|Type|Host|Value|
|----|----|-----|
|TXT Record|selector._domainkey.yourdomain.com|k=rsa; p=MIGfMA...|

#### DMARC (Domain-based Message Authentication, Reporting, and Conformance)

DMARC builds upon SPF and DKIM. It allows domain owners to specify how an email should be handled if it fails SPF or DKIM checks. Additionally, it provides a way for email receivers to report back to the sender about messages that pass or fail DMARC evaluation. Domain owners publish a DMARC policy in their DNS records, indicating their preferences for email handling (e.g., reject, quarantine, or allow) and reporting. When receiving an email, the mail server checks it against the DMARC policy. Based on the results and the policy's instructions, it decides whether to deliver, quarantine, or reject the email. DMARC allows domain owners to protect their domain from unauthorized use, reduce the risk of phishing attacks, and increase the deliverability of legitimate emails. It also provides visibility into email flow, enabling domain owners to identify and address authentication issues and abuse.

Example DMARC record

|Type|Host|Value|
|----|----|-----|
|TXT Record|_dmarc|v=DMARC1; p=quarantine; pct=100; rua=mailto:dmarc-reports@yourdomain.com;|

## Managing Subscriptions and Unsubscribes

### Importance of Subscription Management
- **Legal Compliance**: Ensuring users can easily unsubscribe from email communications is a legal requirement in many jurisdictions. It respects user preferences and protects their right to privacy.
- **Sender Reputation**: Properly managing unsubscribes can improve your sender reputation. ISPs monitor how well senders respect unsubscribe requests. Ignoring these requests can lead to increased spam complaints and harm your sender reputation.
- **User Experience**: Providing a straightforward way to unsubscribe improves the overall user experience. It shows that you value your recipients' choices and can reduce the likelihood of your emails being marked as spam.

### Using Mailkick Gem for Subscription Management
The [Mailkick](https://github.com/ankane/mailkick) gem is a powerful tool for managing email subscriptions and unsubscribes in Rails applications. It provides a simple interface for subscription management and integrates seamlessly with various email services.

## Conclusion
By incorporating email functionality into your Rails applications, you can enhance user interaction and engagement. Whether it's through user authentication emails, notifications, or marketing messages, email remains a vital communication tool in web applications.

## Further Reading
- [Plain Text vs. HTML Email](https://chamaileon.io/resources/choosing-between-plain-text-html-email/): Understand the differences and when to use each format.
- [ActionMailer Basics](https://guides.rubyonrails.org/action_mailer_basics.html): A guide to starting with ActionMailer.
- [ActionMailbox Basics](https://guides.rubyonrails.org/action_mailbox_basics.html): Learn how to work with incoming emails in Rails.
- [Mailkick: Email subscriptions for Rails](https://github.com/ankane/mailkick)
- [What are DMARC, DKIM, and SPF?](https://www.cloudflare.com/learning/email-security/dmarc-dkim-spf/)
