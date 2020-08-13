# deep-unfurling

This project is at this point a sketch - nothing concrete to see here!

## What is Deep Unfurling?

Unfurling is when links are expanded when shared in social media to show descriptive text as well as preview images. Typically, interactive visualizations do implement unfurls, but with a single static image. However, if a user navigates to a specific view, then shares that link, typically the unfurl image is not representative of what the user actually sees when sharing the link. This is where the notion of “deep unfurling” comes in.

Imagine you navigate to the view for a particular US State, for example California. Imagine if you shared that link (URL parameters included) on social media, and the unfurl image were specific to that state. Moreover, what if you navigated to a previous point in time by clicking on the line chart, and were able to share that link to social media and get an unfurl image that shows that particular state at that particular point in time, exactly as you see it. This is the concept of “deep unfurling”.

## How to make it happen?

Unfurling is implemented with meta tags in the HTML source that is rendered from the server. For dynamic unfurls, we’ll need dynamic server-rendered HTML to populate the unfurl tags based on the URL parameters specified when the page is loaded. That’s not to say we need to take advantage of “server side rendering”, or change anything about the way our app is structured. Only the “index.html” content needs to be dynamic. In particular, the image source URL will be different for each unique combination of parameters that we want to expose (e.g. selected geography, selected time slice). But where must that image come from?

To serve deep unfurl images, we should put the following components into place:
 * an image generation server to take automated screenshots on demand, and
 * a cache of previously rendered automated screenshot images.

When someone shares a parameterized link in social media, the dynamically gendered unfurl meta tags will point to an image source URL that will hit an endpoint with roughly the following logic:
 * If a screenshot is in the cache for this combination, return it.
 * Otherwise generate the image, stash it in the cache, and return it.

This outlines how the deep unfurling system generally can function.

## Will it scale?
Scalability concerns will most definitely arise. These concern disk storage capacity and compute capacity of the servers involved. To get a rough estimate of the number of possible unfurl images we’ll need to potentially generate, consider 50 US states, around 100 dates, and (if we want to unfurl the “pinned” region) around 150 counties. That’s roughly 750,000 possibilities. 

Given the high number of possible unfurl images, we should adopt a caching strategy such that previously generated images should be kept around for future use, but also may expire and fall out of the cache. Ideally we should be able to set a fixed maximum amount of storage to be used for storing unfurl images, say 2GB.

There’s also the issue of the images becoming obsolete with respect to the data shown. Since the data behind the graphics updates daily, we may want to set a cache expiry time for each unfurl image, or even clear out the cache entirely every time we roll out a data update.

Generating the automated screen capture images is very computationally intensive. An entire headless browser needs to be spun up, load the app, load the data, and render it. We can estimate it might take around 5 seconds to generate a single automated screenshot (based on experience implementing a similar system for VizHub.com). So as not to overwhelm the capacity of the image generation server, we should implement a queue system such that multiple automated screenshots are done in parallel as needed, but up to a certain concurrency limit (say 5 screenshots being in the generation process at any given moment).

Another dimension of scalability is the compute capacity required to run the server. On the server side there are two primary activities: serving the app (including dynamically rendering the unfurl meta tags, and serving the images from cache), and generating the images. If the volume of traffic to the site is high, then it may be necessary to split up these computational loads between two server machines.

## What tech?
To implement this system, we would use:
 * Node.js with Express.js for serving the app.
 * Puppeteer for generating automated screenshots using headless Chrome.
 * Redis for caching.
 
## Executive summary
To summarize what’s needed:

 * Server-side logic to serve the app with dynamic unfurl metadata.
 * Secondary server dedicated to generating and storing unfurl images.
 * Cache system for unfurl images.
