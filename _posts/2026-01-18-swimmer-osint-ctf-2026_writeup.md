---
title: "SWIMMER OSINT CTF 2026: Writeup"
date: 2026-01-20 13:00:00 +0530
categories: [CTF, Others]
tags: [ctf, writeups, cybersecurity, osint]
excerpt: "A writeup for SWIMMER OSINT CTF 2026 challenges."
---

## TL;DR

* Participated in **SWIMMER OSINT CTF 2026** (hosted by DIVER OSINT CTF)
* Team: **0$1N7_K1dd13$**
* Final rank: **200 / 688 teams**
* Solved: **23 / 29 challenges**
* Personally solved: **8 challenges**
* Focus areas: aviation OSINT, geo-location, archival research, Japanese media OSINT
* Core tools: Google, Google Lens, OpenStreetMap, YouTube, UN Digital Library, Wikipedia

Plus Ultra.

---

## Event Info

Swimmer OSINT CTF 2026 is a 12-hour CTF with all challenges being OSINT. Max team size is 6 and the event is held online.

* **CTFtime:** [https://ctftime.org/event/2986](https://ctftime.org/event/2986)
* **Platform:** [https://swimmer.diverctf.org/](https://swimmer.diverctf.org/)

I played with teammates from **\$cr1pt_K1dd13\$**.
Due to the 6-member limit, we split into two teams.
I competed in **0$1N7_K1dd13$** with *GregorSec, th3mujd11, Scorpius,* and *Codru*.

* Rank: **200 / 688**
* Solved: **23 / 29**
* Personally solved: **8 challenges**

<div style="display:flex; gap:5px">
    <img src="/assets/img/posts/swimmerCTF2026/swimmer.png" alt="SWIMMER CTF banner" style="width: 100%; height: auto;">
    <img src="/assets/img/posts/swimmerCTF2026/team.png" alt="Team photo" width="100%">
</div>

<div style="display:flex; gap:5px">
  <img src="/assets/img/posts/swimmerCTF2026/certificate.png" alt="Certificate" width="80%">
  <img src="/assets/img/posts/swimmerCTF2026/scoreboard.png" alt="Scoreboard" width="100%">
</div>

---

## Challenge Index

| Category       | Challenge        | Core Technique                 | Tools Used                         |
|----------------|------------------|-------------|--------------------------------|
| research_2025  | cx               | Aviation OSINT                 | Google, News Articles              |
| research_2025  | pilot            | Video Analysis                 | YouTube, LinkedIn                  |
| research_2025  | flag_on_the_don  | Event OSINT + Geo              | Google, Gemini, OpenStreetMap      |
| research_2025  | obsolete         | Archival Research              | UN Digital Library, Google         |
| research_2025  | truck            | Geo + Video OSINT              | Google Maps, 11foot8.com           |
| research_2025  | debris           | Geo-location                   | YouTube, Google Maps               |
| tgt_rain       | rain_04_source2       | Reverse Image Search           | Google Lens, Wikipedia             |
| tgt_rain       | rain_05_date        | Event Correlation              | LinkedIn, Image Analysis           |

---

## research_2025

### **cx**

**Tools:** Google, News Articles

**Description**
In Spring 2025, a special flight commemorated the 100th anniversary of an airport that once existed in Hong Kong.
Find the flight number.

**Flag:** `SWIMMER{CX8100}`

**Write-up**

I searched for combinations of:

> “special flight Hong Kong 100th anniversary 2025”

This led me to the following article:
[https://campaignbriefasia.com/2025/04/11/flight-cx8100-takes-off-cathay-soars-once-more-over-kai-tak-in-tribute-to-legendary-flight-path/](https://campaignbriefasia.com/2025/04/11/flight-cx8100-takes-off-cathay-soars-once-more-over-kai-tak-in-tribute-to-legendary-flight-path/)

The article clearly states that **Cathay Pacific operated flight CX8100** to commemorate the Kai Tak Airport flight path anniversary.

---

### **pilot**

**Tools:** YouTube, LinkedIn

**Description**
Identify the person sitting in the indicated seat on flight CX8100.

<img src="/assets/img/posts/swimmerCTF2026/pilot.jpg" alt="Cockpit seat photo">

**Flag:** `SWIMMER{Adrian Scott}`

**Write-up**

I reviewed related articles and videos about flight CX8100:

- [LinkedIn article describing the commemorative flight](https://www.linkedin.com/pulse/100-years-hong-kong-kai-tak-airport-significance-history-mitali-pant-erbxc)
- [YouTube footage showing cockpit scenes](https://www.youtube.com/watch?v=Fjs9AfnyuLg)

The crew consisted of:
Geoffrey Lui, Adrian Scott, Boris Wong, Darren Palmer.

The YouTube footage clearly shows **Adrian Scott** seated on the right-hand side, matching the seat diagram in the challenge image.

---

### **flag_on_the_don**

**Tools:** Google, Gemini, OpenStreetMap

**Description**
On August 28th 2025, an event using *Taiko no Tatsujin* was held in Gunma Prefecture.
Find the OpenStreetMap way ID of the venue.

**Flag:** `SWIMMER{628293186}`

**Write-up**

I found a Japanese PDF listing city events:
[https://www.city.shibukawa.lg.jp/manage/contents/upload/68870b94eeb0d.pdf](https://www.city.shibukawa.lg.jp/manage/contents/upload/68870b94eeb0d.pdf)

I used Gemini to extract event details, confirming:

* Event date: 28 August 2025
* Venue: Shibukawa Civic Hall, Gunma

I located the venue in OpenStreetMap, enabled **Map Data**, clicked the building outline, and obtained the way ID: **628293186**.

<img src="/assets/img/posts/swimmerCTF2026/don.png" alt="Event flyer">

---

### **obsolete**

**Tools:** Google, UN Digital Library

**Description**
Find which countries abstained in UNGA Resolution 50/52 (1995).

**Flag:**
`SWIMMER{CUBA_DEMOCRATIC PEOPLE'S REPUBLIC OF KOREA_LIBYAN ARAB JAMAHIRIYA}`

**Write-up**

I located UNGA Resolution 50/52 in the UN Digital Library:
[https://digitallibrary.un.org/record/284118?ln=en](https://digitallibrary.un.org/record/284118?ln=en)

Countries without Y/N markers were abstentions:

* CUBA
* DEMOCRATIC PEOPLE'S REPUBLIC OF KOREA
* LIBYAN ARAB JAMAHIRIYA

---

### **truck**

**Tools:** Google Maps, 11foot8.com

**Description**
Identify the FQDN on a truck at a given Plus Code.

**Flag:** `SWIMMER{www.MiracleMoversUSA.com}`

**Write-up**

The Plus Code resolved to **The Can Opener Bridge, Durham**.

Searching:

> “The Can Opener Bridge June 21 2025”

led to:
[https://11foot8.com/moving-truck-runs-red-light-and-scrapes-the-11foot88-bridge/](https://11foot8.com/moving-truck-runs-red-light-and-scrapes-the-11foot88-bridge/)

The video clearly shows the truck branding:
**[www.MiracleMoversUSA.com](https://www.MiracleMoversUSA.com)**

<img src="/assets/img/posts/swimmerCTF2026/truck.png" alt="Truck with branding">

---

### **debris**

**Tools:** YouTube, Google Maps

**Description**
Find where a ton bag was initially dropped.

**Flag:** `35.7131787, 139.7296142`

**Write-up**

I found footage from Mezamashi Media:
[https://mezamashi.media/articles/-/232641](https://mezamashi.media/articles/-/232641)
[https://www.youtube.com/watch?v=xo_2KZ3n668](https://www.youtube.com/watch?v=xo_2KZ3n668)

The controller dialogue references:
**Gokokuji Road, Otowa**.

Using road curvature and nearby buildings visible in the footage, I matched the location in Google Maps and extracted the coordinates.

<img src="/assets/img/posts/swimmerCTF2026/debris.png" alt="Debris location screenshot">

---

### **rain_04_source2**

**Tools:** Google Lens, Wikipedia

<img src="/assets/img/posts/swimmerCTF2026/source2-1.png" alt="Archival image 1">

**Flag:** `SWIMMER{10.11501/967480}`

**Write-up**

Reverse image search identified the design as Watanabe Fukuzo’s entry for the National Diet Building competition.

Source:
[https://dl.ndl.go.jp/pid/967480/1/7](https://dl.ndl.go.jp/pid/967480/1/7)

The DOI shown in the archival metadata:
**10.11501/967480**

<img src="/assets/img/posts/swimmerCTF2026/source2-2.png" alt="Archival image 2">

---

### **rain_05_date**

**Tools:** Google, LinkedIn, Image Analysis

<img src="/assets/img/posts/swimmerCTF2026/date.jpg" alt="Event banner">

**Flag:** `SWIMMER{2025/12/07}`

**Write-up**

I identified the green banner as:

> *SUNTORY 10000 Freude Beethoven’s 9th*

Searching Google, found a helpful LinkedIn post:
[https://www.linkedin.com/posts/suntory-holdings-limited_suntory10000freude-givingbacktosociety-activity-7404981711249408000-KAj6](https://www.linkedin.com/posts/suntory-holdings-limited_suntory10000freude-givingbacktosociety-activity-7404981711249408000-KAj6)

This confirmed the event occurred on **7 December 2025**.

---

## Conclusion

This CTF heavily favored Japanese-language OSINT and media tracing.
It significantly improved my skills in:

* event correlation
* video-based geo-location
* archival document analysis
* multilingual OSINT

We didn’t place at the top, but I did learn a lot.

Plus Ultra.

---