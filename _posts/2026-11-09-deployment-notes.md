---
layout: post
categories:
- CSP
title: Deployment Notes (OCS / AP CSP)
description: My notes on deploying team frontends and backends for Open Coding Society, covering Route 53, Cockpit, Docker, nginx, and Certbot
permalink: /csp/deployment-notes
author: Anika
---

## Terms (what they mean)
| Term | Meaning |
|---|---|
| **AWS** | Amazon's cloud. Holds everything below |
| **IAM** | AWS user accounts / logins (mine: `anika2317`) |
| **EC2** | The actual servers (computers) in AWS |
| **Route 53** | Phone book. Makes web addresses (`name.opencodingsociety.com`) |
| **Hosted zone** | Folder of addresses for one domain (use `opencodingsociety.com`) |
| **Record** | One address in the phone book |
| **CNAME** | Address → another name (frontend → `org.github.io`) |
| **A record** | Address → IP number (backend → server) |
| **Cockpit** | Website to open the server's terminal |
| **Terminal** | Where you type commands on the server |
| **Docker** | Runs each team's backend in its own box (container) |
| **Port** | The backend's door number on the server (e.g. 8450). Each team needs its own |
| **nginx** | Receptionist. Sends `name-api.opencodingsociety.com` → the right port |
| **Certbot** | Adds HTTPS (the lock icon) |
| **Frontend** | The website (GitHub Pages) |
| **Backend** | The Flask API (runs on the server) |

## Two different logins
| | AWS console | Cockpit |
|---|---|---|
| Link | nighthawkcodingsociety.signin.aws.amazon.com/console | cockpit.stu.opencodingsociety.com |
| Login | `anika2317` | `ubuntu` |
| Used for | Route 53 records (**admin only = me**) | Server terminal (deploying backends) |

## Big picture
```
Frontend:  name.opencodingsociety.com     → Route 53 (CNAME) → GitHub Pages
Backend:   name-api.opencodingsociety.com → Route 53 (A) → server → nginx → port → Docker/Flask
```

## Important info
- **Server IP (every backend A record):** `3.129.109.200`
- **Example to copy:** PRC → `prc` (CNAME → `poway-recovery-center.github.io`), `prc-api` (A → `3.129.109.200`), port `8450`
- **Avoid port 8587** (starter-code default, everyone has it)
- **Avoid 8500** for sockets (taken)
- **Image names `flask_open` / `socket_open` are taken** → rename to `flask_teamname` etc.

## My job: Route 53 records
1. AWS console → search **Route 53** → **Hosted zones** → click **opencodingsociety.com**
2. Click **Create record**
3. **Frontend record**
   - Record name: `teamname`
   - Type: **CNAME**
   - Value: `githuborg.github.io`
   - TTL: `300`, Routing: Simple
4. Click **Create records** → green "successfully created" ✅
5. **Create record** again → **Backend record**
   - Record name: `teamname-api`
   - Type: **A**
   - Value: `3.129.109.200`
   - TTL: `300`, Routing: Simple
6. Click **Create records** → green ✅

**Rules:** only ever **Create**. Never edit or delete other teams' records. Leave name blank = WRONG (that's the main site).

## Looking around the server (safe, read-only)
| Command | What it does |
|---|---|
| `ls` | List folders (each = a team's backend) |
| `cd foldername` | Go into a folder |
| `cat filename` | Show a file |
| `docker ps` | Show running backends + their ports |
| `cat /etc/nginx/sites-available/pcr-api` | Show PRC's nginx file |

## Picking a port
1. Run `docker ps`
2. Look at **PORTS** column (number before `->`)
3. Pick any 8xxx number **not** in the list

## What teams do (after I make records)
**Their files (GitHub):**
- `docker-compose.yml`: `"8587:8587"` → `"NEWPORT:NEWPORT"`, rename images (`flask_open` → `flask_teamname`)
- If socket: `"8500:8500"` → `"FREEPORT:8500"` (only left number changes)
- `Dockerfile`: `--bind=0.0.0.0:8587` → `NEWPORT`, `EXPOSE 8587` → `EXPOSE NEWPORT`
- Test on laptop: `docker-compose up` → open `localhost:NEWPORT`

**Server (Cockpit terminal):**
```
cd ~
git clone https://github.com/ORG/REPO.git teamname_backend
cd teamname_backend
nano .env                      # their settings, save: Ctrl+X → Y → Enter
docker-compose up -d --build   # start it
curl localhost:NEWPORT         # test it
```

**nginx:**
```
sudo nano /etc/nginx/sites-available/teamname-api
```
Paste (change 2 lines):
```
server {
    listen 80;
    listen [::]:80;
    server_name teamname-api.opencodingsociety.com;
    location / {
        proxy_pass http://localhost:NEWPORT;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
```
sudo ln -s /etc/nginx/sites-available/teamname-api /etc/nginx/sites-enabled
sudo nginx -t                  # MUST say "ok" + "successful"
sudo systemctl reload nginx    # only if test passed
sudo certbot --nginx           # pick ONLY their number
```

**Frontend (GitHub):**
- Repo → **Settings → Pages** → source **GitHub Actions** → Custom domain `teamname.opencodingsociety.com`
- `_config.yml`: `baseurl: "/portfolio"` → `baseurl: ""` (if site looks broken)
- Change backend URL in their code → `https://teamname-api.opencodingsociety.com`

## Golden rules
1. Every team = **own port** (check `docker ps`)
2. **Always `sudo nginx -t` before reload**
3. In Route 53: **only Create**, never edit/delete others

## Done so far
| Team | Frontend | Backend | Port | Server | Frontend setup |
|---|---|---|---|---|---|
| PRC | ✅ prc | ✅ prc-api | 8450 | ✅ | ✅ |
| MGB (Rashi) | ✅ mgb → apcspt1.github.io | ✅ mgb-api → 3.129.109.200 | 8451 (socket 8452) | ⏳ | ⏳ |

## Ask Mr. Mortensen
- Do I deploy every team on Cockpit, or just make Route 53 records?
- Do teams get the Cockpit password?
- CS2027-Backend (new EC2 server): do I make it? (**don't touch until he says**)

## Story (capstone)
- OCS → give back to the community using technical skills
- Need capstone page + demo
- Next Gen OCS (2nd-year student in the pathway)
- Talk about deployment
