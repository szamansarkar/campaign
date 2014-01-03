![campaign.png][9]

> Compose responsive email templates easily, fill them with models, and send them out.

This is the stuff responsible for sending beautiful emails in [Pony Foo][3]. I've now isolated the code and made it into a reusable package, called `campaign`. It comes with a dead simple API, and a beautiful responsive layout, [originally written by MailChimp][7], adapted by me, and easily configurable.

It uses [Mustache][6] to fill out the email templates, and [Mandrill][1] <sub>_(by default)_</sub> to send the emails, although providing your own [`provider`](#providers) to actually send the emails is easy.

# Reference

Quick links for reference.

- [Changelog][changelog]
- [Getting Started](#getting-started)
- [Client Options](#client-options)
- [Send Options](#email-sending-options)
- [Templates](#templates)
- [Styling](#styling-the-layout)
- [Debugging](#debugging)
- [Providers](#providers)
- [License](#license)

# Getting Started

Install using `npm`.

```shell
npm i --save campaign
```

Set it up.

Construct a `client`.

```js
var client = require('campaign')();
```

<sub>_(the default provider needs an API key for [Mandrill][1], read on)_</sub>

Send emails!

```js
client.send(template, options, done);
client.sendString('<p>{{something}}</p>', options, done);
```

<sub>_(detailed information below)_</sub>

# Screenshot

Here is a screenshot of an email sent using this library, as seen on [Pony Foo][3] subscriptions, in production. This email is using the default layout provided by `campaign`.

![sample.png][8]

# Client Options

Here are the default options, they are explained below.

```json
{
    "mandrill": {
        "apiKey": "<not provided>",
        "debug": false
    },
    "from": "<not provided>",
    "provider": "<default>",
    "trap": false,
    "headerImage": "<not provided>",
    "layout": "<defaults>"
}
```

### `trap`

If `true`, then emails won't be sent to any of the recipients, but they'll be sent to the provided `trap` address instead. For example, you could set `trap` to `nico@bevacqua.io`, and all emails would be sent to me instead of the intended recipients. Great for spamming me, and also great for testing.

When you `trap` recipients, the email will get a nifty JSON at the end detailing the actual recipients that would've gotten it.

### `mandrill`

By default, the [Mandrill][1] service is used to send the emails. Mandrill is really awesome and you should be using it. It has a generous free plan.

At the time they host [their API's source code][2] in Bit Bucket, which is kind of weird, but you can read through it nonetheless.

You need to provide an API key in `apiKey`, and that's all there is to it. You might prefer to _ignore this configuration option_, and merely set `process.env.MANDRILL_APIKEY`. That works, too.

### `from`

The `from` address for our emails. The `provider` is responsible for trying to make it look like that's the send address. Not necessarily used for authentication.

### `provider`

You can use other email providers, [creating your own or choosing one](#providers) that comes with `campaign`. To implement it yourself, you need to create a custom `provider` object. The `provider` object should have a `send` function, which takes a `model`, and a `done` callback. You can [read more about custom providers](#creating-custom-providers) below.

### `headerImage`

You may provide the full path to an image. This image will be encoded in `base64` and embedded into the email as a heading. Embedding helps people view your emails offline.

This image should have a `3:1`_ish_ ratio. For instance, I use `600x180` in [my blog][3]'s emails.

### `layout`

The layout used in your emails. Templates for email sending are meant to have the bare minimum needed to fill out an email. Since you want a consistent UX, the same `layout` should be used for every email your product sends.

A default layout `template` is provided. You can provide a different one, just set `layout` to the absolute path of a [Mustache][6] template file. For information about the model passed to the layout, see the **Templates** section.

# Email Sending Options

Once you've created a client, you can start sending emails. Here are the default options, and what you need to fill out.

```json
{
    "subject": "<not provided>",
    "preview": "<options.subject>",
    "to": "<not provided>",
    "images": "<empty>",
    "social": {
        "twitter": "<not provided>",
        "landing": "<not provided>",
        "name": "<not provided>"
    },
    "mandrill": {
        "tags": "<not provided>",
        "merge": "<not provided>"
    },
    "styles": "<defaults>"
}
```

#### `.send` vs `.sendString`

The only difference between `.send` and `.sendString` is that `.send` takes the path to a file, rather than the template itself. `.send` compiles the template and keeps it in a cache, while `.sendString` compiles the template every time.

### `subject`

The email subject.

### `preview`

This is the line that most email clients show as the _preview_ of the email message. It defaults to the subject line. Changing it is extremely encouraged.

### `to`

These are the recipients of the email you're sending. Simply provide a single recipient's email address, or an array of email addresses.

### `images`

If you want to provide the template with embedded images _(other than the [optional email header](#headerimage) when creating the `campaign` client)_ you can set `images` to a list of file paths and names (to later reference them in your templates), as shown below.

```js
[
    { name: 'housing', file: path.join(__dirname, 'housing.png') }
]
```

### `social`

Social metadata used when sending an email can help build your brand. You can provide a `twitter` handle, a `name` for your brand, and a `landing` page.

The `name` is used as the name of the send address, as well as in the "Visit <name>" link.

### `mandrill`

Configuration specifically used by the Mandrill provider.

Mandrill allows you to add dynamic content to your templates, and this feature is supported by the default Mandrill provider in `campaign`, out the box. Read more about [merge variables][4].

##### `mandrill.merge`

Given that Mandrill's `merge` API is **fairly obscure**, we process it in our provider, so that you can configure it assigning something like what's below to `mandrill.merge`, which is cleaner than what Mandrill expects you to put together.

```json
{
    "locals": [{
        "email": "someone@accounting.is",
        "model": {
            "something": "is a merge local for the guy with that email"
        }
    }],
    "globals": [{
        "these": "are merge globals for everyone"
    }]
}
```

[Mandrill][1] lets you tag your emails so that you can find different campaigns later on. Read more about [tagging][5]. By default, emails will be tagged with the template name.

### `styles`

[Read about styles](#styling-the-layout) below.

# Templates

There are two types of templates: the `layout`, and the email's `body` template. A default `layout` is provided, so let's talk about the email templates first, and then the layout.

### Email `body` Templates

The `body` template determines what goes in the message body. The [options](#email-sending-options) we used to configure our email are _also used as the model_ for the `body` template, as sometimes it might be useful to include some of that metadata in the model itself.

The API expects an absolute path to the `body` template.

```js
client.send(body, options, done);
```

Other than the [options listed above](#email-sending-options), you can provide _any values you want_, and then reference those in the template.

### The `layout` Template

The `layout` has one fundamental requirement in order to be mildly functional, it should have a `{{{body}}}` in it, so that the actual email's content can be rendered. Luckily the default `layout` is good enough that **you shouldn't need to touch it**.

Purposely, the layout template isn't passed the full model, but only a subset, containing:

```json
{
    "_header": "<!!options._header>",
    "subject": "<options.subject>",
    "preview": "<options.preview>",
    "generated": "<when>",
    "body": "<html>",
    "trapped": "<options.trapped>",
    "social": "<options.social>",
    "styles": "<options.style>"
}
```

In this case, the `_header` variable would contain whether a header image was provided. Then, `generated` contains the moment the email was rendered, passing the `'YYYY/MM/DD HH:mm, UTC Z'` format string to [`moment`][11]. Lastly, `trapped` contains the metadata extracted from the model when `trap` is set to `true`, in the [client options](#client-options).

### Styling the `layout`

These are the default `styles`, and you can override them in the `options` passed to [`client.send`](#email-sending-options).

```json
{
    "styles": {
        "bodyBackgroundColor": "#eaeadf",
        "bodyTextColor": "#505050",
        "codeFontFamily": "Consolas, Menlo, Monaco, 'Lucida Console', 'Liberation Mono', 'DejaVu Sans Mono', 'Bitstream Vera Sans Mono', 'Courier New', monospace, serif",
        "fontFamily": "Helvetica",
        "footerBackgroundColor": "#f4f4f4",
        "headerColor": "#412917",
        "horizontalBorderColor": "#dedede",
        "layoutBackgroundColor": "#f3f4eb",
        "layoutTextColor": "#808080",
        "linkColor": "#e92c6c",
        "quoteBorderColor": "#cbc5c0"
    }
}
```

### Unsubscribe Facilities

The default `layout` supports an optional `unsubscribe_html` merge variable, which can be filled out like below.

```json
{
    "merge": {
        "locals": [{
            "email": "someone@somewhere.com",
            "model": {
                "unsubscribe_html": "<a href='http://sth.ng/unsubscribe/hash_someone'>unsubscribe</a>"
            }
        }, {
            "email": "someone@else.com",
            "model": {
                "unsubscribe_html": "<a href='http://sth.ng/unsubscribe/hash_someone_else'>unsubscribe</a>"
            }
        }]
    }
}
```

That'd be a perfect use for merge variables, which were described above in the [send options](#email-sending-options). Remember, those are just supported by Mandrill, though. Mandrill [deals with merge variables][4] after you make a request to their API, replacing them with the values assigned to each recipient.

# Debugging

To help you debug, an alternative `provider` is available. Set it up like this:

```js
var campaign = require('campaign');
var client = campaign({
    provider: campaign.providers.console()
});

// build and send mails as usual
```

Rather than actually sending emails, you will get a lot of JSON output in your terminal. Useful!

# Providers

There are a few different providers you can use. The default provider sends mails through [Mandrill][1]. There is also a `console` logging provider, [explained above](#debugging), and a `nodemailer` provider, detailed below.

### Using `nodemailer`

To use with `nodemailer`, simply use that provider.

```js
var nodemailer = require('nodemailer');
var smtp = nodemailer.createTransport('SMTP', {
    service: 'Gmail',
    auth: {
        user: 'gmail.user@gmail.com',
        pass: 'userpass'
    }
});

var campaign = require('campaign');
var client = campaign({
    provider: campaign.providers.nodemailer({
        transport: smtp,
        transform: function (options) {
            // add whatever options you want,
            // or return a completely different object
        }
    })
});

// build and send mails as usual
```

That's that.

### Creating custom providers

If the existing providers don't satisfy your needs, you may provide your own. The `provider` option just needs to be an object with a `send` method. For an example, check out the [`nodemailer` provider source code][10].

You can easily write your own `campaign` provider, like this.

```js
var campaign = require('campaign');
var client = campaign({
    provider: {
        send: function (model, done) {
            // use the data in the model to send your email messages
        }
    }
});

// build and send mails as usual
```

If you decide to go for your own provider, `campaign` will still prove useful thanks to its templating features.

# License

MIT

  [changelog]: CHANGELOG.md
  [1]: http://mandrill.com/
  [2]: https://bitbucket.org/mailchimp/mandrill-api-node/src/d6dcc306135c6100d9bc2e2da2e82c8dec3ff6fb/mandrill.js?at=master
  [3]: http://blog.ponyfoo.com
  [4]: http://help.mandrill.com/entries/21678522-How-do-I-use-merge-tags-to-add-dynamic-content-
  [5]: http://help.mandrill.com/entries/28563573-How-do-I-use-tags-in-Mandrill-
  [6]: https://github.com/janl/mustache.js
  [7]: https://github.com/mailchimp/Email-Blueprints
  [8]: http://i.imgur.com/Coy4m0Y.png
  [9]: http://i.imgur.com/cBFalWm.png
  [10]: https://github.com/bevacqua/campaign/blob/master/src/providers/nodemailer.js
  [11]: http://momentjs.com
