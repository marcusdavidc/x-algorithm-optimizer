# Phoenix Scoring Model (from xai-org/x-algorithm, Jan 2026)

## Source

All data in this file comes from the open source X algorithm at:
`~/Documents/Work/Social Media Guides/Platform Guides/x-algorithm-2026/`

Key files:
- `home-mixer/scorers/weighted_scorer.rs` - The actual weighted scoring formula
- `home-mixer/scorers/phoenix_scorer.rs` - Phoenix ML prediction extraction
- `home-mixer/scorers/author_diversity_scorer.rs` - Author repeat penalty
- `home-mixer/scorers/oon_scorer.rs` - Out-of-network score adjustment
- `phoenix/README.md` - Grok-based transformer architecture
- `phoenix/recsys_model.py` - Ranking model code (JAX)

## Pipeline: How a Post Gets Scored

```
Post Created
    |
    v
Thunder (in-network store) OR Phoenix Retrieval (out-of-network)
    |
    v
Home Mixer orchestration
    |
    v
Pre-scoring Filters (remove duplicates, blocked, muted, old, self-posts)
    |
    v
Phoenix Scorer -> predicts 19 engagement probabilities
    |
    v
Weighted Scorer -> combines into single score using private weights
    |
    v
Author Diversity Scorer -> penalizes repeated authors in feed
    |
    v
OON Scorer -> adjusts out-of-network content scores
    |
    v
Select top K -> your feed
```

## The 19 Phoenix Predictions

From `phoenix_scorer.rs`, the model predicts:

### Discrete Actions (probability 0-1):
1. `ServerTweetFav` (favorite/like)
2. `ServerTweetReply` (reply)
3. `ServerTweetRetweet` (retweet/repost)
4. `ServerTweetQuote` (quote tweet)
5. `ClientTweetClick` (click on tweet)
6. `ClientTweetClickProfile` (click author profile)
7. `ClientTweetVideoQualityView` (watch video >threshold)
8. `ClientTweetPhotoExpand` (expand photo)
9. `ClientTweetShare` (share button)
10. `ClientTweetClickSendViaDirectMessage` (share via DM)
11. `ClientTweetShareViaCopyLink` (copy link)
12. `ClientTweetRecapDwelled` (dwell/stop scrolling)
13. `ClientQuotedTweetClick` (click on quoted tweet)
14. `ClientTweetFollowAuthor` (follow the author)
15. `ClientTweetNotInterestedIn` (not interested)
16. `ClientTweetBlockAuthor` (block)
17. `ClientTweetMuteAuthor` (mute)
18. `ClientTweetReport` (report)

### Continuous Actions:
19. `DwellTime` (predicted seconds spent on post)

## Weighted Scoring Formula

From `weighted_scorer.rs`:

```rust
Final Score =
    P(favorite) * FAVORITE_WEIGHT
  + P(reply) * REPLY_WEIGHT
  + P(retweet) * RETWEET_WEIGHT
  + P(photo_expand) * PHOTO_EXPAND_WEIGHT
  + P(click) * CLICK_WEIGHT
  + P(profile_click) * PROFILE_CLICK_WEIGHT
  + P(video_quality_view) * VQV_WEIGHT  // only if video > MIN_DURATION
  + P(share) * SHARE_WEIGHT
  + P(share_via_dm) * SHARE_VIA_DM_WEIGHT
  + P(share_via_copy_link) * SHARE_VIA_COPY_LINK_WEIGHT
  + P(dwell) * DWELL_WEIGHT
  + P(quote) * QUOTE_WEIGHT
  + P(quoted_click) * QUOTED_CLICK_WEIGHT
  + DwellTime * CONT_DWELL_TIME_WEIGHT
  + P(follow_author) * FOLLOW_AUTHOR_WEIGHT
  + P(not_interested) * NOT_INTERESTED_WEIGHT  // negative
  + P(block_author) * BLOCK_AUTHOR_WEIGHT      // negative
  + P(mute_author) * MUTE_AUTHOR_WEIGHT        // negative
  + P(report) * REPORT_WEIGHT                  // negative
```

The exact weight values are in a private `params` module excluded from the open source release. However, the 2026 practical data gives observed weights:
- Likes: +30 points equivalent
- Retweets: +20 points equivalent
- Reply with author response: +75 bonus
- Reply alone: +13.5

## Author Diversity Scorer

From `author_diversity_scorer.rs`:

The algorithm penalizes seeing the same author repeatedly in one feed session.

```
multiplier(position) = (1 - floor) * decay^position + floor
```

- `position` = how many times this author has already appeared
- `decay` = exponential decay factor (private param)
- `floor` = minimum multiplier (never goes to zero, private param)

First post from an author: full score (multiplier = 1.0)
Second post: reduced score
Third post: further reduced
...and so on

Implication: quality over quantity. 3-5 great posts beat 10 mediocre ones.

## OON (Out-of-Network) Scorer

From `oon_scorer.rs`:

Posts from accounts the user does NOT follow get multiplied by `OON_WEIGHT_FACTOR` (private param). This is typically < 1.0, meaning in-network content has a natural advantage.

For your content to reach non-followers, it needs to score significantly higher on Phoenix predictions than competing in-network content.

## Video Quality View (VQV) Eligibility

From `weighted_scorer.rs`:

Video posts only get the VQV weight boost if the video duration exceeds `MIN_VIDEO_DURATION_MS`. Very short videos (likely under a few seconds) don't qualify.

## Candidate Isolation

From `phoenix/README.md`:

Critical architecture detail: during ranking, **candidates cannot attend to each other**. Each post is scored independently against your engagement history. This means:
- A post's score doesn't change based on what else is in the batch
- Scores are consistent and cacheable
- Your post competes on its own merits, not relative to the batch
