# ITE 116 – Integrative and Programming Technologies
## Reviewer: Cloud Deployment
### For the 30-item quiz

> **Coverage:** what deployment is, what the cloud is, what a Laravel app needs to run, self-hosting vs the cloud, IaaS/PaaS/SaaS, core cloud concepts, environments and `.env`, the deployment pipeline, and Laravel Cloud.
> **Not included:** moving from SQLite to a managed database (ephemeral filesystems, object storage).

---

## 1. Deployment

**Deployment** is taking your application from your computer and running it on a **server** that is:

- **reachable** by anyone on the internet through a real address (a domain);
- **always on**, even when your laptop is closed;
- **configured for real users**: secure, fast, and holding real data.

| Development (your laptop) | Production (a server) |
|---|---|
| `http://127.0.0.1:8000` | `https://notes.example.com` |
| Runs only while the terminal is open | Runs 24/7 |
| One user: you | Many users at once |
| Error pages show full details | Error pages hide details |
| Fake data you can wipe | Real data that must never be lost |
| `npm run dev` | CSS and JS built **once** (`npm run build`) |

- `127.0.0.1` (localhost) means **"this computer only"**. Nobody else can open your app.
- **"It works on my machine"** is the classic excuse. A project that only runs on one laptop cannot serve its client.

---

## 2. The Cloud

**The cloud is someone else's computers, rented over the internet.** It runs in real buildings called **data centers**, full of servers, storage, network equipment, backup power, and cooling, owned by providers such as **AWS, Google, and Microsoft**.

**Cloud computing** = renting computing resources (servers, storage, databases) over the internet, **on demand**, and **paying for what you use**.

### The five essential characteristics (NIST definition)

| Characteristic | Remember it as |
|---|---|
| **On-demand self-service** | You get resources yourself, instantly, without calling anyone |
| **Broad network access** | Everything is reached over the internet |
| **Resource pooling** | Many customers share one huge pool of hardware |
| **Rapid elasticity** | Resources grow and shrink quickly with demand |
| **Measured service** | Usage is metered and billed |

- You already use the cloud daily: **Gmail, Google Drive, Canva, MS Teams**.

---

## 3. What a Laravel App Needs to Run

A Laravel app is a **program that must be running**, not a file you upload. To serve it to the world you need:

| Piece | Job |
|---|---|
| An **always-on computer** | Runs everything 24/7 |
| A **public address and a domain** | Lets the world find the server |
| A **web server** (Nginx, Apache) | Receives HTTP requests and passes them to PHP |
| The **PHP runtime** + **Composer packages** | Runs Laravel and its dependencies |
| **Built front-end assets** | Compiled Tailwind CSS and JS |
| A **database server** | Stores data permanently |
| An **HTTPS certificate** | Encrypts traffic (passwords!) |
| **Updates, backups, monitoring** | Keep it safe and recoverable |

- `php artisan serve` is a **development** server, not for production.

###   Is the cloud required to deploy a framework?

**No. A server is required.** The server can be **your own (self-hosting)** or **rented (the cloud)**. The cloud is just the most practical way for most teams to get one.

---

## 4. Self-Hosting vs Cloud Deployment

- **Self-hosting:** you run the server yourself (an office PC, the school server room, a Raspberry Pi).
- **Cloud deployment:** you rent the server or a platform from a provider.

| | Self-hosting | Cloud |
|---|---|---|
| Up-front cost | Buy the hardware | Little or none |
| Ongoing cost | Electricity, internet, repairs, your time | Usage-based bill |
| Who manages the stack | **You** (OS, web server, PHP, database, SSL, patches) | Partly or fully the **provider** |
| Public access | Needs a public IP + port forwarding; **CGNAT** (ISP shares one IP among many customers) can block it | Public address immediately |
| Uptime | Down with **brownouts**, ISP outages | Redundant power, cooling, network |
| Scaling | Buy another machine | Change a setting |
| Backups | You set them up | Usually built in |
| Security | Fully yours | Provider secures the platform; you secure your app |
| Control | **Total** | Limited to what the platform allows |
| Data location | **In your building** | In the provider's region |
| Skills needed | System administration, networking, Linux | Mostly Git and a dashboard |

**Choose self-hosting when:** the system is used only inside one office (an **intranet**), data **must stay on-site**, or you have hardware, stable power, and a skilled admin.

**Choose the cloud when:** users need access **from anywhere**, you need **reliability**, there's **no system administrator**, or traffic is **small or unpredictable**.

---

## 5. Kinds of Cloud Services: IaaS, PaaS, SaaS

They differ in **how much the provider manages for you**.

| Layer | On-premise | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Application code | You | You | You | Provider |
| Data | You | You | You | Provider (you own it) |
| Runtime (PHP) | You | You | **Provider** | Provider |
| Operating system | You | You | **Provider** | Provider |
| Servers, storage, network | You | **Provider** | Provider | Provider |
| Data center | You | **Provider** | Provider | Provider |
| **Example** | School server room | AWS EC2, DigitalOcean | **Laravel Cloud**, Heroku, Vercel | Gmail, Canva, Google Docs |

**Pizza analogy:**
- **On-premise:** make pizza at home (buy the oven, everything).
- **IaaS:** rent a kitchen with an oven; you still cook.
- **PaaS:** bring your recipe; the restaurant bakes it.
- **SaaS:** just order a pizza.

  **Laravel Cloud is a PaaS.** We bring the **code**; it provides the servers, PHP runtime, patches, and scaling. We never log in to a server or install PHP.

### Shared responsibility

