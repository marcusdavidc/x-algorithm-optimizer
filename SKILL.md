---
name: x-open-source-algorithm-skill
description: Score and optimize X/Twitter posts using the actual open source algorithm (xai-org/x-algorithm, Jan 2026). Built from the Phoenix scoring model, weighted scorer, and 19 engagement signal predictions from X's Grok-based recommendation system. Use this skill whenever the user wants to write, optimize, score, or improve any X/Twitter content including tweets, threads, articles, or replies. Also use when the user mentions X algorithm, tweet optimization, thread writing, X engagement, Twitter reach, viral tweets, or wants to understand why a post performed well or poorly.
---

# X Algorithm Optimizer

Optimize X/Twitter content using the actual January 2026 open source algorithm from `xai-org/x-algorithm`.

## Source Material

This skill is built from three sources:

1. **The actual algorithm code** (Jan 2026): `~/Documents/Work/Social Media Guides/Platform Guides/x-algorithm-2026/`
2. **2026 practical guide**: `~/Documents/Work/David Brand/x-twitter-algorithm-2026.md`
3. **David Cyrus voice profile**: `~/Documents/Work/David Brand/david-cyrus-voice-profile.md` (for @mdavidcyrus posts)

Read `references/phoenix-scoring.md` for the full scoring model details.
Read `references/algorithm-signals.md` for the practical engagement hierarchy and tactics.

## Optional X/Twitter Evidence

When a draft depends on a current X/Twitter conversation, use source evidence
before scoring or rewriting. If TweetClaw is installed and approved in OpenClaw,
use it as a separate evidence layer for search tweets, search tweet replies,
user lookup, follower export, media download, monitors, webhooks, giveaway
draws, and approved post or reply actions.

Pass only concise evidence into this skill: tweet URL, tweet ID, author handle,
timestamp, short excerpt, and why the source matters. Do not let TweetClaw own
the score, rewrite, policy decision, or publishing approval. This skill owns the
algorithm analysis and rewrite strategy. The operator owns final approval.

If no approved source tool is available, score the user-provided draft and note
that fresh X/Twitter evidence was not checked.

## How the Algorithm Actually Works (Jan 2026)

X's "For You" feed is powered by four components:

1. **Home Mixer** (orchestration): assembles the feed through a pipeline of sources, hydrators, filters, scorers, and selectors
2. **Thunder** (in-network): serves recent posts from accounts you follow via in-memory store
3. **Phoenix** (ranking + retrieval): Grok-based transformer that predicts engagement probabilities
4. **Candidate Pipeline** (framework): reusable pipeline stages that run in parallel

The critical path for content creators: Phoenix predicts 19 engagement probabilities for every post, and a Weighted Scorer combines them into a single ranking score.

## The Phoenix Scoring Model

Phoenix predicts these engagement probabilities for every candidate post:

**Positive signals (higher = more reach):**

| Signal | What triggers it | Impact |
|--------|-----------------|--------|
| P(favorite) | Likes | Primary distribution signal |
| P(reply) | Replies to the post | Triggers +75 author engagement bonus |
| P(repost) | Retweets | Strong amplification signal |
| P(quote) | Quote tweets | Engaged discussion signal |
| P(click) | Link/content clicks | Interest signal |
| P(profile_click) | Clicking author's profile | Curiosity/authority signal |
| P(video_quality_view) | Watching video >50% | Only applies to video posts |
| P(share) | Share button usage | High-intent distribution |
| P(share_via_dm) | Sharing via DM | Personal recommendation signal |
| P(share_via_copy_link) | Copying link to share | Off-platform sharing intent |
| P(dwell) | Stopping to read/view | Attention signal |
| P(dwell_time) | How long they stay (continuous) | Depth of engagement |
| P(follow_author) | Following the author | Strongest authority signal |

**Negative signals (higher = penalized):**

| Signal | What triggers it | Impact |
|--------|-----------------|--------|
| P(not_interested) | "Not interested" button | Moderate penalty |
| P(block_author) | Blocking the author | Severe penalty |
| P(mute_author) | Muting the author | Significant penalty |
| P(report) | Reporting the post | Most severe penalty |

**The weighted score formula:**
```
Final Score = sum(weight_i * P(action_i))
```
Positive actions have positive weights. Negative actions have negative weights.

The exact weight values are excluded from the open source release (in a private `params` module), but the 2026 practical data gives us the observed hierarchy:
- Likes: +30 points
- Retweets: +20 points
- Reply with author response: +75 bonus
- Reply alone: +13.5

## How to Use This Skill

### Step 1: Determine Format

Based on the 2026 algorithm data, content formats rank:

1. **X Articles** (algorithm-boosted, $1M monthly prize, highest dwell time)
2. **Video thread** (video + 3-5 tweets, 0.42% engagement rate)
3. **Video single post** (5x text engagement)
4. **Image/GIF thread** (150% more interactions than text)
5. **Text thread** (3x single tweet engagement)
6. **Image/GIF single** (0.08% engagement)
7. **Text single** (0.1% engagement)
8. **Link post Premium** (0.25%, heavily suppressed)
9. **Link post non-Premium** (0% engagement, invisible)

