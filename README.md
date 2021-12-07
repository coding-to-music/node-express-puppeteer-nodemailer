# Scraper Template With node express puppeteer nodemailer

This is a scrapper template repo; Stack: NodeJS, Express, Puppeteer and Nodemailer

![Dependencies](https://img.shields.io/badge/dependencies-up%20to%20date-brightgreen.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

## Basic Overview

This template is open to be used, just click on `Use this template` and create your own repo with a base.

For development purposes execute:

```java
    npm run dev
```

This command above will create an instance attached to [nodemon](https://nodemon.io/). f

For deploying purposes, instead it is required to execute:
```java
    npm start
```

https://dev.to/crisarji/how-i-met-your-scraper-4n6o


## How I met your...Scraper?
With node express puppeteer nodemailer


Hello developer pal!, glad to see you here.

In this post, I'll share my experience after running into a topic I had not met before... web scraping!.

## Show Me The Topics

The topics to be focused on are:

## Problem to solve: Booking a weekly service
- Project dependencies
- NodeJS folders structure
- Express, Routing and Services
- Services Visualization
- Puppeteer(Booking Service)
- Nodemailer(Email Service)
- Local use and remote Deploy
- Bonus: Dealing with Captcha
- Conclusion
- `Disclaimer`: This post comes from a particular scenario I've been struggling with, I'm not preaching this is the best approach to follow for web scraping, nor the worst, any contribution is more than welcome in the threads below!

- `Note`: There is available also a template project on GitHub in case it could be useful and save you some time.

## Problem to solve: Booking a weekly service
A couple weeks ago, I got subscribed to a weekly delivery service, I am pretty happy with the service!, it is fast, efficient, always on time!, since day 1 the service has had no issues, not even delays, what's the only fallback I have found so far?, the booking process!

This could be a little picky from my end, I know, but see the steps I need to do every-single-day:

- Open a website(only works on Chrome, no other browser)
- Fill in my user/password
- Move to Members path
- Check my information and select the delivery address(displayed in a dropdown)
- Move to the Next Step
- Select the day of the week I want to book me the service(come on!, it is a week from today, as usual)
- Move to the Next Step
- Select the time of the day I want to book me the service on(it is the same time as every single day, dammit)
- Finish the process
- A "Thank you page is displayed"(without the result of the process I just did)
- Move to `Members` path(again) and look for my upcoming bookings table result

These steps needs to be done every-single-day, and it is a pain in the back, cause if for some reason I forget to do it, my preferred time could have been taken, and I need to look for a different time, then I need to be aware of the delivery time(it would be different a week from today than the rest of the days); am I clear why this is a pain?, I hope so...

After a few days of missing the booking, I decided to automate the process with the help of some tools, I was not sure about how to start, so I research and gladly met web scrapping(don't get me wrong, I had heard about it, but there is a slightly difference between hearing and researching with a purpose, at least form my end 🤷).

So, what's the web scraping?, there are plenty of definitions out there on the Internet, the one that is more accurate for this post purposes is:

Web scraping is the process of using bots to extract content and data from a website.

Source: What is web scraping?
This is exactly what this post is about, create a sort of robot that will fill in information on my behalf in a site and later it will extract a result for me, and put it on my inbox.

## Project dependencies
The tools used for accomplishing this enterprise are:

Main Dependencies

- NodeJS
- Express
- Puppeteer
- Nodemailer

Dev Dependencies

- Nodemon

package.json
```java
"dependencies": {
    "express": "^4.17.1",
    "nodemailer": "^6.6.2",
    "puppeteer": "^10.1.0"
  },
  "devDependencies": {
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^3.4.0",
    "nodemon": "^2.0.9",
    "prettier": "^2.3.2"
  }
```

Prettier and Nodemon come handy for having a nice experience, not mandatory though, fell free to use any other tool.

## NodeJS folders structure
For this project, the structure is simple and set as follows:
```java
scraper-template/
    ├── index.js
    ├── package.json
    └── routes/
      ├── booking.js
    └── screenshots/
      ├── home-page.png
    └── services/
      ├── bookingHandler.js
      ├── emailSender.js
```

There is one route for express to serve, two services for booking and emailing the results and a folder for screenshots, which only steps in development environment.

## Express, Routing and Services
The index.js is a simple file with a 20 lines extension:

```java
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;
const booking = require('./routes/booking');

app.get('/', (req, res) => {
  res.json({ message: 'ok' });
});

app.use('/booking', booking);

/* Error handler middleware */
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  console.error(err.message, err.stack);
  res.status(statusCode).json({ message: err.message });
  return;
});

app.listen(port, '0.0.0.0', () => {
  console.log(`Scrapper app listening at http://localhost:${port}`);
});
```

The routes/booking.js includes the expressjs, services and config references, let's decompose it!:

express.js

The references to the packages used:
```java
const express = require('express');
const router = express.Router();
...
...
```

services.js

The references to the defined services for handling the bookings and sending emails, a preview can be found below on Services Visualization
```java
...
...
const emailSender = require('../services/emailSender');
const bookingHandler = require('../services/bookingHandler');
...
...
```

config.js

All the vales in here are process.env vars, these includes keys for login(webSiteUser, webSitePassword), email impersonation(authUser, appPassword) and email receivers(emailFrom, emailTo):
```java
...
...
const {
  webSiteUser,
  webSitePassword,
  authUser,
  appPassword,
  emailFrom,
  emailTo,
  preferTime,
} = require('../config');
```java
book-me endpoint

This route does the booking process for a user with a preferred time(if any):
```java
router.get('/book-me', async function (req, res, next) {
  try {
    const bookMeResult = await bookingHandler.bookMe(
      webSiteUser,
      webSitePassword,
      preferTime
    );
    res.send(`The result of the booking was::${bookMeResult}`);
  } catch (err) {
    console.error(`Error while booking me for next week`, err.message);
    next(err);
  }
});
...
...
```
book-me endpoint

This route gets the bookings the user has set for the upcoming week:
```java
...
...
router.get('/my-bookings', async function (req, res, next) {
  try {
    const bookingResult = await bookingHandler.myBookings(
      webSiteUser,
      webSitePassword
    );
    emailSender.sendEmail(bookingResult, {
      authUser,
      appPassword,
      emailFrom,
      emailTo,
    });
    res.format({
      html: () => res.send(bookingResult),
    });
  } catch (err) {
    console.error(`Error while getting the booking for this week`, err.message);
    next(err);
  }
});
```
## Services Visualization

### Service emailSender:

![https://github.com/coding-to-music/node-express-puppeteer-nodemailer/blob/main/images/email-service.png?raw=true]


### Service bookingHandler:

![https://github.com/coding-to-music/node-express-puppeteer-nodemailer/blob/main/images/booking-service.png?raw=true]

## Puppeteer(Booking Service)
Here is where the magic begins!, only one reference for rule the whole process:

```java
const puppeteer = require('puppeteer');
```

After this import, puppeteer is ready to roll!; there are plenty of examples on the internet, most of them apply all the concepts for web scraping in one single file, this is not the case.

This project applies some separations that, from my perspective, makes it easier to understand what's going on every step throughout the whole process, so let's dive into the sections:

### -- Start the Browser --

The first interaction is starting the Browser. Puppeteer works perfectly with Chronium and Nightly, for this project the reference used is the default one, with Chrome(the web site to scrap only opens on Chrome), but if Firefox preferred, take a look at this thread on StackOverflow.

In the piece of code below, there is a var initialized for isProduction, this var is ready for being used when deployed on a web platform(Heroku we'll talk about it later), and another for isDev, I repeat, this is for explanation purposes, it is not required to have 2 when one of them can be denied and cause the same result.

When `isProduction` the launch is done headless by default, it means that the process is done in the background without any UI, also some args are included for a better performance, refer to the list of Chromium flags here.

When `isDev`, the headless is false, and args also include one for opening te dev tools after loading the browser.

```java
const isProduction = process.env.NODE_ENV === 'production' ? true : false;
const isDev = !isProduction;
const authenticationError = 'Failed the authentication process';
const bookingError = 'Failed the booking process';

async function startBrowser() {
  let browser = null;
  let context = null;
  let page = null;
  if (isProduction) {
    browser = await puppeteer.launch({
      args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-dev-shm-usage'],
    });
    page = await browser.newPage();
  } else {
    browser = await puppeteer.launch({
      headless: false,
      defaultViewport: null,
      slowMo: 75,
      args: [
        '--auto-open-devtools-for-tabs',
        '--disable-web-security',
        '--disable-features=IsolateOrigins,site-per-process',
        '--flag-switches-begin --disable-site-isolation-trials --flag-switches-end',
      ],
    });
    context = await browser.createIncognitoBrowserContext();
    page = await context.newPage();
  }
  return { browser, page };
}
```
As seen above, the site is loaaded in Incognito, but can be opened in a regular tab.

### -- Do Login --

For doing the login, some puppeteer features come in play:

- `goto`: allows the navigation to a web site
- `type`: types a value in an input field
- `click`: allows clicking on buttons, table cells, submits
- `waitForSelector`: recommended for allowing the page to recognize a particular selector before moving along
`screenshot`: takes a screenshot on demand, and store it in the app(it is possible to redirect the screenshots to remote services, in dev just place them in a root folder)

```java
async function doLogIn(page, webSiteUser, webSitePassword) {
  await page.goto(constants.baseUrl + constants.loginEndpoint, {
    timeout: constants.timeOut,
    waitUntil: 'load',
  });
  isDev && console.log('Navigation to Landing Page Succeeded!!!');

  await page.type('#loginform-email', webSiteUser);
  await page.type('#loginform-password', webSitePassword);
  await page.click('button[type="submit"]');
  isDev && console.log('Login submitted');

  await page.waitForSelector('#sidebar');
  isDev && (await page.screenshot({ path: 'screenshots/home-page.png' }));

  return await findLink(page, constants.scheduleEndpoint);
}
```

Something to remark in the code above is that when dealing on development environment, the screenshots are taken, in production those are skipped(on purpose for the sake of the example)

## -- Find a link --

This can change from page to page, but for this project, there is a link that was tracked down to the point that only loggedin members are able to see, for finding this or any other, a function is available, which receives as parameters the page instance and the endpoint to look for as an href:

```java
async function findLink(page, endpoint) {
  const pageLinks = await page.evaluate(() =>
    Array.from(document.querySelectorAll('a[href]'), a => a.getAttribute('href')),
  );
  return pageLinks.includes(endpoint) || null;
}
```

### -- Close the Browser --

Just pass the browser instance as parameter and close it.
```java
async function closeBrowser(browser) {
  return browser.close();
}
```
Note: not going to elaborate on the details of the booking process, just take into account:

- It is a wizard
- The wizard has 3 steps, the final one is a submit
- The name of the elements in the query selectors are tied to the site I'm scraping on, feel free to change them as much as you need
- The idea is to share how to find elements, how to use query selectors, how to get the outerHtml on elements, wait for them to be available, all of this using Puppeteer

## Nodemailer(Email Service)
Email service is contained in 30 lines of code, it is a define structure required by the import of nodemailer

`Note`: When using Gmail, it is mandatory to enable less secure apps, this will create a new password for just the particular application you are trying to link to, can read more here in nodemailer or in Google Support

```java
const nodemailer = require('nodemailer');

async function sendEmail(weekBookings, { authUser, appPassword, emailFrom, emailTo }) {
  const mail = nodemailer.createTransport({
    service: 'gmail',
    auth: {
      user: authUser,
      pass: appPassword,
    },
  });

  const mailOptions = {
    from: emailFrom,
    to: emailTo,
    subject: 'Your bookings for this week',
    html: weekBookings,
  };

  mail.sendMail(mailOptions, (error, info) => {
    if (error) {
      console.log(error);
    } else {
      console.log('Email sent: ' + info.response);
    }
  });
}

module.exports = {
  sendEmail,
};
```

There is not too much complication in here, pass the authUser, appPassword, email from/to and the html to be send as email.

## Local use and remote Deploy
How to be sure that everything is working as expected?, well two options:

## -- Locally --

For running this locally Postman is the tool(don't judge me too much, I am used to it... used to Postman I meant, anyway)
```java
  WEB_SITE_USER=YOUR_USER@YOUR_EMAIL_DOMAIN.com WEB_SITE_PASSWORD=YOUR_PASSWORD
  GMAIL_AUTH_USER=YOUR_USER@gmail.com GMAIL_APP_PASSWORD=YOUR_APP_PASSWORD
  GMAIL_EMAIL_FROM=YOUR_USER@gmail.com GMAIL_EMAIL_TO=YOUR_USER@gmail.com
  BOOKING_PREFER_TIME=06:55:00 npm run dev
```

This command will start the local server using nodemon setting all the expected process.env variables in port 3000 by default, so just use Postman for hitting http://localhost:3000/booking/book-me or http://localhost:3000/booking/my-bookings and a result will be retrieved. 

## -- Remote --

For deploying remotely the platform used id Heroku, not getting in details but found this helpful post in case you decide to follow that path(read carefully the Heroku's sections, and highly suggested to use Kaffeine).
All the process.env passed to the terminal when running locally are set as Heroku's environment variables, then the deploy is transparent.

## Bonus: Dealing with Captcha
Sometimes the sites you try to scrap are sort of "protected" by Captcha, I say "sort of" cause there are ways to skip it, even some companies pay to regular users to help them to recognize captchas, you can read more over here.

The page scraped for this post behaves "interesting", sometimes the reCaptcha is ignored, some others appear right after submitting the login, so randomly fails; I opened an issue in puppeteer-extra, an npm lib extension for puppeteer which works hand-to-hand with 2captcha, I'm watching the issue closely, in case of getting a fix for the random issue I'll edit the post.

In case you were wondering, the hit of the endpoints after deployed to Heroku are done by a Cron-Job, it is fast and easy, and I received a custom email when the process randomly fails(the idea is to make it work permanently!).

## Conclusion
As shown above, the web scraping is a great technique for making life easier, some hiccups could appear along the way(Captcha, Deploy servers restrictions or conditions) though some how it is possible to make it through!; maybe you could have a better way to do it, let's discuss in a thread below!

Thanks for reading!