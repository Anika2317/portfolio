---
layout: post
categories:
- CSP
title: "Deployment Lesson: Getting Your Project on the Internet"
description: A full class lesson on deploying a team frontend and backend for Open Coding Society, covering domains, DNS, IPs, ports, Docker, nginx, HTTPS, and Certbot
permalink: /csp/deployment-lesson
author: Anika
---

<style>
.post-content table { width: 100%; display: table; table-layout: auto; }
.post-content th, .post-content td { text-align: left !important; vertical-align: top; overflow-wrap: anywhere; }
.post-content pre, .post-content pre code { white-space: pre-wrap; overflow-wrap: anywhere; }
.post-content details { border: 1px solid #888; border-radius: 6px; padding: 8px 12px; margin: 8px 0; }
.post-content summary { cursor: pointer; font-weight: bold; }
</style>

## Why this lesson matters

Right now your project probably works on `localhost`, which means **only your laptop** can see it. Deployment is the process of putting it on the internet so anyone, anywhere can type a web address and use it.

By the end of this lesson you should be able to:

1. Explain what happens between typing `prc.opencodingsociety.com` and seeing a page.
2. Define domain, subdomain, DNS, public IP, private IP, port, HTTP/HTTPS and certificate.
3. Change your team's backend to use its **own port**.
4. Run your backend on the server with Docker.
5. Connect it to the internet with nginx and HTTPS.
6. Point your frontend at your deployed backend.
7. Fix the most common deployment errors.

---

## Part 1: The big idea (an analogy)

Think of the internet like a city.

| Real world | Internet |
|---|---|
| A business name ("Poway Recovery Center") | **Domain / subdomain** (`prc.opencodingsociety.com`) |
| The phone book that turns names into addresses | **DNS** (AWS Route 53) |
| The street address of a building | **Public IP** (`3.129.109.200`) |
| Apartment numbers inside the building | **Ports** (`8450`, `8451`, ...) |
| The receptionist in the lobby | **nginx** |
| Each apartment's tenant | Each team's **Docker container** running Flask |
| A sealed, tamper-proof envelope | **HTTPS** |
| The ID badge that proves who you are | **Certificate** |

All the teams' backends live in **one building** (one AWS EC2 server). The receptionist (nginx) reads the name on each visitor's request and sends them to the right apartment (port).

---

## Part 2: Vocabulary

| Term | What it means for us |
|---|---|
| **AWS** | Amazon's cloud. Holds everything below |
| **EC2** | The actual server (a computer in AWS) that runs all the backends |
| **Domain** | `opencodingsociety.com`. The main root web address we own for the organization |
| **Subdomain** | A specific slice of the main domain given to one team (e.g. `prc.opencodingsociety.com` = Poway Recovery Center frontend, `mgb-api.opencodingsociety.com` = MGB's backend) |
| **DNS** | Domain Name System. The internet's phone book. For us it's **AWS Route 53**, where CNAME and A records point subdomains to GitHub Pages or the AWS server |
| **Hosted zone** | The folder of records for one domain (`opencodingsociety.com`) |
| **Record** | One entry in the phone book |
| **CNAME record** | Name → another name. Used for **frontends** (`prc` → `poway-recovery-center.github.io`) |
| **A record** | Name → IP address. Used for **backends** (`prc-api` → `3.129.109.200`) |
| **Public IP** | `3.129.109.200`. The exact public address of our EC2 server. Anyone on the internet can reach it |
| **Private IP / Port** | Inside the server, Docker puts each team's backend in its own private box (e.g. port 8450 for PRC, 8451 for MGB). The internet can't reach 8450 directly; nginx answers the public door and passes traffic privately to that port |
| **Docker** | Packages your backend and everything it needs into a box (container) so it runs the same on any computer |
| **nginx** | The receptionist. Listens on the public doors (80 and 443) and forwards `name-api.opencodingsociety.com` → the right port |
| **HTTP vs HTTPS** | HTTP = normal, unencrypted. HTTPS = encrypted and secure. GitHub Pages forces frontends to use HTTPS, so backends **must** use HTTPS too or the browser blocks the connection |
| **Certificate** | The digital ID made when you run `sudo certbot --nginx`. It's what gives `prc-api.opencodingsociety.com` its lock icon |
| **Certbot** | The tool that creates the certificate and updates nginx for HTTPS |
| **Cockpit** | A website (`cockpit.stu.opencodingsociety.com`) that gives you a terminal on the server |
| **Frontend** | The website people see (GitHub Pages) |
| **Backend** | The Flask API that stores and serves data (runs on the server) |

---

## Part 3: The journey of a request

Here is everything that happens when someone uses your site.

```
FRONTEND
  Browser types  prc.opencodingsociety.com
    → Route 53 (CNAME)  says "that's poway-recovery-center.github.io"
    → GitHub Pages sends back the HTML/CSS/JS

BACKEND (when the page calls the API)
  JavaScript fetches  https://prc-api.opencodingsociety.com/api/...
    → Route 53 (A record)  says "that's 3.129.109.200"
    → EC2 server, public door 443 (HTTPS)
    → nginx reads the name "prc-api..." and looks up its config
    → proxy_pass to localhost:8450   (private, inside the server)
    → Docker container → Flask → response goes all the way back
```

Key point: **frontend and backend are two separate deployments** with two separate subdomains. The frontend lives on GitHub; the backend lives on AWS.

### Naming pattern

| Piece | Pattern | PRC example |
|---|---|---|
| Frontend | `teamname.opencodingsociety.com` | `prc.opencodingsociety.com` |
| Backend | `teamname-api.opencodingsociety.com` | `prc-api.opencodingsociety.com` |
| Port | a free `8xxx` number | `8450` |

---

## Part 4: DNS records (done by the admin)

Only the admin can make Route 53 records, but you should understand what gets made for your team. Two records per team:

| | Frontend record | Backend record |
|---|---|---|
| Record name | `teamname` | `teamname-api` |
| Type | **CNAME** | **A** |
| Value | `yourgithuborg.github.io` | `3.129.109.200` |
| TTL | 300 | 300 |

**Why CNAME for the frontend?** GitHub Pages' IP addresses can change, so we point at GitHub's *name* instead.
**Why A for the backend?** Our server has one fixed public IP, so we point straight at it.

You can check that your records work from any terminal:

```
nslookup teamname.opencodingsociety.com
nslookup teamname-api.opencodingsociety.com   # should show 3.129.109.200
```

New records can take a few minutes to show up (that's what TTL controls).

---

## Part 5: Ports, and why every team needs its own

One server, many teams. If two backends try to use the same port, the second one fails with **"port is already allocated"**. So every team picks its own.

### How to pick a free port
1. In the Cockpit terminal, run `docker ps`
2. Look at the **PORTS** column (the number before `->`)
3. Pick any `8xxx` number **not** in that list

### Ports to avoid
- **8587**: the starter-code default, so everyone has it
- **8500**: already taken for sockets
- Image names **`flask_open` / `socket_open`** are also taken, so rename them (`flask_teamname`)

---

## Part 6: Prepare your backend repo (on your laptop)

Change the starter port to your new port in two files.

**`docker-compose.yml`**
```
ports:
  - "8587:8587"        →   - "NEWPORT:NEWPORT"
image: flask_open      →   image: flask_teamname
```
If you have a socket service:
```
  - "8500:8500"        →   - "FREEPORT:8500"     # only the LEFT number changes
```
> The left number is the port on the **server**; the right number is the port **inside the container**. Only the left side has to be unique across teams.

**`Dockerfile`**
```
--bind=0.0.0.0:8587    →   --bind=0.0.0.0:NEWPORT
EXPOSE 8587            →   EXPOSE NEWPORT
```

**Test it locally before touching the server:**
```
docker-compose up
```
Then open `http://localhost:NEWPORT`. If it doesn't work on your laptop, it won't work on the server. Commit and push.

---

## Part 7: Run your backend on the server

Log into Cockpit (`cockpit.stu.opencodingsociety.com`) and open the terminal.

### Look around first (these commands are safe and read-only)

| Command | What it does |
|---|---|
| `ls` | List folders (each one is a team's backend) |
| `cd foldername` | Go into a folder |
| `cat filename` | Show a file |
| `docker ps` | Show running backends and their ports |

### Clone and start
```
cd ~
git clone https://github.com/ORG/REPO.git teamname_backend
cd teamname_backend
nano .env                      # add your settings; save with Ctrl+X → Y → Enter
docker-compose up -d --build   # build and start in the background (-d)
curl localhost:NEWPORT         # test it from inside the server
```

If `curl` returns HTML or JSON, your backend is running privately. It is **not** on the internet yet. That's nginx's job.

### Updating later
```
cd ~/teamname_backend
git pull
docker-compose up -d --build
```

---

## Part 8: nginx, the receptionist

Create a config file that tells nginx: "when someone asks for `teamname-api.opencodingsociety.com`, send them to `localhost:NEWPORT`."

```
sudo nano /etc/nginx/sites-available/teamname-api
```

Paste this and change the **two marked lines**:
```
server {
    listen 80;
    listen [::]:80;
    server_name teamname-api.opencodingsociety.com;   # ← change
    location / {
        proxy_pass http://localhost:NEWPORT;          # ← change
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Turn it on, **test**, then reload:
```
sudo ln -s /etc/nginx/sites-available/teamname-api /etc/nginx/sites-enabled
sudo nginx -t                  # MUST say "syntax is ok" and "test is successful"
sudo systemctl reload nginx    # ONLY if the test passed
```

> ⚠️ nginx serves **every team**. A broken config + reload can take down everyone's site. Always run `sudo nginx -t` first.

Now `http://teamname-api.opencodingsociety.com` should work, but only over plain HTTP.

---

## Part 9: HTTPS and the certificate

Your frontend is on GitHub Pages, which is **always HTTPS**. Browsers refuse to let an HTTPS page call an HTTP backend (a "mixed content" error). So the backend needs a certificate.

```
sudo certbot --nginx
```

Certbot lists every domain on the server. **Pick only your team's number.** It will:
1. Prove to Let's Encrypt that you control `teamname-api.opencodingsociety.com` (this only works if the DNS A record already points here)
2. Create the certificate
3. Edit your nginx file to listen on 443 and redirect HTTP → HTTPS

Visit `https://teamname-api.opencodingsociety.com` and look for the lock icon. 🔒

---

## Part 10: Connect the frontend

1. GitHub repo → **Settings → Pages**
   - Source: **GitHub Actions**
   - Custom domain: `teamname.opencodingsociety.com`
2. In `_config.yml`, if the site looks broken (no styles, broken links):
   ```
   baseurl: "/portfolio"   →   baseurl: ""
   ```
   (On a custom domain, the site lives at the root, not under `/portfolio`.)
3. In your frontend code, change the backend URL:
   ```
   http://localhost:8587   →   https://teamname-api.opencodingsociety.com
   ```

Push, wait for the Action to finish, and visit `https://teamname.opencodingsociety.com`.

---

## Part 11: Worked example (PRC)

| Step | PRC value |
|---|---|
| Frontend record | `prc` CNAME → `poway-recovery-center.github.io` |
| Backend record | `prc-api` A → `3.129.109.200` |
| Port | `8450` |
| Server folder | `~/prc_backend` |
| nginx file | `/etc/nginx/sites-available/prc-api` with `proxy_pass http://localhost:8450;` |
| Certificate | `sudo certbot --nginx` → picked only `prc-api.opencodingsociety.com` |
| Frontend URL | `https://prc.opencodingsociety.com` |
| Backend URL in frontend code | `https://prc-api.opencodingsociety.com` |

---

## Part 12: Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `nslookup` finds nothing | Record missing or still spreading | Wait a few minutes; ask admin to check Route 53 |
| `port is already allocated` | Another team uses your port | `docker ps`, pick a free port, update both files |
| `curl localhost:NEWPORT` fails | Container not running or crashed | `docker ps`; `docker-compose logs`; check `.env` |
| **502 Bad Gateway** | nginx works but nothing is on that port | Container down, or `proxy_pass` port is wrong |
| `nginx -t` fails | Typo in your config (missing `;` or `}`) | Fix the line it names. **Don't reload** |
| Certbot fails | DNS doesn't point to the server yet | Check the A record with `nslookup`, then retry |
| Browser: "mixed content" / blocked | Frontend calls `http://` backend | Run Certbot and use `https://` in your code |
| Browser: CORS error | Backend doesn't allow your frontend's domain | Add `https://teamname.opencodingsociety.com` to your Flask CORS allowed origins |
| Site has no styles / broken links | Wrong `baseurl` | Set `baseurl: ""` in `_config.yml` |
| Old code still showing | Server didn't rebuild | `git pull` then `docker-compose up -d --build` |

---

## Part 13: Golden rules

1. Every team gets its **own port**. Check `docker ps` before choosing.
2. **Always `sudo nginx -t` before reloading nginx.**
3. In Route 53: **only create**, never edit or delete other teams' records.
4. In Certbot: **only pick your own domain**.
5. Only touch **your own** folder and **your own** nginx file on the server.
6. Test locally first. If it fails on your laptop, it will fail on the server.

---

## Check your understanding

<details>
<summary>1. What's the difference between a domain and a subdomain?</summary>
<p>The domain is the root address we own (<code>opencodingsociety.com</code>). A subdomain adds a prefix to give one team its own slice (<code>prc.opencodingsociety.com</code>).</p>
</details>

<details>
<summary>2. Why is the frontend a CNAME but the backend an A record?</summary>
<p>The frontend points at another <em>name</em> (GitHub Pages, whose IPs can change). The backend points straight at our server's fixed public IP, <code>3.129.109.200</code>.</p>
</details>

<details>
<summary>3. Can someone on the internet visit <code>3.129.109.200:8450</code> directly? Why does that matter?</summary>
<p>No. 8450 is private inside the server. The public only reaches nginx on ports 80/443, and nginx passes the request privately to 8450. This is why you need an nginx config.</p>
</details>

<details>
<summary>4. In <code>"8452:8500"</code>, which number must be unique and why?</summary>
<p>The left one (8452). It's the port on the shared server. The right one is inside your own container, so it can be the same as other teams'.</p>
</details>

<details>
<summary>5. Your frontend loads but every API call fails with a "mixed content" error. What's wrong?</summary>
<p>The frontend is HTTPS (GitHub Pages) but it's calling an <code>http://</code> backend. Run <code>sudo certbot --nginx</code> for your API domain and change the URL in your code to <code>https://</code>.</p>
</details>

<details>
<summary>6. You get a 502 Bad Gateway. Is the problem DNS, nginx, or Docker?</summary>
<p>DNS and nginx are working (you reached nginx). nginx can't reach your app, so the container is down or <code>proxy_pass</code> has the wrong port. Check <code>docker ps</code>.</p>
</details>

<details>
<summary>7. Why must you run <code>sudo nginx -t</code> before reloading?</summary>
<p>nginx is shared by every team. Reloading a broken config can take every team's backend offline.</p>
</details>

---

## Your deployment checklist

- [ ] Picked a free port (checked `docker ps`)
- [ ] Updated `docker-compose.yml` (port + image names)
- [ ] Updated `Dockerfile` (`--bind` + `EXPOSE`)
- [ ] `docker-compose up` works on my laptop
- [ ] Admin made my `teamname` and `teamname-api` records
- [ ] Cloned repo on the server, added `.env`, ran `docker-compose up -d --build`
- [ ] `curl localhost:NEWPORT` works on the server
- [ ] Made nginx file, `nginx -t` passed, reloaded
- [ ] Ran `sudo certbot --nginx` for **only** my domain
- [ ] `https://teamname-api.opencodingsociety.com` shows a lock
- [ ] GitHub Pages custom domain set, `baseurl` fixed
- [ ] Frontend code uses `https://teamname-api.opencodingsociety.com`
- [ ] Full site works at `https://teamname.opencodingsociety.com` 🎉