The provider secures the platform; **you secure what you put on it.** Always your job, even on a PaaS:
- the security of **your code** (validation, authentication, authorization);
- your **secrets** (`APP_KEY`, passwords, API keys);
- your **data** and who can access it;
- your **bill**.

---

## 6. Core Cloud Concepts

| Term | Meaning |
|---|---|
| **Region** | A geographic location of a provider's data centers |
| **Latency** | The delay caused by the distance data travels. **Rule:** choose the region **closest to your users** |
| **Vertical scaling** (scale up) | Give one server more CPU and RAM. *A bigger jeepney* |
| **Horizontal scaling** (scale out) | Run more copies of the app side by side. *More jeepneys on the route* |
| **Autoscaling** | The platform adds or removes copies automatically with traffic. *A dispatcher at rush hour* |
| **Replica / instance** | One running copy of the app |
| **Load balancer** | Spreads incoming requests across replicas |
| **Scale-to-zero** | The app sleeps when idle (no compute bill) and wakes on the next request. Trade-off: the first visitor waits slightly longer |
| **Pay-as-you-go** | Billed by usage: per second of compute, per GB stored, per GB transferred |

- With horizontal scaling, all replicas use **one shared database**.
- Pay-as-you-go risks: a **forgotten environment** keeps costing money; a runaway bug or traffic spike raises the bill. **Set spending limits and delete what you don't use.**

---

## 7. Environments and `.env`

| Environment | Purpose | Who uses it |
|---|---|---|
| **Local / development** | Writing and testing code | You |
| **Staging** | A copy of production for final testing | Team, adviser, testers |
| **Production** | The real system | Real users |

  **Never test on production.** A mistaken `migrate:fresh` deletes every real user's data, and there is no undo.

**Same code, different configuration.** Configuration lives in environment variables (`.env`), not in the code. This is a principle of the **Twelve-Factor App**.

| Variable | Local | Production | Why |
|---|---|---|---|
| `APP_ENV` | `local` | `production` | Tells Laravel which mode it's in |
| `APP_DEBUG` | `true` | **`false`** | Debug pages reveal file paths and code, a gift to attackers |
| `APP_KEY` | from `key:generate` | **its own secret** | Encrypts sessions and cookies; never share or commit it |
| `APP_URL` | `http://127.0.0.1:8000` | `https://your-domain` | Builds links and asset URLs |

  **`.env` never goes to GitHub.** Laravel's `.gitignore` already excludes it. On a cloud platform, enter the values in the dashboard.

---

## 8. The Deployment Pipeline

```
git push ──▶ 1. FETCH ──▶ 2. BUILD ──▶ 3. DEPLOY ──▶ 4. RELEASE ──▶ live
```

| Stage | What happens | Laravel commands |
|---|---|---|
| **1. Fetch** | Pull the latest commit from the Git branch | — |
| **2. Build** | Install dependencies, compile assets | `composer install --no-dev` · `npm run build` |
| **3. Deploy** | Run one-time tasks right before going live | `php artisan migrate --force` |
| **4. Release** | Switch traffic to the new version | handled by the platform |

| Term | Meaning |
|---|---|
| `--no-dev` | Skip packages only needed while developing; the app is smaller and safer |
| `npm run build` | Vite compiles CSS and JS **once** into `public/build`. Skip it and you get **"Vite manifest not found"** |
| `migrate --force` | Laravel **refuses** to run migrations in production without `--force` (a safety check) |
| **Zero-downtime deployment** | The new version starts **before** the old one stops |
| **Push to deploy** | Every `git push` to the chosen branch starts a deployment |
| **CI/CD** | Continuous Integration / Continuous Deployment: automatically testing and releasing every change |
| **Rollback** | Return to the previous working version |

  **If it isn't committed and pushed, it isn't deployed.** Commit `composer.lock` and `package-lock.json` so the server installs exactly the versions you tested. Never push `.env`, `vendor/`, or `node_modules/`.

---

## 9. Laravel Cloud

**Laravel Cloud** is the official deployment platform from the Laravel team. It is a **PaaS**.

```
Organization (your account / team)
 └── Application  — one project, one Git repository
      ├── Environment: production (branch: main)
      │     ├── Compute    — replicas running your code
      │     ├── Resources  — database, cache, object storage
      │     └── Domain     — your-app.laravel.cloud / custom domain
      └── Environment: staging (branch: develop)
```

| Term | Meaning |
|---|---|
| **Application** | One project, connected to one GitHub, GitLab, or Bitbucket repository |
| **Environment** | One running copy of the app (production, staging) with its own branch, settings, compute, and resources |
| **Compute** | The servers that run your code: **Flex** (lightweight, can scale to zero) or **Pro** (higher performance, always on) |
| **Resources** | Managed services attached to an environment |
| **Infrastructure canvas** | The visual dashboard where you attach compute and resources |

**Managed resources:** **Laravel MySQL** (daily backups), **Laravel Serverless Postgres** (can scale to zero), **Laravel Valkey** (Redis-compatible cache), **Laravel Object Storage** (S3-compatible files). Attaching a resource **injects its connection variables automatically**.

**What Laravel Cloud does for you:**
- **Push to deploy** (on by default), with build and deploy commands you set once
- **Zero-downtime** releases
- A free **`laravel.cloud` domain** per environment, plus **custom domains**
- **Automatic HTTPS** certificates
- **Load balancing**, and **autoscaling** on higher plans
- **Scale-to-zero**
- **Edge network** with DDoS protection (with Cloudflare)
- **Logs and metrics**, and a **Commands** tab for one-off commands (e.g., `php artisan db:seed --force`)

**Cost:** usage-based. Starter plan: **$5/month with $5 of usage credit** (as of November 2026).

---

