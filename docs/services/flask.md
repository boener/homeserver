# 🐍 Flask Service

## 🧠 Overview

Flask is the **public-facing experimentation layer** of the home server.

It is used to host small internet-facing projects, toys, jokes, and interactive pages.

This is not a traditional production web app. It is better understood as:

> a lightweight creative playground for Python-powered web experiments

Examples of its role include:
- joke pages
- small browser games
- novelty web interactions
- local-AI-connected experiments

---

## 🎯 Purpose in This System

Flask exists to make it easy to build and publish small web ideas without a heavy framework.

This fits the project well because it allows:
- fast iteration
- simple Python code
- easy routing behind Caddy
- experimentation without much infrastructure overhead

In practice, Flask is the server’s **public toybox**.

---

## 🌍 Exposure Model

### Public Access
Flask is intentionally internet-facing through:
- `https://www.ianboen.com` and `https://ianboen.com`
- `https://boener.duckdns.org` (retiring)

### Routing
Caddy receives incoming requests and reverse proxies them to Flask on:
- `127.0.0.1:5000`

This means Flask itself stays internal while Caddy handles:
- TLS
- public hostname access
- edge routing

---

## 🏗️ Architecture Role

Browser
→ Caddy
→ Flask
→ response

Flask is the **application logic layer**.

Caddy is the **web entry and HTTPS layer**.

This separation is important because it keeps responsibilities clean:
- Caddy handles networking + certificates
- Flask handles behavior + content

---

## 🧩 Current Use Pattern

Flask is used for small custom web projects that may change often.

Past / current examples include:
- novelty homepage content
- the “Swirling Poop Emoji” page
- a comment-box experiment that sent user comments to a local model running on the Windows machine and generated poop-themed responses
- heart-splitter emoji game
- The current Cowrie log display

The exact app content may change, but the structural role remains the same:

> Flask is the platform for public-facing experiments and amusements.

---

## 🧠 Why Flask Is a Good Fit Here

Flask works well for this system because it is:
- lightweight
- easy to understand
- Python-based
- flexible enough for weird ideas

That matters because the goal is not enterprise web development.
The goal is to quickly make strange or funny web things and put them online.

---

## ⚙️ Runtime Model

### Port
- `5000`

### Process Management
- managed via `systemd`

### Reverse Proxy
- handled by Caddy

### Public URL
- `https://www.ianboen.com` and `https://ianboen.com`
- `https://boener.duckdns.org`

---

## 🔌 Relationship to Other Machines

One interesting part of this setup is that Flask can act as the public-facing piece of a larger multi-machine experiment.

Example pattern:
- Flask page runs on the home server
- a user interacts through the browser
- Flask sends data to another machine or local service
- that other system performs processing
- Flask returns the result

This is important because it means the home server can serve as a **web front-end for other systems**, not just a standalone host.

---

## ⚠️ Operational Notes

### Public Means Public
Flask is intentionally exposed to the internet.

That means changes here should be made with more awareness of:
- input handling
- logging
- accidental breakage
- security mistakes

### Fast Iteration, Low Ceremony
This service is expected to change often.
That is normal.
The documentation should preserve the *role and architecture*, not every joke or page variant.

---

## 🧪 Troubleshooting Model

If the Flask site is broken, the problem is usually one of these:

1. **Caddy issue**
   - domain/routing/TLS problem
2. **Flask issue**
   - app crashed or code error
3. **systemd issue**
   - service not running
4. **dependency issue**
   - Python environment/module problem
5. **cross-machine dependency issue**
   - if Flask depends on another host or model service

This is the right mental model for debugging.

---

## 🎯 Summary

Flask is the home server’s **public creative application layer**.

It is used to publish fun, weird, lightweight Python web projects to the internet through Caddy.

This makes the server not just a storage or media box, but a place to quickly build and deploy interactive ideas.
