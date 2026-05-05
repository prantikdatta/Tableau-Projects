# Data Dictionary

**Domain:** Music streaming analytics platform
**Scope:** 2021-01-01 to 2024-12-31 (48 months)
**Currency:** USD
**Grain:** 9 dimension tables, 2 fact tables, 1 bridge table

---

## Model Diagram

```
                      ┌──────────────┐
                      │  dim_genre   │
                      └──────┬───────┘
                             │
                 ┌───────────┴──────────┐
                 ▼                      ▼
         ┌──────────────┐      ┌──────────────┐
         │  dim_artist  │◄─────│  dim_track   │
         └──────────────┘      └──────┬───────┘
                                      │
               ┌──────────────────────┴──────────────────┐
               │                                         │
               ▼                                         ▼
    ┌───────────────────────┐                  ┌─────────────────┐
    │ bridge_playlist_track │                  │ fact_listening_ │
    └──────────┬────────────┘                  │    session      │
               │                               └──────┬──────────┘
               ▼                                      │
        ┌──────────────┐    ┌──────────────┐          │
        │ dim_playlist │    │  dim_device  │──────────┤
        └──────┬───────┘    └──────────────┘          │
               │                                      │
               └──────────┬───────────────────────────┤
                          ▼                           │
                   ┌──────────────┐                   │
                   │   dim_user   │───────────────────┤
                   └──┬───────┬───┘                   │
                      │       │                       │
     ┌────────────────┘       └──────────────┐        │
     ▼                                       ▼        │
┌─────────────────┐                  ┌──────────────┐ │
│ dim_subscription│◄─────────────────│   fact_sub   │ │
│     _plan       │                  │ scription_   │ │
└─────────────────┘                  │    event     │ │
                                     └──────┬───────┘ │
                                            │         │
                   ┌──────────────┐         │         │
                   │ dim_country  │◄────────┴─────────┘
                   └──────────────┘
                   ┌──────────────┐
                   │   dim_date   │  (conformed date dimension)
                   └──────────────┘
```

---

## Dimension Tables

### dim_genre
Musical genre lookup.

| Column | Type | Description |
|---|---|---|
| **genre_id** (PK) | int | Surrogate key |
| genre_name | string | Genre label (Rock, Pop, Jazz, etc.) |

*Rows: 10*

### dim_artist
Music artists.

| Column | Type | Description |
|---|---|---|
| **artist_id** (PK) | int | Surrogate key |
| artist_name | string | Real-world artist name |
| home_country | string | Artist's country of origin |
| debuts_year | int | Year of first major commercial release |
| genre_id (FK) | int | → dim_genre |

*Rows: 448*

### dim_track
Individual tracks.

