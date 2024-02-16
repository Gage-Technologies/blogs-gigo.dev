11-17-2023
## How GIGO integrated the Mailgun API into the platform

## Introduction

As GIGO evolved into a more comprehensive platform for developers, the need for effective and reliable email communication became increasingly apparent. Our platform’s growth necessitated a dependable email service that could cater to a variety of crucial tasks. Email communication at GIGO isn’t the typical promotional spam messages; it’s an integral tool for user engagement, platform functionality, and user notifications. Recognizing these needs, finding an email service provider that was reliable, scalable, and could integrate smoothly with our existing setup was imperative.

## Why GIGO chose Mailgun

Our decision to integrate Mailgun into GIGO was driven by specific, practical needs. As the developer tasked with setting up GIGO’s email functionality, my focus was on finding an email service that could seamlessly fit into our existing infrastructure and scale with our growth. Mailgun emerged as the most suitable choice, primarily due to its robust API, reliability, and scalability.
Mailgun’s compatibility with our existing tech stack was a key factor. The platform supports various programming languages such as Python, Ruby, PHP, and most importantly Go, which aligns well with our tech stack at GIGO. This versatility made it easier to integrate Mailgun into our existing systems without significant overhauls.
Scalability was also a major consideration. Mailgun’s ability to handle substantial volumes of emails — up to 15 million per hour — meant that it could grow alongside GIGO. This, coupled with its flexible pricing model, offered us the scalability needed for our expanding platform.

## How GIGO integrated Mailguns API

As the developer responsible for implementing GIGO’s email functionality, I want to share the process I undertook to integrate Mailgun’s API into our platform.

### Setting Up the Mailgun Client

The first step in this integration involved establishing the Mailgun client. This was crucial as it acted as the main access point for all email functionalities within GIGO. The setup required creating a new instance of the Mailgun client with our specific Mailgun domain and API key. Here’s a snippet of the code used:

    func SendMonthInactiveMessage(ctx context.Context, mailGunKey string, mailGunDomain string, recipient string) error {
     // create new Mailgun client
     mg := mailgun.NewMailgun(mailGunDomain, mailGunKey)
    
     return nil
    }

In this function, by passing the mailGunKey and mailGunDomain, I successfully instantiated the Mailgun client.

### Validating User Emails Before Sending

Ensuring the validity of email addresses before sending emails was a crucial step. To tackle this, I implemented a multi-tiered validation process. This began with basic checks like verifying the email’s length and format using regex checks, and then utilizing Mailgun’s built-in email validation functionality. These preliminary checks were important to avoid unnecessary costs associated with the Mailgun validator.

The following code illustrates the multi-tiered email validation process:

    func EmailVerification(ctx context.Context, mailGunVerificationKey string, address string) (map[string]interface{}, error) {
     // perform simple email check before advanced validation
     if address == "" || len(address) > 511 {
      return map[string]interface{}{"valid": false}, fmt.Errorf("email cannot be empty")
     }
    
     // Basic email validation using regex
     re := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
     if !re.MatchString(address) {
      return map[string]interface{}{"valid": false}, fmt.Errorf("email was invalid")
     }
    
     // create new Mailgun validator client
     validator := mailgun.NewEmailValidator(mailGunVerificationKey)
    
     ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
     defer cancel()
    
     // validate email address via Mailgun
     email, err := validator.ValidateEmail(ctx, address, true)
     if err != nil {
      return map[string]interface{}{"valid": false}, fmt.Errorf("error while attempting to validate email with mailgun validator client : %v", err)
     }
    
     // check if the email is valid and not disposable
     if email.IsValid != true {
      return map[string]interface{}{"valid": false}, nil
     } else if email.IsDisposableAddress {
      return map[string]interface{}{"valid": false}, nil
     }
    
     // flag to hold MailboxVerification
     flag := false
    
     if email.MailboxVerification == "unknown" || email.MailboxVerification == "false" {
      flag = false
     } else if email.MailboxVerification == "true" {
      flag = true
     } else {
      return map[string]interface{}{"valid": false}, fmt.Errorf("MailboxVerification not true, false, or unknown")
     }
    
     return map[string]interface{}{"valid": flag}, nil
    }

In this function, I start with basic string checks, progress to regex validation, and finally, use Mailgun’s advanced validation to ensure that the email is not only correctly formatted but also valid and not disposable.

### Setting up the email template

