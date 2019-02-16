![Mailer image](/blogs/2019/preview-email/assets/mailer.gif)

## Why does it matter to not send an email during development on local?

- At times, while doing development developer tends to send an email intentionally or unintentionally to verify the email functionality for a specific feature which intern could lead to getting bombarded with testing emails which is of no use.

- Then it could be possible that the developer needed to reply on the same email saying,

<center><i>“Please ignore the last mail as it was by mistakenly sent during testing on local”</i></center>

The situation still could be handled with peace when the victim is the colleague of yours

- What if ‘to’ recipient contains some team mailer group….

- What if ‘to’ recipient contains the CEO’s mail id….

![Situation image](/blogs/2019/preview-email/assets/situation.gif "A situation of you shooting a mail and realizing it was sent to CEO")

## How do we avoid such scenario?

- Well, my approach to this problem started with the exploration of various tools for setting up the local mailer environment of sorts which would be 💯 free of cost.

- Came across quite a few open source tools like [mailcatcher](https://mailcatcher.me/), [papercut](https://github.com/changemakerstudios/papercut), [FakeSMTP](http://nilhcem.com/FakeSMTP/) to configure local SMTP Mail exchange server.

- After a while, revisited the problem and felt that I overlooked the problem. The problem is not to have our very own SMTP server.

- Intentionally or unintentionally I would only need to view the final rendered email to verify mostly `from`, `to`, `cc`, `subject`, `body` and not the sending functionality
  Then I put the pedal to the metal and explored 🔍 [email-templates](https://www.npmjs.com/package/email-templates) and soon realized that we could easily cater to our needs without having to use the external tool at all.

## Implementation

<script src="https://gist.github.com/ashwinsoni/396d9eab6588e09bbc4b4dc6c245d8a7.js"></script>

<script src="https://gist.github.com/ashwinsoni/6c1bb23e6c0ebe33b90ae9954be48bc8.js"></script>

<script src="https://gist.github.com/ashwinsoni/f5ab8946383246a02dc0d50eacddd029.js"></script>

The directory structure for reference

```
.
├── Mailer.js
├── index.js
├── package-lock.json
├── package.json
└── templates
    └── body.pug
```

### 📁Mailer.js

```
const email = new Email({
  views: { root: “./templates” },
  preview: isLocal
});
```

- has the configurationpreview option(at line:13) available while instantiating the library which would enable or disable the email preview on a default browser
  views option(at line:12) indicates the directory path( ./templates) where all your templates reside, using pug/jade([default and recommended by the email-templates](https://www.npmjs.com/package/email-templates#install)) for this example

```
await email.send({
  message: {
    from,
    to,
    subject,
    html: await email.render('/test', data)
  }
});
```

- Once the configuration is done, then use its send function(at line:18) specifying from to subject html
  html property would render the view. Filename(./templates/test.pug will be picked up in this case) and the data will be injected in the template

### 📁index.js

```
const message = {
  from: "sender@domain.com",
  to: "receiver@domain.com",
  subject: "Local mail testing",
  data: { name: "world" }
};
Mailer.send(message)
```

- Then you simply call the send function of Mailer.js as shown in index.js (at line:10) which will inject the name variable into the test.pug and render the email in a new tab

- Local mail snapshot for reference:

![snapshot image](/blogs/2019/preview-email/assets/snapshot.png)

## Did you know that you can preview the rendered mail before actually sending them before this? 🤔

I didn’t.

<center><i>“So if you’re asking me what to do with all this knowledge you’re accumulating, I say… Pass it on…”
 ~ Professor Norman in Lucy (2014)</i></center>
