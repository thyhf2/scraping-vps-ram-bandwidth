# Scraping VPS: How Much RAM Do You Need, What Should It Cost, and What Happens When the Target Blocks You? (Plan Sizes, Bandwidth Math, Billing for Short Jobs)

If your scraper fetches HTML with requests, httpx or Scrapy and never opens a browser, 1 vCPU and 2 GB of RAM is enough machine, and a €5.00/mo server will spend most of its life waiting on the target site. If it drives Playwright or Puppeteer, budget 500 MB to 1 GB of RAM per concurrent browser context and start your shopping at 8 GB. A VPS fixes the three things that keep breaking a laptop crawl: the machine sleeping, the home IP changing, and a job that runs longer than your working day.

What follows is the sizing arithmetic, the bandwidth translation for a million pages, the part of blocking that no server fixes, and Ava Hosting's current plans with prices, so you can pick a size and stop reading.

> **Quick picks.** HTTP-only crawl: Ava Hosting VPS One, 1 vCPU / 2 GB ECC RAM / 25 GB NVMe, €5.00/mo, unlimited bandwidth. Playwright with up to about eight contexts: VPS Extra, 6 vCPU / 12 GB / 120 GB NVMe, €30.00/mo. A one-off weekend job: check the billing section first, because Ava bills monthly and an hourly host may cost less for three days of work.
>
> 👉 [See Ava Hosting's current VPS plans and prices](https://my.ava.hosting/aff.php?aff=15562)

## Ava Hosting's plans, and which ones fit a crawler

Ava Hosting lists two catalogues that overlap heavily, so it helps to know which one you're looking at. The VPS/VDS line runs from a €5.00 monthly entry plan to 16 GB:

| Plan | vCPU / RAM / NVMe | Port | Price | Order |
| --- | --- | --- | --- | --- |
| VPS Nano | 1 / 512 MB ECC / 20 GB | 100 Mbps | €11.00/yr | [Order VPS Nano](https://my.ava.hosting/aff.php?aff=15562) |
| VPS One | 1 / 2 GB ECC / 25 GB | 1 Gbps | €5.00/mo | [Order VPS One](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Basic | 2 / 4 GB ECC / 50 GB | 1 Gbps | €10.00/mo | [Order VPS Basic](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Lucky | 2 / 6 GB ECC / 70 GB | 1 Gbps | €15.00/mo | [Order VPS Lucky](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Power | 4 / 8 GB ECC / 90 GB | 1 Gbps | €20.00/mo | [Order VPS Power](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Extra | 6 / 12 GB ECC / 120 GB | 1 Gbps | €30.00/mo | [Order VPS Extra](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Ultra | 8 / 16 GB ECC / 160 GB | 1 Gbps | €40.00/mo | [Order VPS Ultra](https://my.ava.hosting/aff.php?aff=15562) |

The plan pages list unlimited bandwidth, free DDoS protection, weekly backups, full root access and a control panel on Intel Xeon E5 hardware with NVMe storage and a 99.9% uptime SLA. The second catalogue, Self-hosted Start, prices the same CPU and RAM at the same monthly rate with different disk allocations, plus a €2.00 entry point:

| Plan | vCPU / RAM / NVMe | Price | Order |
| --- | --- | --- | --- |
| Start | 1 / 1 GB / 15 GB | €2.00/mo | [Order Start](https://my.ava.hosting/aff.php?aff=15562) |
| Mini | 1 / 2 GB / 25 GB | €5.00/mo | [Order Mini](https://my.ava.hosting/aff.php?aff=15562) |
| Basic | 2 / 4 GB / 50 GB | €10.00/mo | [Order Basic](https://my.ava.hosting/aff.php?aff=15562) |
| Pro | 4 / 8 GB / 100 GB | €20.00/mo | [Order Pro](https://my.ava.hosting/aff.php?aff=15562) |
| Business | 6 / 12 GB / 150 GB | €30.00/mo | [Order Business](https://my.ava.hosting/aff.php?aff=15562) |
| Power | 8 / 16 GB / 200 GB | €40.00/mo | [Order Power](https://my.ava.hosting/aff.php?aff=15562) |

Two details are worth catching before you click anything. From €20.00 upward, the Self-hosted Start line lists more disk for identical money: 100 GB against 90 GB at €20.00, and 200 GB against 160 GB at €40.00. Disk is what fills up when you save screenshots and keep a local queue. The other detail is the €2.00 Start plan. One Chromium tab on a real page can use 400–700 MB by itself, so that plan is a mail server or a static site, not a scraping box. Even for plain HTTP work, 1 GB gets tight the moment Redis or a local Postgres joins the process.

## Do the RAM arithmetic before you compare prices

Published numbers on headless memory, from the hosts that document it:

- A chrome-headless-shell process with one blank tab sits at 180–250 MB.
- Load a real page (React app, product listing, infinite scroll) and it climbs to 400–700 MB per tab.
- Playwright recording trace or video adds roughly 100 MB per context.
- A practical planning figure is 500 MB per concurrent tab, then 20% headroom for the OS, the Python or Node runtime, and Redis.
- A second, blunter rule from another provider: about 1 GB of RAM per concurrent browser instance.

Those estimates differ by nearly a factor of two, which is the whole reason to plan at the top of the range and then check actual RSS on your own target pages after the first run. The formula is short:

**RAM ≈ concurrent contexts × 500 MB × 1.2**, and swap 500 MB for 1 GB if you record traces or video.

Run it on a few real configurations:

- 4 contexts, no recording: 2.4 GB, so Ava Hosting VPS Basic at 4 GB (€10.00/mo) with room left for the queue.
- 8 contexts, no recording: 4.8 GB, so VPS Lucky at 6 GB (€15.00/mo).
- 8 contexts with trace or video: 9.6 GB, so VPS Extra at 12 GB (€30.00/mo).
- 15 contexts, no recording: 9 GB, still VPS Extra. The same 15 contexts with traces need 18 GB, and VPS Ultra at 16 GB falls short. Cut the context count or add a second box.

CPU comes second, but it isn't irrelevant: JavaScript execution, layout and screenshots all burn cycles, and a 1 vCPU machine with 8 GB would be lopsided. Ava Hosting's 2 GB of RAM per vCPU is a sensible ratio for memory-bound browser work. Disk is the constraint people forget, because a few thousand full-page PNGs grow faster than HTML does, and crawl logs plus dedup state share the same volume.

👉 [Compare the 8 GB VPS Power and the 12 GB VPS Extra](https://my.ava.hosting/aff.php?aff=15562)

## Bandwidth: a million HTML pages is about 50 GB

A million HTML pages comes to roughly 50 GB of transfer. The same million pages rendered in a headless browser, images and fonts included, is several times that. Run the arithmetic down and 50,000 pages is about 2.5 GB of raw HTML, or somewhere around 10 GB once a browser renders them.

Crawls are download-heavy and bursty. A job that pulls tens of thousands of pages overnight is ordinary, and that spike is where metered bandwidth turns into a line on an invoice. On a plan with a monthly allowance, the crawl either gets throttled at the cap or billed as overage. Ava Hosting lists unlimited bandwidth on every VPS plan, so the overnight spike doesn't produce a surprise charge, and the 1 Gbps port (100 Mbps on VPS Nano) keeps proxy throughput and result uploads from bottlenecking. Upload speed is the half people ignore: home connections typically upload far slower than they download, and browser traffic through a proxy chain feels that.

## What a VPS does not fix

An IP that sends too many requests gets blocked, and slowing down usually works better than pushing harder. Against a target running Cloudflare, DataDome, Akamai or PerimeterX, the ceiling belongs to any single address the target can fingerprint. No plan size moves that ceiling.

The arrangement that does work separates the jobs. Orchestration, storage and API-friendly targets stay on the server's static IP, and browser traffic goes out through rotating residential proxies. Hosts that sell scraping servers describe the same split; ghostrdp states plainly that the VPS is for orchestration, storage and endpoints that don't care about residential origin, with the actual browser sessions behind proxies.

Shared addresses cause the second problem. If other tenants' crawlers share your IP, their behaviour can land you in a rate-limit bucket before your first request goes out. A dedicated IPv4 isolates your reputation. Ava Hosting's plan pages list port speed, bandwidth and DDoS protection but don't state whether the IPv4 is dedicated, so ask support before you design a crawl that depends on IP reputation. Ask the same question about a burned address: some hosts treat a fresh server as the reset button, and Servury's documentation recommends deploying a new server to get a new IP, while NorthHost states outright that each VPS keeps a fixed IPv4 with no rotation.

One compliance decision belongs in the design stage rather than the debugging stage. If the pages you collect contain personal data, including anything tied to EU residents, where that data is stored and how long you keep it is a choice you make before the crawl starts.

## Billing: monthly plans against three-day jobs

Ava Hosting bills monthly, with a single annual plan (VPS Nano at €11.00/yr). Its refund policy offers a 30-day money-back window, limited to first-time hosting accounts, excluding services bought with promotional codes or discounts, and void if the acceptable-use policy is violated. That window is the longest among the hosts compared here: ghostrdp offers 48 hours on a first order, Infrawire 7 days.

For a crawl that runs for a weekend, hourly or daily billing beats any monthly plan. noded.cloud bills its VPS I at €0.0056/hr and deploys in about a minute, so a burst job means spinning up a bigger box and deleting it the same day. Servury sells by the day and describes renting five servers for a three-day job, then deleting them. Ava Hosting publishes no hourly option, so flat monthly pricing wins only when the workload is continuous.

Where that flat pricing lands: VPS One at €5.00/mo for 1 vCPU, 2 GB and 25 GB is in line with noded.cloud's VPS I at €4/mo for 2 GB, 1 vCPU and 20 GB. At 8 GB the picture is less one-sided. noded.cloud lists VPS III at €10/mo for 8 GB, 2 vCPU and 60 GB; ghostrdp lists Standard at €12.99/mo for 4 vCPU, 8 GB and 70 GB; Infrawire lists Bronze at €14.55/mo for 4 cores, 8 GB and 80 GB NVMe, shown against a €24.25 price the page treats as the original without explaining what renewal costs. Ava Hosting's VPS Power is €20.00/mo with 4 vCPU, 8 GB and 90 GB. The extra buys a larger NVMe volume, a 1 Gbps port, weekly backups and the 30-day refund window, and that trade is worth taking only if you'll use the disk and the backups.

👉 [Check Ava Hosting's full VPS lineup and current monthly rates](https://my.ava.hosting/aff.php?aff=15562)

## When the laptop is the better machine

A few hundred pages a day, or a crawl you start once a week while sitting at your desk, doesn't need a server. The laptop is free and your environment is already on it.

A single target site that starts returning 429s after 40,000 requests won't improve with more cores or more RAM. That money goes further on proxies and a slower request rate.

If what you actually need is many short-lived IPs, several small servers or a rotating proxy pool do more for you than one large machine.

## The buying rules

- Parsing HTML only, no browser engine: pick the smallest plan that fits your queue. VPS One at €5.00/mo covers a sitemap crawl or a price monitor; move to VPS Basic at €10.00/mo if Postgres and Redis live on the same box.
- Running N Playwright contexts: RAM at or above N × 500 MB × 1.2, using 1 GB per context when you record traces. Take the plan above your result, not the one that matches it exactly.
- Crawling around the clock with many instances: one box stops being the answer. Split into a hub and worker nodes, or connect several small servers over a private network.

Two things to confirm before you pay: whether the IPv4 on your chosen plan is dedicated, and what the renewal price is, since Ava Hosting publishes "starting from" monthly rates without renewal terms on the plan pages.

👉 [Pick a plan on Ava Hosting and check today's prices](https://my.ava.hosting/aff.php?aff=15562)