After setting up the Mailgun client and ensuring robust email validation, my next focus was on crafting the email templates. The objective was to create templates that were not only visually appealing but also concise and informative. I uploaded the HTML code for each template and tested them rigorously to ensure they rendered correctly across different email clients and devices. For brevity, the following example focuses on the core structure of the email template, omitting styling details.

    <!DOCTYPE html>
    <html>
    <head>
    </head>
    <body>
    
    <div class="container">
      <!-- Title -->
      <div class="title">Don't Forget About GIGO Dev</div>
      <!-- Main Image -->
      <img src="https://api.gigo.dev/static/ui/sad-gigo.png" alt="GIGO Image" style="max-width:90%;height:auto;margin-top:20px;">
        <div class="content">
      <!-- Content Text -->
    <div class="content">
      <p>Exciting news! GIGO has undergone significant enhancements with cool new projects added since your last visit. It's time to jump back in and continue your programming journey with us.</p>
      <p>
        Let's get coding!
      </p>
    </div>
      <!-- Call to Action Button -->
     <a href="https://gigo.dev/" class="button" style="color: white;">Resume Learning</a>
    </div>
      <div class="footer">
        <!-- Footer content with unsubscribe link -->
    <p>If you'd rather not receive notifications, you can <a href="https://gigo.dev/unsubscribe" class="unsubscribe">unsubscribe here</a>.</p>
      </div>
    </div>
    
    </body>
    </html>

### Finally, Setting the Active Template and Sending the Email

To accomplish these final steps, I wrote a function that would handle the process of setting the template and dispatching the email. The function SendMonthInactiveMessage was created to reach out to users who had been inactive for a month, encouraging them to re-engage with GIGO. This is a demonstration of how the function was structured and implemented:

    // SendMonthInactiveMessage sends a message to a user that has not been active for one month
    func SendMonthInactiveMessage(ctx context.Context, mailGunKey string, mailGunDomain string, recipient string) error {
        // create new Mailgun client
        mg := mailgun.NewMailgun(mailGunDomain, mailGunKey)
    
        // validate email addresses
        _, err := mail.ParseAddress(recipient)
        if err != nil {
            return fmt.Errorf("invalid recipient email: %v", err)
        }
    
        // configure email content
        message := mg.NewMessage("", "", "", recipient)
    
        // set the preconfigured email template
        message.SetTemplate("monthinactivehtml")
    
        // set the email template version
        message.SetTemplateVersion("monthinactive")
    
        // send the message
        _, _, err = mg.Send(ctx, message)
        if err != nil {
            return fmt.Errorf("failed to send welcome email: %v", err)
        }
    
        return nil
    }

This function creates a new Mailgun client, validates the recipient’s email address, sets up the email content with the desired template, and sends the message.

### The End Result

Following the successful integration of Mailgun’s API into GIGO and the meticulous crafting of our email templates, here is a visual representation of the final product — the email sent to our users.

![](https://cdn-images-1.medium.com/max/2000/1*PRcec7DR2z74qGdRsi711A.png)

## Conclusion

The integration of Mailgun into GIGO marks a significant milestone in our journey to enhance user experience and engagement. As detailed in this article, the process was not just about incorporating an email service but also about ensuring that every aspect of email communication was thoughtfully executed. From selecting the right email service provider to creating aesthetically pleasing and functional email templates, each step was taken with our users’ needs in mind.

This endeavor has not only streamlined our email communication system but has also strengthened our commitment to delivering a seamless and engaging platform for our developer community. We’ve ensured that each email sent out is more than just a message — it’s a reflection of GIGO’s dedication to its users.

For those interested in diving deeper into our journey or exploring our code, we invite you to visit [GIGO’s website](https://gigo.dev/) and our [open-source project on GitHub](https://github.com/Gage-Technologies/gigo.dev). Here, you can see firsthand the work and passion that goes into making GIGO a continually evolving and user-centric platform.

As GIGO grows, we remain committed to adopting and integrating technologies that enhance our platform’s capabilities, always with the aim of enriching the developer experience. Stay tuned for more updates and insights as we forge ahead on this exciting journey.

Check out the GIGO twitter for platform updates: h[ttps://twitter.com/gigo_dev](https://twitter.com/gigo_dev)
Stop by the GIGO discord to connect with the team and community: ht[tps://discord.gg/xg5mkG8CvD](https://discord.gg/xg5mkG8CvD)
Shameless GIGO Reddit plug: [https://www.reddit.com/r/gigodev/](https://www.reddit.com/r/gigodev/)

Get started on the GIGO platform at [gigo.dev](http://gigo.dev/)
