<!--
.. title: So, you want to start a side project?
.. slug: side-project
.. date: 2023-12-03 00:00:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->

Over the years, I have started or worked on many side projects and spent a lot of time maintaining them. This has taught me a lot about what makes a project easy and hard to maintain. This post is a reflection on some of the lessons learned over that time.

First, lets start off with some assumptions:

1. You want to start a side project, not a side hustle. A side hustle is an entrepreneurial exercise. It is something you eventually want to turn into a business or job, even if it is not on day one. The objective of a side project is the creative satisfaction of the project itself. It might be a learning experience, or exploration of personal interests.
2. Your side project is software or programming related.
3. The project is in some way public. It might be open source. You want to make something with a userbase or community beyond just yourself.
4. Crucially, you care about maintaining this project over a period of time. It is not just a throwaway learning exercise.

If you are looking to start a side project, this implies you have some time on your hands right now. This is a great place to be, but it won't be true forever. Life happens, and it usually happens unexpectedly.

So, how can we optimise for projects that are low-maintenance or at least projects that require the type of maintenance that can be done on our own terms? A side project that may require attention urgently and unexpectedly can quickly become a burden.

## Third party APIs

Third Party APIs are one of the most likely sources of sudden and unexpected maintenance tasks. You may or may not get warning when changes happen. If you use an API with authentication or credentials, you probably had to sign up for an account so the upstream service provider probably does at least hold some contact details they could use to inform you of changes. Expect the unexpected from any API where you use public or anonymous access.

Sometimes an API you depend on will:

- Make a non-backwards compatible change.
- Introduce rate limits or enforce stricter rate limits.
- Change their terms of service such that your project now violates them in some way. 
- Withdraw service completely or shut down.

All of this is very in vogue at the moment.

## Scrapers

Web scrapers are like third party APIs, only worse. API authors expect that other people's code depends on their API. They may still choose to make a breaking change anyway, but there is some incentive there to maintain a stable platform for their users. Nobody assumes or cares that your code relies on scraping their website. It certainly won't be a consideration in changing it. You definitely won't get any communication informing you of a change that impacts your code. Website authors may even be actively trying to prevent you from scraping them. Code that relies on web scraping is certain to break at some point. It is matter of *when*, rather than *if*.

## Infrastructure

Any kind of infrastructure you run (web servers, DB, cache, etc) is going to come with some maintenance overhead. Applying security upgrades, backup and restore, ensuring uptime, etc. The exact tasks that come up will vary a bit depending on the type of infrastructure, but could include a mix of tasks that can be planned in advance and things that happen unexpectedly. You can plan or defer applying an upgrade, but data loss requiring a restore from backup will happen when you least expect it.

There are some tradeoffs to be made here. Using a managed service can outsource some of this maintenance. For example, if we consider something like a Postgres DB: Running your own Postgres instance on a VPS leaves everything up to you (of course, with a side project, managing this yourself could be part of the joy or satisfaction). A fully managed service like Heroku Postgres will handle most of this stuff for you transparently. Something like RDS or Fly.io's "semi-managed" Postgres sits somewhere in the middle of those two extremes.

A fully managed service comes with some costs though. A managed service can be it's own source of breaking changes or deprecations. Some platforms have a greater or lesser reputation for stability (think AWS vs GCP 😀). The more obvious cost is the literal financial cost though, which brings us on to..

## Finances

If you go down the route of a project that requires some sort of infrastructure, that has a cost associated with it, and somebody needs to cover that. Maybe you will pay for it out of your own pocket. In general, side projects are not revenue generating and need to run on a modest budget. Often that budget will be zero. Maybe you can run a service using free tier offerings. However, even if you're using it for free, someone is still paying. For the moment, the company offering that "free" service is covering that cost out of marketing budget because offering a free tier is good promotion, but that might not stay true forever. If your project is popular, you might also consider soliciting some sponsorship to cover your costs, either from your community or a corporate sponsor.

Again, funding concerns are another common source of suddenly urgent work.

- Your project may become more popular, outgrowing your current pricing plan or the level of sponsorship your project currently attracts.
- If your project runs on a free tier service, that offering will probably be withdrawn at some point. Most of them are eventually.
- If you have a corporate sponsor, also assume it will not last forever. Sponsorships of open source and community projects are often the first things to be cut when times are tough.

All of this can leave you quickly scrambling to migrate to another service, find ways to consume fewer resources, or present an immediate existential threat to your project.

## Personal data

nopenopenopenopenope

If your side project stores personal data, congratulations. You now have compliance obligations. Choo choo 🚂 All aboard the fun train! A service that stores any kind of personal data (e.g: user accounts) is not a good choice for a side project. This one is a hard no from me.

## It's not all doom and gloom

That is a list of things that can, to one degree or another, generate some maintenance overhead. So what are some types of projects that don't have any of those characteristics (or at least as few of them as possible)?

## Command line applications (compiled language)

If your project is distributed as a compiled binary and it doesn't call any external APIs, there are relatively few externalities that can break this type of project or require attention from a maintainer. This is even more true for a statically linked binary. The only real exception to this might be needing to respond to a security issue.

## Command line applications (dynamic language) or Libraries

This type of project has similar properties to a compiled command line tool. However, with anything that the user installs via `pip install`, `npm install`, etc, your dependencies are resolved at install time. This means your previously working code can be broken for some users by non-backwards compatible changes made in an in-range dependency version. In theory SemVer saves us here, but in most languages (other than javascript) it is necessary to support wide ranges. This type of breakage is not super common, but it does happen.

## Static sites

Static content is good content. If you have the type of static site that can be served from a S3 bucket, there are multiple places that will host it for free and scale it to handle as much traffic as the internet can throw at it. If you do need to move it, it is relatively easy and you have zero infrastructure to maintain.

For a low-maintenance project side project that involves a website, "could this be a static site" is generally a good question to ask. Sometimes by making a compromise or two, it is possible to get rid of a web server and DB and replace them with a static site. This is usually an advisable tradeoff. A good example of this might be choosing a SSG for your blog, instead of hosting a CMS.

It is worth noting that this is not true of the type of "static site" which is heavily tied to the specific features of a platform like Vercel or Netlify. These basically have the same tradeoffs as managed infrastructure with the additional downside of vendor lock-in.

## End

So, that's some thoughts on the characteristics of a low maintenance side project. Go forth. May your side project bring you many hours of joy and few unexpected urgent maintenance issues.
