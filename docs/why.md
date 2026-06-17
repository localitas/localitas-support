# Why We Built Localitas

## The Future of Computing Shouldn't Be Rented

When we looked at the future computing landscape, we saw a clear trajectory: everything is moving toward rented infrastructure. Cloud computing, AI APIs, storage, even the software you use daily — it's all pay-per-use, pay-per-seat, pay-per-token. By default, the next generation of developers, creators, and families won't own any of it.

We think there should be an alternative.

Consumer hardware has caught up. A $599 Mac Mini with Apple Silicon can run a 35-billion parameter AI model. It has unified memory that lets the CPU and GPU share the same pool — no expensive dedicated GPUs needed. It uses 5 watts at idle. That's $5 a year in electricity to run what would cost thousands per month in cloud GPU rentals.

The hardware is ready. What was missing was the software to make it useful — something that turns a computer sitting on your desk into a platform you can build on, deploy from, and access from anywhere. That's what Localitas is.

We built Localitas so that you can:

- **Run AI locally** without per-token charges or your data being used to train someone else's model.
- **Build and ship apps** to the internet from your own hardware, without cloud hosting bills.
- **Store your files, photos, and credentials** on hardware you physically control.
- **Scale by adding machines**, not by upgrading subscription tiers.

We're not anti-cloud. Cloud infrastructure is excellent for many use cases. But we believe you should have a choice — and right now, for most people, there isn't one. Localitas is that choice.

## Why Isn't Everything Open Source?

Many of the apps we built are fully open source and MIT-licensed. You can find them at [github.com/localitas](https://github.com/localitas) — the calendar, contacts, email, notes, weather, maps, dictionary, news, stocks, and more. Each is a standalone app you can fork, customize, and deploy.

The core engine and the SaaS tunnel service are source-available but not open source. We want to be transparent about why.

We're a small team of engineers with young families. We have mortgages, daycare bills, and limited hours in the day. Open-sourcing everything sounds noble in the abstract, but it doesn't pay for groceries. The core engine and the tunnel infrastructure represent years of work — Raft consensus, distributed SQLite replication, secure tunneling, app orchestration — and we need to sustain that work financially.

Here's what that means in practice:

- **The apps are open source.** Fork them, modify them, contribute back. MIT license, no strings attached.
- **The core engine is free to use.** You download it, install it, run it on your hardware. You don't pay us for the software.
- **The tunnel service is how we sustain the project.** It costs us real money to run edge servers, manage DNS, provision SSL certificates, and maintain uptime. The Family Plan ($20/month) covers that infrastructure and funds ongoing development.
- **Your data never leaves your hardware.** The tunnel is just a pipe — we don't see, store, or process your traffic. We can't read your files, your chat messages, or your AI conversations. The tunnel service is a relay, not a platform.

We believe this is a fair trade. You get a powerful, private, self-hosted platform with a growing ecosystem of open-source apps. We get to keep building it without burning out or taking venture capital that would pressure us to enshittify the product.

If the project grows to the point where we can open-source the core and sustain it through community support alone, we will. Until then, we're choosing sustainability over ideology — and we hope you understand why.