| Column | Type | Description |
|---|---|---|
| **track_id** (PK) | int | Surrogate key |
| title | string | Track title |
| artist_id (FK) | int | → dim_artist |
| genre_id (FK) | int | → dim_genre (always matches the artist's genre) |
| release_date | date | Track release date (≥ artist's debut year) |
| is_algorithmic_recommendation | bool | Track is algorithm-surfaced by the platform |

*Rows: 774*

### dim_user
Platform users.

| Column | Type | Description |
|---|---|---|
| **user_id** (PK) | int | Surrogate key |
| account_created | date | Account creation date |
| age | int | User age |
| gender | string | Self-reported gender (Male, Female, Non-binary, Prefer not to say) |
| country_code (FK) | string | ISO 2-letter → dim_country (home country) |
| subscription_tier | string | Current tier label: Free / Premium / Family |
| plan_id (FK) | int | → dim_subscription_plan |
| family_account_id (FK) | int | → dim_user (family head; self-ref, nullable) |
| device_os | string | Primary device OS |
| is_fraud_cluster | bool | User flagged as part of a fraud cluster |

*Rows: 961*

### dim_device
Playback devices.

| Column | Type | Description |
|---|---|---|
| **device_id** (PK) | int | Surrogate key |
| device_type | string | Mobile, Desktop, Car, Smart Speaker, Smart TV, etc. |
| os_name | string | Device OS |

*Rows: 366*

### dim_playlist
User-created playlists.

| Column | Type | Description |
|---|---|---|
| **playlist_id** (PK) | int | Surrogate key |
| creator_user_id (FK) | int | → dim_user |
| playlist_title | string | Playlist name |
| is_public | bool | Public visibility flag |
| created_date | date | Creation date |
| num_tracks | int | Track count (matches bridge_playlist_track rows) |

*Rows: 767*

### dim_subscription_plan
Subscription plan catalog.

| Column | Type | Description |
|---|---|---|
| **plan_id** (PK) | int | Surrogate key |
| plan_name | string | Free / Premium / Family |
| monthly_price_usd | decimal | Monthly recurring charge in USD |
| max_accounts | int | Accounts allowed per plan |
| ad_supported | bool | Whether ads are served on this tier |
| offline_listening | bool | Offline download availability |

*Rows: 3*

### dim_country
Geographic rollup.

| Column | Type | Description |
|---|---|---|
| **country_code** (PK) | string | ISO 2-letter code |
| country_name | string | Full country name |
| region | string | UN sub-region (e.g., Northern Europe) |
| continent | string | Continent |

*Rows: 10*

### dim_date
Conformed calendar dimension.

| Column | Type | Description |
|---|---|---|
| **date_key** (PK) | string | YYYY-MM-DD |
| full_date | date | Calendar date |
| year | int | Calendar year |
| quarter | int | 1-4 |
| month | int | 1-12 |
| month_name | string | January…December |
| week_of_year | int | ISO week number |
| day_of_month | int | 1-31 |
| day_of_week | int | 1=Monday … 7=Sunday |
| day_name | string | Monday…Sunday |
| is_weekend | bool | Saturday or Sunday |
| is_month_end | bool | Last calendar day of month |
| is_quarter_end | bool | Last calendar day of quarter |
| is_year_end | bool | Dec 31 |

*Rows: 1,461 (full coverage 2021-01-01 through 2024-12-31)*

---

## Bridge Table

### bridge_playlist_track
Many-to-many link between playlists and tracks.

| Column | Type | Description |
|---|---|---|
| playlist_id (FK) | int | → dim_playlist |
| track_id (FK) | int | → dim_track |
| position | int | Track position within the playlist |
| added_date | date | When the track was added to the playlist |

*Composite PK: (playlist_id, track_id) — 21,192 rows*

---

## Fact Tables

### fact_listening_session
One row per listening session (a user playing a track on a device at a point in time).

| Column | Type | Description |
|---|---|---|
| **session_id** (PK) | int | Surrogate key |
| user_id (FK) | int | → dim_user |
| track_id (FK) | int | → dim_track |
| artist_id (FK) | int | → dim_artist (denormalized; matches dim_track) |
| genre_id (FK) | int | → dim_genre (denormalized; matches dim_track) |
| playlist_id (FK) | int | → dim_playlist (context of the play) |
| device_id (FK) | int | → dim_device |
| country_code (FK) | string | → dim_country (country at time of session — may differ from user home) |
| subscription_tier | string | Tier at time of session (may differ from current dim_user tier) |
| listen_start_ts | datetime | Session start timestamp |
| listen_seconds | int | Duration actually listened |
| skipped | bool | Whether the track was skipped |
| session_weekday | string | Derived weekday name |
| new_artist_discovered | bool | First play of this artist by this user |
| estimated_revenue_usd | decimal | Revenue for this session (ad revenue for Free tier; per-stream royalty for paid when ≥ 30s) |

*Rows: ~224,000*

**Revenue computation:**
- Free tier → `(listen_seconds / 60) * 0.003` (ad CPM model)
- Premium/Family tier → `$0.004 per qualifying play (≥ 30 seconds)`

### fact_subscription_event
One row per subscription lifecycle event (signup, upgrade, downgrade, churn, retention).

| Column | Type | Description |
|---|---|---|
| **event_id** (PK) | int | Surrogate key |
| user_id (FK) | int | → dim_user |
| event_type | string | signup / upgrade / downgrade / churn / retention |
| event_ts | date | Event date |
| from_tier | string | Tier before event (nullable for signup) |
| to_tier | string | Tier after event |
| mrr_before_usd | decimal | MRR before event |
| mrr_after_usd | decimal | MRR after event |
| mrr_change_usd | decimal | Delta (positive = expansion, negative = contraction/churn) |
| country_code (FK) | string | → dim_country (country at time of event) |
| trigger_context | string | Marketing / attribution context (free text) |

*Rows: ~3,600*

**Event type semantics:**
- `signup` — user joins (from_tier is typically null)
- `upgrade` — moves to a higher-priced plan
- `downgrade` — moves to a lower-priced plan
- `churn` — moves to Free or cancels
- `retention` — renewal or retained on same plan

For each user, the **most recent event's `to_tier`** is guaranteed to match `dim_user.subscription_tier`.

---

## Referential Integrity Guarantees

All primary keys are unique. All foreign keys resolve. Session-level facts carry their own as-of-event `country_code` and `subscription_tier` (may differ from `dim_user` — this is intentional and enables tenure-at-event, roaming, and pre/post-upgrade behavioural analysis).

## Data Volume Note

The dataset has been engineered to support meaningful analytical pattern detection while remaining laptop-friendly:
- **fact_listening_session: ~224,000 rows** (~58 sessions/user/year average, with heavy-user / light-user variance via lognormal engagement distribution)
- **fact_subscription_event: ~3,600 rows** (~3.8 events/user lifetime average)
- All dimension tables retained at their current cardinalities.

The dataset is deliberately seeded with detectable patterns under ±15–20% noise — sufficient density for trend, seasonal, cohort, and leading-indicator analysis.
