---
layout: post
title: "CPTS Detailed Guide"
date: 2025-11-22 08:00 +0530
categories: [CPTS Stuff]
tags: [CPTS]
---

# Certified Penetration Testing Specialist

![BANNER](https://www.hackthebox.com/storage/blog/GRXAcEp8ZhICxjMTvCELE81LDEANJIj8.jpg)

First, I want to tell you **why you should read me?** Like, *how am I unique?*  
Meanwhile, CPTS is done by many people, and you may find hundreds of articles and blogs on this topic.

Well… the answer is:

**“Not because I did CPTS, but because I did CPTS with 14/14 flags (100 points), because I did CPTS at the age of 18, and because I completed the entire Penetration Tester Job Role Path on a 10-year-old laptop with 4 GB RAM, no SSD, and an i3 5th gen processor!”**

I know the above words may smell like overconfidence or show-off, but it’s important. Because if you’re reading this, then you should know who the person behind this writing actually is.

---

## CPTS Path and Exam

Here, I mainly explain **how I did the path** and in the end, I also clear some common questions.

---

## April — June

This was the very starting period of my unwanted / unintentional vacations.  
Unintentional because during this time I gave my 12th-grade exams, and now I was free for a flat 3 months.

I purchased the **Penetration Tester Job Role Path** for $8 using my student email and started learning the path. I completed the whole path by studying around **8–10 hours per day**.  
I was also making notes during this timeline!

---

## July

In this period, I became a bit comfortable and started exploring the cyber world — things related to CPTS like:  
“how to prepare”, “how to pass”, and all that.

Also, I started attending college because from July my college had started.

---

## August — September

Again I shifted my gears and started doing the whole CPTS path skill assessments once again.  
Moreover, I also did some Hack The Box CTFs, watched ippsec playlists, and tried to solve the same boxes on my own.

I met some online friends who guided me very well during this period.

---

## October

On **October 18**, I scheduled the CPTS exam (right between Diwali).  
And the rest is history!

![CPTS certificate](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*UO3fqqWGjKVsP2aZKHfsrg.jpeg)

I got my result in **less than 24 hours**!  
Very surprising, but the results were good for me!

---

# FAQs

### 1. What is the level of the exam and its difficulty?

The exam machines are of **medium level** compared to the HTB boxes.  
But the combination of **multiple vulnerability chaining + rabbit holes + time constraints + commercial-grade reporting** makes it HARD.

---

### 2. Where to study?

The **Penetration Tester Job Role Path** is **90% enough** for passing the exam.  
For the remaining 10%, you must try doing:

1. **Pro Labs (first priority)**  
2. **HTB Boxes (second priority)**

I didn’t do Pro Labs myself, but they simulate the real exam environment. That’s why everyone recommends them.

---

### 3. How much time does reporting take?

It depends on you!  
I made a **very detailed report (185 pages)** and it took me just **2 days**.

---

# My Mistakes (Must Avoid)

![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*xuTy712J7uwm7T9zJNx5Zg.jpeg)

### 1. Not taking double pivoting seriously enough  
I learned how to use Ligolo-ng and double pivoting **one day before my exam**.  
Very risky, because I was hoping pivoting wouldn’t be necessary — and I was wrong.

### 2. Doing HTB boxes instead of Pro Labs  
I focused on boxes, which is normal for everyone.  
But in CPTS, things don’t work like that.  

Boxes are nearly useless when compared to the **actual exam experience and exam-like feel**.  
They may teach you 2–3 new attack methods, but the *core* of CPTS is Pro Labs.

### 3. Not trying attacks on “dumb things”  
Maybe the worst mistake I made at the start of the exam.  
I won’t give full details, but keep in mind:

**“This is an exam designed to test your skills. Keep logic aside and try every possible thing on every single component. It may seem dumb at first, but you never know where something is hidden.”**

### 4. Relying on one tool  
The exam is designed to test your mental toughness.  
Sometimes legacy or highly-used tools may not work or may give weird errors.  
When this happens:

- Do more research  
- Try the tool again  
- Try different arguments  
- Switch tools  
- Use alternative
- Use Lab's *pwnbox*
- End if all else fails, try again after resetting the exam lab — sometimes it’s just a *glitch*.

### 5. Not taking small naps  
For the first 4 days, I worked almost **15–16 hours non-stop**.  
When you're stuck and not getting anything, simply shut down your PC and take a **20–40 min nap** (use an alarm).

The mind works in mysterious ways. You may get hints in your dreams!  
I discovered this accidentally — I slept for 30 minutes, woke up, and got my answer in just 10 minutes.

---

# Reporting Tips

I got my results in **less than 24 hours**. Maybe they liked my report very much 😅 — my friends and mentors were shocked. *Not because I passed, but because of the "fast result"!*

## Personally Implemented Tips

- Highlight every screenshot, wherever and whenever possible.  
- Many people advise **not to use too many images** (which is a valid point), but never hesitate if an image helps you explain something. If images make your report clearer, **use them** — you can always compress later.  
- Don't use [I Love PDF](https://www.ilovepdf.com/) to compress your images, because it compresses the PDF but doesn't let you fine-tune quality and size.

For example, your report PDF might be 40 MB, and when you compress using [I Love PDF](https://www.ilovepdf.com/) it may shrink drastically to 10–12 MB. But the limit is 20 MB... What if you want to keep better quality and get the PDF to ~19 MB to stay within the limit while preserving clarity?

### Solution

Use [PDF24-Tool](https://tools.pdf24.org/en). It's also available on the Microsoft Store.

![PDF24](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*XrMJ3dirbY_XtPm_EamVIw.png)

It might not be as popular as [I Love PDF](https://www.ilovepdf.com/), but it's best for our purpose!

* Open the tool.

![openTOOL](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*YHdrsTqgntLEfXSjhLuf-A.png)

* Upload your PDF and adjust DPI and image quality accordingly. Here DPI means **how clear your PDF looks while zooming** (try to keep it reasonably high). Image quality means **how clear your screenshots appear in the report** (again, try to keep it high).

![Adjustments](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*tKHj6FVfPciqrBPNEc-bUA.png)

* Think like **you are teaching every step to a non-technical person** and make the report accordingly.  
* You can also include your own Python script in the report, but don't bluff — they want a **commercial-grade report**, not your personal preferred reporting style.

# Final Words

At the end, everything depends on your hard work… and also a bit on luck.  
Because maybe you are the greatest hacker on the planet — but not the luckiest one.

> **“Even world-class and national swimmers can drown — greatness isn’t immunity. Swim brave, stay humble.”**

