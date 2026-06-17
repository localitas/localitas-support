# Why We Built Localitas

## We looked at the future and didn't like what we saw

When we looked at where computing is heading, the picture was pretty clear: everything is rented. Cloud computing, AI, storage — it's all someone else's infrastructure that you pay to use. Per-token. Per-seat. Per-month. Forever.

That means by default, people won't own anything. Not their compute. Not their data. Not even the AI they talk to every day.

We wanted to build an alternative to that future.

The thing is — the hardware is already here. A Mac Mini costs $599 and can run a 35-billion parameter AI model. It uses 5 watts idle. That's like $5 a year in electricity. What used to cost thousands per month in cloud GPUs now sits quietly on your desk.

What was missing was the software to make that hardware actually useful — to turn it into something you can build on, deploy from, and access from anywhere in the world. So we built it.

Localitas lets you run AI without per-token fees. Build apps and ship them to the internet without a hosting bill. Store your photos, files, and passwords on hardware you physically own. And when you need more power, you just plug in another Mac.

We're not against the cloud. It's great for a lot of things. But we think you should have a choice. Right now, for most people, there isn't one. We'd like to change that.

## Why isn't everything open source?

A lot of what we built is open source. The standalone apps — calendar, contacts, email, notes, weather, maps, dictionary, stocks, and more — they're all MIT-licensed on [GitHub](https://github.com/localitas). Fork them, change them, make them yours.

But the core engine and the SaaS tunnel service aren't open source. We want to be honest about why.

We're engineers with young families. We have kids, mortgages, and not enough hours in the day. We love building this, but we also need to eat. Open-sourcing everything sounds great in a blog post, but it doesn't pay for daycare.

So here's how we set it up:

- **The apps are fully open source.** MIT license. Do whatever you want with them.
- **The core engine is free to use.** Download it, install it, run it. You don't pay us for the software.
- **The tunnel service is how we keep the lights on.** Running edge servers, managing DNS, provisioning SSL — that costs real money. The Family Plan at $20/month covers that and funds development.
- **We never see your data.** The tunnel is just a pipe. We can't read your files, your messages, or your AI conversations. We don't want to.

If the project grows to where we can open-source the core and sustain it through community support, we absolutely will. But right now, we're choosing to be sustainable so we can keep showing up and building this thing for years to come.

We hope that makes sense.
