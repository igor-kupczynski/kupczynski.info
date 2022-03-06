---
title: Notes on creating a telegram bot
tags:
- golang
- cloud
aliases:
- /2020/11/28/notes-on-personal-telegram-bot.html
---
I've created a telegram bot to give me a daily updates to a webpage I was tracking. This is a hobby project and the hosting price is an important concern.

- [The problem](#the-problem)
  - [Context](#context)
- [The solution](#the-solution)
  - [Why not open-source the code?](#why-not-open-source-the-code)
- [Notes on the project](#notes-on-the-project)
  - [Telegram bot API is really good for experimentation](#telegram-bot-api-is-really-good-for-experimentation)
    - [Anyone can chat to your bot](#anyone-can-chat-to-your-bot)
  - [Google App Engine](#google-app-engine)
    - [Secrets are hard to manage](#secrets-are-hard-to-manage)
    - [If you don't know GAE start with the _concepts_](#if-you-dont-know-gae-start-with-the-concepts)
    - [Run it for cheap :)](#run-it-for-cheap-)
    - [Old versions of the app may be still running](#old-versions-of-the-app-may-be-still-running)
    - [The app is not guaranteed to scale up from 0 to 1 after deployment](#the-app-is-not-guaranteed-to-scale-up-from-0-to-1-after-deployment)
    - [Don't forget to clean up the lingering images!](#dont-forget-to-clean-up-the-lingering-images)
    - [The built-in observability tools are really good (at least for a personal project)](#the-built-in-observability-tools-are-really-good-at-least-for-a-personal-project)
  - [Structuring the app](#structuring-the-app)
    - [Go concurrency patterns are not that useful](#go-concurrency-patterns-are-not-that-useful)
    - [Protecting the internal endpoints](#protecting-the-internal-endpoints)
    - [Use firestore to store the state of the app](#use-firestore-to-store-the-state-of-the-app)
    - [Evaluate FaaS as an alternative](#evaluate-faas-as-an-alternative)
- [Summary](#summary)

## The problem

I want to be notified about new real estate ads from my neighborhood.

### Context

- There's a popular real estate ads portal I currently check every now and then to get the "feel" of the market I'm interested in. I want to _outsource_ the checking to a bot and be notified about new ads.
- This is for curiosity and to track price trends. I don't need to know about every new ad. I'm interested only in ads matching certain criteria. I don't need to be notified right away — once a day, maybe even once a week is OK.
- This is a hobby project, hosting price, educational value, and developer happiness are important and legitimate concerns for the solution 😀

## The solution

1. **Write a telegram bot to scrape the ads portal and send notifications about new ads matching the given criteria.**

   - I use telegram for some group chats and it's already present on my phone.
   - It has a nice [bot API][botapi].
   - A user/bot can mark messages they send as _silent_ so they don't distract the recipient.

2. **Write the bot with go.**

   - I'm involved in some go projects at work, and I feel quite productive with the language.
   - Small backend services are the sweet spot for go.
   - I've contributed to some go projects, but starting from scratch is a different game. I've wanted to practice that.
     - I've ignored [go-telegram-bot-api/telegram-bot-api](https://github.com/go-telegram-bot-api/telegram-bot-api). Writing the API model from scratch is a bit of work, but I wanted to do that for educational purposes (see [_Don't Reinvent The Wheel, Unless You Plan on Learning More About Wheels_](https://blog.codinghorror.com/dont-reinvent-the-wheel-unless-you-plan-on-learning-more-about-wheels/)).

3. **Host the bot on Google App Engine (GAE) in the Google Cloud (GCP).**
   
   - I don't want to run the bot on my laptop, and I don't want to maintain the server at home. We need to go to the CLOUD! 🌥
     - Of course, we can also host a VM on digital ocean (or somewhere similar) for $5 / month ;).
       - Sidenote: If you want to try digital ocean [my ref link](https://m.do.co/c/649b18bb5e59) will give you $100 credits to start (over 60 days; and $25 for me).
     - I wanted to use a managed solution though, mostly to avoid maintenance work on the project as much as possible.
   - Why Google App Engine and not Heroku, or Azure/AWS equivalents?
     - Mostly to get more hands-on experience with GCP.
     - They have a [generous free tier][gcpfreetier] and I expect my use case to fit into the limits and cost me $0 / month (or close to it).


### Why not open-source the code?

Because of the website I'm scraping. The laws on the subject are complex, scraping for personal purposes is different than showing the public how to do it. I don't want to go through the hassle of figuring it out in the major jurisdictions. I could take out the scraping part and open-source the rest, but for now, I'm not willing to put more personal time into the project.

I hope you'll find the notes useful regardless!


## Notes on the project

### Telegram bot API is really good for experimentation

The docs are really good and easy to work with. There's no tutorial, so I recommend to use VS code + [REST client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) to experiment. This is useful later on when you try to debug or understand the API a bit better. The REST client gives us a nice, interactive, almost REPL like dev-experience.

1. Talk to _BothFather_ on telegram to create a new bot: [[docs]](https://core.telegram.org/bots#3-how-do-i-create-a-bot).


2. Start with the following requests:

{% raw %}
```
@token = token-from-botfather
@root = https://api.telegram.org/bot{{token}}


#### Info
GET {{root}}/getMe


#### Get new messages sent to the bot
GET {{root}}/getUpdates


#### Respond back to a chat
@chat_id = numeric-chat-id
POST {{root}}/sendMessage?chat_id={{chat_id}}&text=Hello


#### Check webhook status
GET {{root}}/getWebhookInfo
```
{% endraw %}

![VS code REST client session example](/archive/2020-11-telegram-bot-api-experiments.png)


#### Anyone can chat to your bot

You may want some form of authentication before allowing telegram users to use all the bot features. In my case, I ask for a user token in order for the bot to notify you about new ads.

### Google App Engine

I've really enjoyed working with the platform. There's a learning curve, but there's not a lot of complexity if you think about it as a docker container runner :)

#### Secrets are hard to manage

As a responsible [twelve-factor app](https://12factor.net/config) citizen you want to provide the config, including secrets, via environment variables. On GAE you put them in `app.yaml` file. The problem is that you can't keep the file plaintext in your repo — everyone with access to the code will be able to access the secrets as well.

See, e.g.  [https://cloud.google.com/appengine/docs/standard/go/sending-messages](https://cloud.google.com/appengine/docs/standard/go/sending-messages)

> Add the key to the environment variables section in your app's app.yaml

> Note that anyone with access to your app.yaml will also have access to your API keys. If you can't store app.yaml in a secure location, we recommend you store the API keys in a secure location such as Datastore and retrieve the keys at runtime, or keep the keys in your code but encrypt them with a keystore, such as Cloud Key Management Service.

This is a far cry from the ease of adding "secret" env variables in other services. E.g. adding secrets to GitHub Actions is just a few clicks in the UI.

The alternative is to use something like [gitcrypt](https://www.agwa.name/projects/git-crypt/) to transparently encrypt the file in your repo.

Remember, we talk about a personal time hobby project, not an _enterprise_ deployment where secret management is likely a solved problem.

#### If you don't know GAE start with the _concepts_

See https://cloud.google.com/appengine/docs/standard/go/concepts, especially the _How Instances are Managed_ section.

#### Run it for cheap :)

**Note** when I comment on pricing in this article it is based on my understanding of the GCP price model. Cloud pricing models are notoriously complex — do your own research, utilize the GCP trial to figure out the costs of your project. Don't blindly follow the advice of some random-person-on-the-internet.

At the time of writing GAE free tier looks like this [[src]][gcpfreetier]:
```
- 28 hours per day of "F" instances
- 9 hours per day of "B" instances
- 1 GB of egress per day
- The Free Tier is available only for the Standard Environment
```

Put something like this in the `app.yaml` config file to stay within the limits:
```
instance_class: F1
automatic_scaling:
  max_instances: 1
```

#### Old versions of the app may be still running

When you do `gcloud app deploy` GCP build and deploys new version of your app. The problem is that the old versions may be still running. Unless you do traffic splitting they probably get 0 traffic, but they may serve existing connections, or they may do some operations in the background. Keep that in mind, delete the old versions when you don't need them (and avoid async operations, GAE is not designed for that, see [_Go concurrency patterns are not that useful_](#go-concurrency-patterns-are-not-that-useful) section).

![Old versions still running](/archive/2020-11-gae-old-versions-still-running.png)


#### The app is not guaranteed to scale up from 0 to 1 after deployment

- Watch out for any init actions — your app may not be started until it gets the first request.
- If there are 0 instances pre-deployment, google may keep it a 0 post-deployment.
- The [docs say that an instance is always running](https://cloud.google.com/appengine/docs/standard/go/how-instances-are-managed#instance_states), but I don't think that's the case:

  > An instance of an auto-scaled service is always running. However, an instance of a manual or basic scaled service can be either running or stopped.

![No instances running](/archive/2020-11-0-instances-runnning.png)


#### Don't forget to clean up the lingering images!

[[docs]](https://cloud.google.com/appengine/docs/standard/go/testing-and-deploying-your-app)

>  Each time you deploy a new version, a container image is created using the Cloud Build service. That container image then runs in the App Engine standard environment. (...) Once deployment is complete, App Engine no longer needs the container images. Note that they are not automatically deleted, so to avoid reaching your storage quota, you can safely delete any images you don't need. 

The images are stored in a Google Cloud Storage bucket. Actually, I've noticed that GCP created two new buckets, one of them had a 15d lifecycle policy attached, but the other not.

#### The built-in observability tools are really good (at least for a personal project)

- Logs: there's a generous 50GB / project free tier for logs as long as you stay within the default retention period (30 days). See [[pricing]](https://cloud.google.com/stackdriver/pricing).

- Traces to profile the app.

  As you can see on the screenshot there's not that much to my app. You can see the histogram, with long-running requests easily distinguishable over the 1000 ms line. It's not that I have 5-sec behemoths in the bot, but rather there were 0 instances, and GCP needed to start one for me to serve the request.

  I'm not sure how useful this is for a small personal "app" but it doesn't hurt to have it, you can easily eyeball if there's anything that takes longer than expected.

  ![Traces](/archive/2020-11-traces.png)


- Debugger

  The debugger is really amazing on paper, but I couldn't get it to work. Based on the docs it is not available for Go in GAE [[src]](https://cloud.google.com/debugger/docs/setup/go).

  ![Debugger no game](/archive/2020-11-debugger.png)

### Structuring the app

#### Go concurrency patterns are not that useful

Due to autoscaling our app may be scaled-down anytime when it is not servicing requests. This means that we can't rely on goroutines (and channels) to complete operations in the background.

A GAE app should be based on the request-response model. It gets a request, does the processing, and responds to the request.

On the other hand, we prefer to respond to customer requests right away and queue the heavy lifting for async processing (see [_Queue everything and delight everyone_](https://decafbad.com/blog/2008/07/04/queue-everything-and-delight-everyone/)).

GCP provides us with primitives to compensate the lack of async processing. We can use [_task queues_](https://cloud.google.com/tasks/docs/dual-overview) and [_cron jobs_](https://cloud.google.com/appengine/docs/standard/go/scheduling-jobs-with-cron-yaml).

- **Task queues** allow us to (unsurprisingly!) queue tasks. A task represents a request to execute against our app. It's almost as if we've externalized the go channels :)

- **Cron jobs** allow us to run a request on a specific schedule.


#### Protecting the internal endpoints

If we use _task queues_ and _cron jobs_ we end up with a few "internal" endpoints. We may not want to let them be called externally.

There's an easy way to only allow internal traffic from tasks or jobs:

- Check if the header `X-AppEngine-TaskName` exists [see [docs](https://cloud.google.com/tasks/docs/creating-appengine-handlers)].
- Check if the header `X-Appengine-Cron: true` exists [see [docs](https://cloud.google.com/appengine/docs/standard/go/scheduling-jobs-with-cron-yaml)].

GAE guarantees that such headers will be deleted from external requests. The list of the headers removed/overwrote by GCP is [[here](https://cloud.google.com/appengine/docs/standard/go/reference/request-response-headers#removed_headers)].


#### Use firestore to store the state of the app

If there's a state you want to preserve between restarts of your app (e.g. a list of telegram chats to send the new ads to, a list of already processed ads) you need a database. The disk is out of the question because you don't get persistent storage in GAE. Firestore is the only DB available in the GCP free tier.

Firestore takes a bit of getting used to. I suggest watching a couple of [_Get to know Cloud Firestore_](https://www.youtube.com/watch?v=v_hR4K4auoQ&list=PLl-K7zZEsYLluG5MCVEzXAQ7ACZBCuZgZ) videos. They are excellent — short and to the point.

Firestore offers an emulator which you can use locally for integration tests:

- Start it with
  ```
  $ gcloud beta emulators firestore start --host-port=localhost:8068
  ```
- Then pass `FIRESTORE_EMULATOR_HOST=localhost:8068` to the tests.



#### Evaluate FaaS as an alternative

If you outsource the state to the database (you have to because the app may be scaled down to 0 nodes), use queues/cron (the app may be scaled down to 0), avoid initialization code (the app is not guaranteed to run after the deployment) then your app becomes a collection of HTTP request handlers interacting with the DB.

At this point, you may consider using a Function-as-a-Service platform (e.g. GCP Cloud Functions, AWS Lambda).


## Summary

A telegram bot is an excellent option as the UI facade for your personal project. You can run it for cheap in Google App Engine. After the initial learning curve, the platform is pleasant to work with and requires little maintenance. I hope this post will give you an idea of the challenges involved in developing for GAE.


[botapi]: https://core.telegram.org/bots/api "Telegram Bot API"
[gcpfreetier]: https://cloud.google.com/free "GCP free tier"