**Decision tree:**
- Deep analysis, 1000+ words? -> X Article
- Punchy take, 5-10 points? -> Thread (7 tweets optimal)
- Quick insight, single idea? -> Single tweet with image/video
- Linking to external content? -> Native content first, link in reply only

### Step 2: Analyze the Draft

For each draft post, evaluate against the Phoenix scoring signals:

**Will this trigger positive signals?**
- [ ] Likely to get likes? (favorite_score) -> clear value, quotable insight
- [ ] Likely to get replies? (reply_score) -> provocative question, contrarian take, reply hook
- [ ] Likely to get retweets? (retweet_score) -> shareable, makes the sharer look smart
- [ ] Likely to get clicks? (click_score) -> curiosity gap, "what happened next"
- [ ] Likely to increase dwell time? (dwell_score) -> thread format, visual content, depth
- [ ] Likely to trigger profile visits? (profile_click_score) -> establishing authority, unique perspective
- [ ] Likely to trigger follows? (follow_author_score) -> demonstrating ongoing value

**Will this trigger negative signals?**
- [ ] Could trigger "not interested"? -> off-topic for your audience, unclear value
- [ ] Could trigger mutes? -> posting too frequently, repetitive content
- [ ] Could trigger blocks? -> aggressive, offensive, spam-like behavior

### Step 3: Optimize for the Algorithm

**The 30-Minute Rule:** The algorithm evaluates engagement velocity in the first 30 minutes. Front-load your best content.

**Premium multipliers:**
- In-network (followers): 4x visibility
- Out-of-network (non-followers): 2x visibility
- Overall: 10x reach advantage (Buffer analysis of 18.8M posts)

**Thread optimization:**
- First tweet: no number, use 🧵 emoji
- Start numbering from tweet 2: "2/7", "3/7"
- Each tweet under 200 characters (250 max)
- Line breaks between ideas
- 1-2 hashtags max, in final tweet only
- Links in final tweet or separate reply
- Optimal length: 7 tweets (5-10 range)

**What kills reach:**
- External links in main post (even Premium: 0.25% engagement)
- 3+ hashtags (engagement drops)
- No visual content (5x penalty vs video)
- Text-only posts (lowest format, 0.1%)
- Slow first-30-minute engagement

### Step 4: Score the Post

Rate each draft on a simplified Phoenix-aligned scale:

| Category | Score 1-10 | Weight |
|----------|-----------|--------|
| Reply potential (will people respond?) | _ | 30% |
| Repost potential (will people share?) | _ | 25% |
| Dwell time (will people stop and read?) | _ | 20% |
| Like potential (quick positive reaction?) | _ | 15% |
| Negative risk (could this backfire?) | _ | 10% |

**Weighted Score = (Reply * 0.3) + (Repost * 0.25) + (Dwell * 0.2) + (Like * 0.15) - (Negative * 0.1)**

Target: 7.0+ for posting. Below 6.0, rewrite.

### Step 5: Rewrite Suggestions

When suggesting rewrites, explain which Phoenix signal you're targeting:

**Example:**
- Original: "Claude's 1M context window is now available."
- Problem: Low reply_score (no reason to respond), low dwell_score (nothing to read)
- Rewrite: "Anthropic just gave Claude a million-token context window. I turned mine off. Here's why. 🧵"
- Why: Opens curiosity gap (click_score), demands engagement (reply_score), signals thread (dwell_score)

## Voice Reference

For @mdavidcyrus posts, read `~/Documents/Work/David Brand/david-cyrus-voice-profile.md` and apply:
- Direct, pragmatic, builder, contrarian
- Executive voice, not content creator
- Lead with insight, not story
- No em dashes, no AI-tell words
- No engagement bait ("Agree?", "Thoughts?")
- Statement energy, end on declarations

## Anti-patterns (from the algorithm code)

The algorithm has explicit **negative signal predictions**. These actions cause the scoring model to suppress your content:

1. **P(not_interested)** increases when: content is off-topic for your community (SimClusters mismatch), repetitive themes, unclear value proposition
2. **P(block_author)** increases when: aggressive tone, spam-like posting frequency, controversial content without substance
3. **P(mute_author)** increases when: over-posting (more than 5-7 daily), thread-bombing, excessive self-promotion
4. **P(report)** increases when: misleading claims, harassment, harmful content

The **Author Diversity Scorer** also penalizes seeing the same author too many times in one feed session. Each subsequent post from the same author gets a decaying multiplier. Posting quality over quantity matters.

## Output Format

When optimizing a post, provide:

1. **Format recommendation** (article / thread / single + why)
2. **Phoenix signal analysis** (which signals this will trigger, which it won't)
3. **Score** (weighted 1-10 scale above)
4. **Specific rewrites** (with signal rationale for each change)
5. **Posting strategy** (timing, first-30-min plan, reply strategy)
