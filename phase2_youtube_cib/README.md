# YouTube Fake Engagement & Bot Network Detection

**author** — Shamili Kolli, India  
---

## What this is

I built a system that catches fake engagement on YouTube - bot farms that dump likes and views on videos to make them look popular. 

The project uses **real YouTube data** (1.68 million rows tracking 10,000 videos over time) to find two things:
1. **Individual videos** with suspicious engagement patterns
2. **Groups of videos** being manipulated together by the same bot network

I found **15 bot networks** in my sample, including one group of **85 videos** that were clearly coordinated. More than half the suspicious activity was organized, not random.

---

## Why I built this

I started with a simpler project using fake data (Faker library) where I wrote rules to catch bots. Got 100% accuracy - which felt great until I realized the problem: **I was checking my own homework.** The fake data had patterns I put there, and my rules just matched them. Useless in the real world.

So I switched to real YouTube data where I don't know what's fake. Built an **unsupervised system** that finds weird patterns without needing labeled examples. Since fraud patterns change constantly, we can't rely on old examples and hence have to find the patterns newly.

---

## How it works (simple version)

### Step 1: Feature Engineering — "What does fake look like?"

Real engagement grows gradually. Bots spike hard and disappear. I engineered features that capture this:

| Feature | What it means |
|---------|-------------|
| `like_view_ratio` | Likes compared to views - 50% is weird, 0.001% is normal |
| `like_spike_ratio` | How big was the spike compared to the video's normal? 100x = bot, 1.2x = fine |
| `sustainability` | After the spike, does engagement keep going or die? Bots die, a legit viral content sustains |

**The sustainability feature is the key one.** I designed it from scratch after thinking about how bot farms actually behave - they disconnect after dumping likes. Viral videos keep growing showing a sustained growth in likes / views, because real people share them.

### Step 2: Anomaly Detection — "Find the weird ones"

Used **Isolation Forest** to flag the 8% most unusual videos. No labels needed - it learns what's "normal" and flags deviations.

**Why 8%?** I tried 5% first but the coordination signals were too weak. At 8%, I found the same bot networks but with clearer overlap and shared timing. Validated by checking the flagged videos actually had bot signatures (low sustainability, high spikes, same peak hours).

### Step 3: Find Coordinated Networks — "Who's working together?"

Real fraud (bot network) isn't operated alone . Bot farms hit multiple videos at once. I built three signals:

1. **Peak hour clustering** — 85 videos all spiking at Hour 12? Not coincidence.
2. **Temporal co-spiking** — Videos with max likes in the same hour window
3. **DBSCAN clustering** — Group suspicious videos by engagement fingerprint

**Important fix:** First I tried clustering ALL videos. Result: 9,500 normal videos in one giant blob. Not providing usefull information. Then I realized — **only cluster their suspect list.** Did that instead. Found 15 clean networks.

### Step 4: Tiered Enforcement — "What do we do about it?"

Since we can't manually review everything. Built a scoring system that routes videos to different actions:

| Score | Tier | Action |
|-------|------|--------|
| 6+ | Tier 1 | Auto-remove fake engagement |
| 5-6 | Tier 2 | Human investigator looks at the network |
| 4-5 | Tier 3 | Watchlist - check again next cycle |
| 2-4 | Tier 4 | Probably false positive, re-check |
| 0-1 | Clean | Leave it alone |

---

## Results

| What | Number |
|------|--------|
| Videos analyzed | 9,999 (10K sample from 270K total) |
| Flagged as suspicious | 800 (8%) |
| Bot networks found | 15 |
| Biggest network | 85 videos |
| Videos in networks | 282 (56% of suspicious) |
| Acting alone | 218 |
| Auto-action (Tier 1) | 261 videos |
| Human review (Tier 2) | 21 videos |

**Key insight:** Most fake engagement is organized. If you only look at individual videos, you miss the operation.

---

## The data

- **Source:** Real YouTube video statistics (anonymized)
- **Full dataset:** 45.5 million rows, 270K videos, 3.5GB
- **What I used:** 1.68 million rows (10K video sample with complete history)
- **Why sample:** Iterative development on a laptop. Pipeline designed to scale to full dataset.

---

## What this shows for the YouTube job

| They want | I did |
|-----------|-------|
| Fraud investigations | Built investigation pipeline, found 15 bot networks |
| Find product weaknesses | Discovered bot farms exploit time windows |
| Run anti-abuse experiments | Tested contamination rates, validated feature impact |
| Design metrics & experiments | Built tiered system with measurable thresholds |
| Work with engineers | Designed modular pipeline, not just a notebook |

---

## What's missing (honest)

1. **No ground truth** — I don't know for sure which videos are actually fake. Need to inject synthetic bots and measure precision.
2. **Didn't use comment text** — The data has comment counts but not content. NLP would help catch spam.
3. **No SHAP explainability** — Need to explain WHY each video was flagged, for appeals and stakeholder trust.
4. **Batch only** — Real YouTube needs real-time streaming. This is offline analysis.
5. **One platform** — Bot farms work across YouTube, Instagram, TikTok together. Cross-platform would catch more.

---

## Tech used

Python, Pandas, Scikit-learn (Isolation Forest, DBSCAN), Matplotlib

---

## Contact

- **Email:** shamilikolli@gmail.com
- **Phone:** +91-9908762112
- **Location:** Hyderabad, India
- **LinkedIn:** [your-link]

Looking for: **Engineering Analyst, Trust & Safety, YouTube — Hyderabad**