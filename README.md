# x-open-source-algorithm-skill

**Score and optimize X/Twitter posts using the actual open source algorithm.**

Built from [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) (January 2026 release), the Grok-based Phoenix recommendation system that powers X's "For You" feed.

> Every other X optimization skill is based on blog posts and guesswork. This one is built from the actual source code.

---

## What makes this different

Every other X optimization skill is based on blog posts and guesswork. This one is built from the actual source code.

In January 2026, X open-sourced their recommendation algorithm. The repo contains four components:

| Component | What it does |
|-----------|-------------|
| **Home Mixer** | Orchestrates the feed pipeline (Rust) |
| **Thunder** | In-memory store for in-network posts (Rust) |
| **Phoenix** | Grok-based transformer that scores every post (JAX/Python) |
| **Candidate Pipeline** | Reusable pipeline framework (Rust) |

This skill extracts the scoring logic from `home-mixer/scorers/` and `phoenix/` and turns it into a practical content optimization workflow.

---

## What it does

1. **Analyzes draft posts** against the 19 engagement signals Phoenix actually predicts
2. **Scores posts** on a weighted scale matching the algorithm's scoring formula
3. **Recommends format** (article vs thread vs single tweet) based on algorithm ranking
4. **Rewrites content** with specific signal rationale (e.g., "this change targets reply_score")
5. **Plans posting strategy** (timing, first-30-min velocity, reply approach)

---

## The Phoenix Scoring Model

Phoenix predicts 19 engagement probabilities for every candidate post. The weighted scorer combines them:

```
Final Score = sum(weight_i * P(action_i))
```

### Positive signals (higher = more reach)

| Signal | Source in code | What triggers it |
|--------|---------------|-----------------|
| `P(favorite)` | `ServerTweetFav` | Likes |
| `P(reply)` | `ServerTweetReply` | Replies |
| `P(repost)` | `ServerTweetRetweet` | Retweets |
| `P(quote)` | `ServerTweetQuote` | Quote tweets |
| `P(click)` | `ClientTweetClick` | Clicking the post |
| `P(profile_click)` | `ClientTweetClickProfile` | Clicking author profile |
| `P(video_quality_view)` | `ClientTweetVideoQualityView` | Watching video past threshold |
| `P(photo_expand)` | `ClientTweetPhotoExpand` | Expanding a photo |
| `P(share)` | `ClientTweetShare` | Sharing the post |
| `P(share_via_dm)` | `ClientTweetClickSendViaDirectMessage` | Sharing via DM |
| `P(share_via_copy_link)` | `ClientTweetShareViaCopyLink` | Copying the link |
| `P(dwell)` | `ClientTweetRecapDwelled` | Stopping to read |
| `P(dwell_time)` | `DwellTime` (continuous) | Seconds spent on post |
| `P(follow_author)` | `ClientTweetFollowAuthor` | Following the author |

### Negative signals (higher = penalized)

| Signal | Source in code | What triggers it |
|--------|---------------|-----------------|
| `P(not_interested)` | `ClientTweetNotInterestedIn` | "Not interested" button |
| `P(block_author)` | `ClientTweetBlockAuthor` | Blocking |
| `P(mute_author)` | `ClientTweetMuteAuthor` | Muting |
| `P(report)` | `ClientTweetReport` | Reporting |

### Additional scorers

| Scorer | What it does | Source file |
|--------|-------------|------------|
| **Author Diversity** | Penalizes repeated authors in one feed session via exponential decay | `author_diversity_scorer.rs` |
| **OON (Out-of-Network)** | Multiplies out-of-network scores by a factor < 1.0 | `oon_scorer.rs` |
| **Video Quality View** | Only applies VQV weight if video exceeds minimum duration | `weighted_scorer.rs` |

> **Note:** The exact weight values (FAVORITE_WEIGHT, REPLY_WEIGHT, etc.) are in a private `params` module excluded from the open source release. This skill combines the scoring structure from the source code with observed weights from empirical 2026 data.

---

## Observed engagement hierarchy (2026 data)

From analysis of 18.8M+ posts:

| Action | Observed weight | Notes |
|--------|----------------|-------|
| Reply with author response | +75 | Author replies to your reply |
| Likes | +30 | Primary distribution signal |
| Retweets | +20 | Amplification |
| Reply alone | +13.5 | Without author engagement |
| Replies | 150x likes | Conversation signals dominate |

### Premium multipliers

| Metric | Premium | Non-Premium |
|--------|---------|-------------|
| In-network visibility | 4x | 1x |
| Out-of-network visibility | 2x | 1x |
| Overall reach per post | 10x | 1x |

---

## Content format ranking

Based on algorithm behavior (best to worst):

1. X Articles (algorithm-boosted, $1M monthly prize)
2. Video thread (video + 3-5 tweets)
3. Video single post (5x text engagement)
4. Image/GIF thread (150% more interactions)
5. Text thread (3x single tweet, 7 tweets optimal)
6. Image single (0.08% engagement)
7. Text single (0.1% engagement)
8. Link post Premium (0.25%, suppressed)
9. Link post non-Premium (0%, invisible)

---

## Installation

### Claude Code

```bash
# Clone into your skills directory
git clone https://github.com/marcusdavidc/x-open-source-algorithm-skill.git ~/.claude/skills/x-open-source-algorithm-skill
```

### Manual

Copy the `x-phoenix-post-scorer/` folder into `~/.claude/skills/`.

---

## Usage

The skill triggers automatically when you ask Claude to write, optimize, or score X/Twitter content.

### Example prompts

```
"Write an X thread about Claude's 1M context window"
"Score this tweet for algorithmic reach"
"Optimize this thread for the X algorithm"
"Should this be a thread or an article?"
"Why did my last post underperform?"
```

### What you get back

1. **Format recommendation** with algorithm rationale
2. **Phoenix signal analysis** (which of the 19 signals your draft targets)
3. **Weighted score** (1-10, targeting 7.0+ for posting)
4. **Specific rewrites** with signal-by-signal rationale
5. **Posting strategy** (timing, first-30-min plan)

---

## Skill structure

```
x-phoenix-post-scorer/
├── README.md                          # This file
├── SKILL.md                           # Main skill instructions
└── references/
    ├── phoenix-scoring.md             # Full scoring model from source code
    └── algorithm-signals.md           # 2026 practical engagement data
```

---

## Sources

- [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) (Jan 2026, Apache 2.0)
- [twitter/the-algorithm](https://github.com/twitter/the-algorithm) (2023, historical reference)
- [Buffer analysis of 18.8M posts](https://buffer.com/resources/x-premium-review/) (Premium reach data)
- [X Algorithm 2026 deep dives](https://socialbee.com/blog/twitter-algorithm/) (behavioral observations)

---

## Key insight from the source code

The Jan 2026 algorithm eliminated all hand-engineered features. From the README:

> "We have eliminated every single hand-engineered feature and most heuristics from the system. The Grok-based transformer does all the heavy lifting."

This means the algorithm now learns entirely from your engagement history. The Phoenix transformer predicts what you'll engage with based on what you've engaged with before. Content quality and engagement velocity matter more than ever. Gaming specific features is no longer possible because there are no features to game. The only path to reach is creating content that genuinely triggers engagement.

---

## License

MIT

Built from Apache 2.0 licensed source code (xai-org/x-algorithm). This skill contains analysis and optimization guidance derived from the open source algorithm, not the algorithm code itself.
