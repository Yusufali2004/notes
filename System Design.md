# System Design Interview Handbook

> **Source:** freeCodeCamp, *"Design Software Systems Like a Senior Engineer – Complete Course"* by Hayk Simonyan (3h 16m). Notes follow the video transcript. Timestamps like `[1:33:05]` let you jump back to the video.
>
> **Legend:** Unmarked text = from the video. Blocks starting with **➕ Added** = my enrichment (extra concepts, corrections, interview tips).
>
> **Numbers** are the video's assumptions. In an interview, always state and confirm your own.
>
> **How to use:** First 2-3 times, read Parts 1-5 fully and redraw each system from memory. After that, use only [Part 10: Cheat Sheet](#part-10-cheat-sheet).

## Video map

| Topic | Time |
|---|---|
| Why system design matters, formats, what is scored | 0:00 |
| The 5-step framework, trade-offs and extensions | 9:21 |
| **System 1: Music streaming (Spotify-style)** | 23:55 |
| Requirements and capacity / Architecture and flows / API / Storage / Scaling | 23:55 / 29:24 / 35:23 / 40:04 / 44:37 |
| **System 2: Hotel reservation (Booking.com-style)** | 50:52 |
| Requirements / Capacity and architecture / API and idempotency / Data model | 50:52 / 58:22 / 1:11:02 / 1:22:34 |
| Concurrency / Sharding and caching / Elasticsearch / Replication and global routing | 1:33:05 / 1:34:44 / 1:40:41 / 1:45:19 |
| **System 3: WhatsApp-style chat** | 1:50:45 |
| Scale estimate / Architecture and real-time / Data model and API flows / Multi-device | 1:50:45 / 1:56:38 / 2:07:04 / 2:13:10 |
| Fanout, media, CDN / Event-driven and batch / Encryption and rate limiting | 2:22:37 / 2:31:44 / 2:39:18 |
| Sharding and cleanup / Failure handling and global scale / Feature flags / How to get better | 2:49:45 / 2:57:53 / 3:09:08 / 3:11:14 |

## Table of Contents

1. [Part 1: Why System Design Matters and What Is Scored](#part-1-why-system-design-matters-and-what-is-scored)
2. [Part 2: The 5-Step Framework](#part-2-the-5-step-framework)
3. [Part 3: System 1, Music Streaming](#part-3-system-1-music-streaming)
4. [Part 4: System 2, Hotel Booking](#part-4-system-2-hotel-booking)
5. [Part 5: System 3, WhatsApp-Style Chat](#part-5-system-3-whatsapp-style-chat)
6. [Part 6: Concepts Reference](#part-6-concepts-reference)
7. [Part 7: Capacity Math Reference](#part-7-capacity-math-reference)
8. [Part 8: How to Actually Get Better](#part-8-how-to-actually-get-better)
9. [Part 9: Interview Questions and Answers](#part-9-interview-questions-and-answers)
10. [Part 10: Cheat Sheet](#part-10-cheat-sheet)

---

# Part 1: Why System Design Matters and What Is Scored

## 1.1 Why it matters `[0:00]`

- Most software engineering roles expect system design knowledge, either for the job or for the interview.
- For **senior and staff** roles it is one of the deciding factors. It is increasingly expected for **mid-level** too, because engineers now use AI to write implementation code.
- Companies want to know: do you understand the **bigger picture**, how components talk to each other, how to **design from scratch**, and how to **discuss trade-offs**?
- Depth expected differs by level (mid vs senior+), but you must prepare either way.

## 1.2 Four interview formats `[2:19]`

| Format | What happens |
|---|---|
| Whiteboarding (most common) | Design a full system or feature from scratch in a session |
| Bottleneck discussion | Interviewer describes the company's existing infrastructure; you suggest fixes |
| Past-project walkthrough | Questions to check you really understand the architecture of what you built |
| Conceptual questions | Test depth of knowledge on system design concepts |

Even if you know every concept, you still need a **framework to navigate a 45 or 60 minute** discussion.

## 1.3 Why it feels harder than coding rounds `[3:17]`

- Coding rounds have clear inputs/outputs and you can run tests. System design is the opposite.
- Requirements are **vague**. It is **your job** to clarify and lock the scope, and to define the output (your solution).
- **No single right answer.** It tests **how you think**, not just what you know.

## 1.4 The five dimensions you are scored on `[4:51]`

Most candidates only memorize a few architectures (Spotify, WhatsApp). That is just one dimension.

| # | Dimension | What the interviewer looks for | How to show it |
|---|---|---|---|
| 1 | **Communication and collaboration** | Do you ask clarifying questions and validate assumptions, or assume and start building? Is it easy to work with you? | Clarify first. Think out loud. Check in with the interviewer. |
| 2 | **Structured thinking** | Can you break a vague problem into outlined steps, or jump between random decisions and designs? | State your outline. Always be clear what you are doing now and what is next. |
| 3 | **Technical judgment** | Can you explain **why** you chose SQL, this cache, this queue? Based on requirements, not habit. | Justify every choice with a requirement. Mention alternatives. |
| 4 | **Practical depth** | Do you think about failure modes, scale, and what can go wrong? Separates people who shipped production systems from theory-only learners. | Talk about failures, bottlenecks, monitoring. |
| 5 | **Time management** | Do you avoid getting stuck on one area and running out of time with only high-level design? | Ask early where they want focus, track time, move on when needed. |

Each pillar maps to real work: poor time management in the interview means poor time management on the job, and so on.

---

# Part 2: The 5-Step Framework

A repeatable structure for any whiteboard system design interview. **60 minutes is more common than 45.** `[9:21]`

## 2.1 Overview (60-minute budget)

| Step | What | Time |
|---|---|---|
| 1 | **Clarify requirements** (functional and non-functional), scope, out-of-scope | 5-10 min |
| 2 | **Data model and API layer** (can swap order with step 3) | ~15-20 min |
| 3 | **High-level design** (components and read/write data flows) | ~15 min |
| 4 | **Bottlenecks, scale, reliability** | 5-10 min |
| 5 | **Wrap-up: trade-offs and extensions** | last few min |

Steps 2 and 3 are interchangeable: start with data/API if modeling entities first helps you, or start with high-level design if imagining the user flow first helps. The speaker usually starts with high-level design. Do not spend more than 15-20 min on either.

> **➕ Added (mnemonic):** Clarify, Data/API, High-level, Scale, Trade-offs. Say "outline" at the start: *"I'll clarify requirements, model data and APIs, sketch the architecture, then discuss scale, failures, and trade-offs."*

## 2.2 Step 1: Clarify requirements `[11:12]`

Do **not** design anything yet. This is the most common mistake.

**Functional requirements:** the main use cases, who the users are, what platform, and what is **explicitly out of scope**. You cannot design "Spotify" in 60 minutes (it takes multiple teams years), so lock the core use cases.

**Non-functional requirements (constraints):**
- Daily / monthly active users
- Read-to-write ratio
- Latency requirements
- Acceptable downtime (availability)

**Check-in before moving on:**
> "Does this scope and these requirements make sense before I move to high-level design?"

If they say yes, you are good. If they hint, you missed something in the functional or non-functional requirements. Fix it first.

## 2.3 Step 2: Data model and API layer `[13:11]`

**Data model**
- Main entities (e.g., YouTube: videos and users) and relationships.
- Primary keys, foreign keys, key indexes.
- How big each record can get.
- Which data needs **strong consistency** and where **eventual consistency** is fine.

**API contracts**
- Main endpoints with request and response formats.
- Pagination, error handling, rate limiting, versioning.
- Follow REST best practices (it is usually REST).

**Database choice:** pick (Postgres, MongoDB, etc.), explain **why given the requirements**, and walk through the trade-offs of what you did not pick.

## 2.4 Step 3: High-level design `[15:00]`

- Design components and show how data flows through them.
- **If nothing comes to mind, start with the four almost-always-present components:**
  1. **API servers** (accept requests)
  2. **Load balancer** (distributes traffic across them)
  3. **Database** (persists data)
  4. **Cache layer** (optional; caches common reads)
- Map the **user journey**: front end (web/mobile/desktop) -> API gateway / load balancer -> API servers -> cache -> (optional message queues) -> database.
- Walk through **read and write** operations, request by request.
- If you use a cache, mention **TTL and the invalidation mechanism**.
- For different parts of the system, discuss **consistency vs availability vs partition tolerance (CAP)**. Different components may need different choices. This shows you understand the trade-offs.

## 2.5 Step 4: Bottlenecks, scale, reliability `[16:56]`

This is where you show what breaks under heavy load, not only the happy path. It is one of the clearest signals separating **mid-level** from **senior** candidates.

- Find **single points of failure (SPOFs)** and propose fixes.
- **Fallbacks:** can you fall back to a slower but functional path?
- **Redundancy:** multi-AZ replicas, application servers auto-scaling across AZs, distributed cache with replication.
- **Monitoring, logging, alerting:** often overlooked, a strong senior signal. Cover API latency, error rates, thresholds, alerts.
- Check in: "Are there specific failure scenarios you want me to address?"

## 2.6 Step 5: Trade-offs and extensions `[18:40]`

- Summarize key decisions, what you **prioritized**, and what you **sacrificed**, with the **why**.
- Propose extensions: "If I had more time or load grew 10x, I would...".
- You cannot produce a perfect design in 60 minutes. The goal is to show you can **extend and improve** it.

## 2.7 Do and don't `[20:01]`

| Do | Don't |
|---|---|
| **Ask clarifying questions before designing** (jumping to a solution is a red flag) | Stay silent and draw diagrams |
| **Treat the interviewer as a colleague**: two engineers solving together | Talk nonstop without inviting their opinion (monologue) |
| **Suggest multiple approaches before committing** | Pick a technology without justification |
| **Explain trade-offs** for every technology choice | Skip the "why not X" |
| **Check in at each major step** ("did we miss anything?") | Treat it like a take-home with no communication |

**Important detail:** even for an obvious choice (e.g., SQL for payments where consistency matters most), **say why SQL and why not NoSQL**. If you don't, the interviewer assumes you don't know.

**Recap:** requirements -> data and API -> high-level design (read/write flows) -> scale, failure modes, reliability -> trade-offs and what you'd improve.

> **➕ Added: Trade-off sentence you can reuse for every decision**
> "I'm choosing **A** over **B** because the requirement is **R**. The downside is **D**, which I'd mitigate with **M**."

---

# Part 3: System 1, Music Streaming

*(Spotify-style.)* Classic because it covers audio delivery, metadata, object storage. The video's flow: requirements and capacity -> high-level architecture and flows -> API -> storage/DB -> scaling. `[23:55]`

## 3.1 Requirements `[25:02]`

**Scope (given):** ~**500,000 users**, ~**30 million songs**.

**Functional**
- Artists **upload** songs.
- Users **search** and **play** songs.
- Users **create and manage playlists**.
- User **profiles** (to keep playlists).
- Basic **monitoring and observability**: server health checks, error tracking, performance metrics.

**Audio assumptions**
- Formats: **OGG and AAC** (compressed, small).
- Bitrates: **64 kbps** (mobile data saving), **128 kbps** (standard), **320 kbps** (premium).
- Average file size at 128 kbps: **~3 MB**. (Confirm this with the interviewer.)

## 3.2 Capacity planning `[26:33]`

| Item | Calculation | Result |
|---|---|---|
| Song storage | 30M x 3 MB | **90 TB** raw audio (plan **2-3x** for regional replicas and artist re-uploads/versions) |
| Song metadata | 30M x ~100 bytes (title, artist ref, duration, file URLs) | **3 GB** |
| User metadata | 500K x 1 KB (profile, preferences, playlists) | **0.5 GB** |
| Daily bandwidth | avg listen ~3.5 min; 128-160 kbps ~ **3-4 MB per stream**; **10-15 songs/user/day** | Large **egress cost** (cloud providers charge when data leaves their network) |

**Key insight:** audio dominates both **storage cost and bandwidth**. Metadata is tiny. Main storage = **object storage**.

> **➕ Added (finishing the bandwidth math):** 500K users x 10-15 songs x 3-4 MB ~ **15-30 TB/day** egress. At 50M users it becomes ~**1.5-3 PB/day**, which is why CDN is essential at scale.

## 3.3 High-level architecture `[29:24]`

```
 Mobile / Web app
        |
        v
 +----------------+      round robin or least connections
 | Load Balancer  |      + health checks on API servers
 +-------+--------+
         v
 +-------------------+     validate JWT
 | REST API servers  |-----------------------+
 | (stateless)       |                       |
 +----+---------+----+                       |
      |         | generate signed URL        |
      v         v                            v
 +-----------+  +--------------------+  (later: CDN in front of
 | SQL DB    |  | Blob / Object      |   blob storage + regions)
 | metadata, |  | Storage (S3)       |
 | profiles  |  | audio files        |
 +-----------+  +--------------------+
```

- **Load balancer:** round robin or **least connections**; also does health checks.
- **API servers:** RESTful, validate **JWT**, query SQL for metadata, produce **signed URLs**.
- **SQL database:** song metadata and user profiles.
- **Blob storage (e.g., S3):** audio files. Scalable, durable, cost-effective for large files.
- **Signed URL:** a one-time URL that **expires** after some time so audio links are not permanently accessible.

### Read workflow: play a song `[31:16]`

1. User taps play -> `GET /songs/{id}` -> load balancer -> API server.
2. API validates **JWT**. If invalid, return an **authentication error** and stop.
3. Query **song metadata** from SQL.
4. Generate a **signed URL** in the API server.
5. Return **metadata + signed URL** to the app.
6. App makes **range requests** for audio chunks to blob storage using the signed URL; blob storage **streams** the audio back.
7. Analytics: separate `POST /songs/{id}/play` with metadata to store for later.

### Write workflow: artist uploads `[33:03]`

1. App `POST`s song + metadata -> load balancer -> API server.
2. Validate **file format and size** (limits). If invalid, reject (unsupported format/size).
3. Store the audio file in **blob storage**.
4. **Extract audio metadata** (duration, bitrate) and combine with body fields (title, artist...).
5. **Insert metadata into SQL.**
6. Organize files in a **hierarchical key** like `artist/{artistId}/album/{albumId}/{songId}`. (Object storage has no real folders; it is a naming illusion that is still handy.)
7. **CDN invalidation:** the CDN caches artist pages, album metadata, song details; clear stale entries so users see latest content.
8. Respond "upload successful".

## 3.4 API design `[35:21]`

| Purpose | Request |
|---|---|
| Search with pagination | `GET /search?query=...&type=song&limit=20&offset=0` |
| Trending (optional genre filter) | `GET /songs/trending?genre=...&limit=...` |
| Songs by artist | `GET /artists/{id}/songs?limit=...` |
| Song metadata + stream URL | `GET /songs/{id}` |
| Direct streaming (alternative to signed URLs) | `GET /songs/{id}/stream` |
| Get playlist (optionally with songs) | `GET /playlists/{id}?include=songs` |
| Create playlist | `POST /playlists` body: name, public/private |
| Add songs (with position) | `PUT /playlists/{id}/songs` body: song IDs + position |
| Remove song | `DELETE /playlists/{id}/songs/{songId}` |
| Like / unlike | `POST /songs/{id}/like` |
| My playlists | `GET /users/me/playlists?limit=...` |
| My liked songs | `GET /users/me/likes?limit=50&offset=...` (use consistent **plural** naming) |
| Follow artist | `POST /users/me/follow` body: artistId |
| Record play event | `POST /songs/{id}/play` |

> **➕ Added:** Prefer **cursor pagination** over `offset` for large lists. Make `PUT/DELETE` idempotent. Rate-limit `search`.

## 3.5 Storage and schema `[40:03]`

**Blob storage (S3 / Azure Blob):** audio is **immutable**. To change a song, re-upload and delete the old one. Key layout: `artist/{id}/album/{id}/{songId}`.

**Relational DB (PostgreSQL / MySQL):**

```
users          (id PK, email, password_hash, created_at, last_login,
                subscription_type, country)
playlists      (id PK, owner_id FK->users, name, created_at)
playlist_items (playlist_id, song_id, position, added_at)      -- join table
songs          (id PK, title, duration, file_url, ...)
artists        (id PK, name, country, ...)
artist_songs   (artist_id, song_id)                            -- join table
```

**Why SQL:** lots of **joins** (playlists-songs, songs-artists) and complex queries like "find all songs by artists a user follows".

**Indexes:** `songs.title`, `artists.name`, join-table `song_id`, `playlist_items` index (fast "songs in playlist" and "playlists containing song").

> **➕ Added (clarification):** Plain B-tree indexes make **exact and prefix** lookups fast, but they do **not** handle typos or fuzzy matching. For typo-tolerant search at scale use a search engine (see Elasticsearch in Part 4).

## 3.6 Scaling to millions `[44:33]`

**New scale:** **50M users, 200M songs.**

| Item | Calculation | Result |
|---|---|---|
| Song metadata | 200M x 100 B | **20 GB** |
| User metadata | 50M x 1 KB | **50 GB** |
| Audio | 200M x 3 MB | **600 TB** (x2-3 for replicas/versions) |

**Steps:**
1. **CDN edges per region** (North America, Europe, South America...). Use **LRU** eviction; on a cache miss, fetch from origin storage.
2. **Load balancers in multiple regions** (at least two) mapping requests to API servers.
3. **API servers are stateless**, so scale by simply adding replicas. Design them truly stateless (no sessions, no local storage).
4. **Database read/write split:** a **leader** for writes, **follower** replicas (kept in sync) for reads such as song/playlist metadata.
5. **Sharding** if still not enough: e.g., geographic shards (US shard, Europe shard) and **partition songs by artist ID ranges**. Gives horizontal scalability and keeps related data together.
6. **Blob storage replicated across regions**: Europe users upload to Europe storage, North America to its own; all use **signed URLs** for access.
7. **Start simple.** Do not over-engineer for few users; scale gradually.

## 3.7 The senior vs mid-level pyramid `[48:51]`

- **Bottom (largest, most important): Planning/blueprinting.** Requirements, capacity estimation, architecture.
- **Middle: Tech selection.** SQL vs NoSQL (and which one), caching layers, eviction policy, storage.
- **Top (smallest): Coding.** Needs the least brainpower and AI can do it faster and cheaper.

Many mid-level developers have this **upside down** (start coding with no plan). Engineers who think through problems before touching the keyboard never struggle to find roles.

## 3.8 Trade-offs to mention

| Decision | Trade-off |
|---|---|
| Signed URLs | Protect audio links; they expire so clients must refresh |
| SQL for metadata | Joins and consistency; harder to scale writes (solved with replicas, then shards) |
| CDN | Low latency, less origin load and egress; cache invalidation complexity |
| Stateless API | Easy scaling; state must live in DB/cache |
| Start simple | Less over-engineering; must evolve as load grows |

---

# Part 4: System 2, Hotel Booking

*(Booking.com / Airbnb style.)* Not a simple CRUD app with caching. The real challenges: **concurrency**, **consistency across systems**, and **designing for failure**. Classic bug: two users book the last room at the same time with a naive `select then insert`, so you charge two cards for one room. `[50:52]`

Video plan: requirements and estimation -> high-level design (API, schema, flows) -> scale to **~1,000 QPS and ~17 TPS** -> handle failures.

## 4.1 Clarifying questions and requirements `[53:19]`

**Clarifying questions and the chosen answers**
| Question | Answer |
|---|---|
| Payments on the platform? | **Yes** (need a payment system) |
| Booking channels? | **Web and mobile** |
| Cancellations and overbooking? | **Yes.** Hotels overbook (e.g., ~10%) expecting cancellations |
| Dynamic pricing? | **Yes**, price varies by demand and dates (e.g., Christmas) |
| Admin panel to manage hotels? | **Yes** (add/update/remove hotels) |

**Tip for finding core features:** *"If we removed this from Booking.com, does it stop being Booking.com?"* Searching by location passes this test.

**Functional**
- Search by **location, dates, guests, price range**, and other filters.
- **Reserve**, **view**, and **cancel** reservations.
- **Prevent double bookings.**
- **Payment processing** (Stripe, PayPal, ...).
- **Notifications** for bookings, cancellations, payments (to users and hotel admins/brokers).

**Non-functional**
- **Strong consistency for booking** (non-negotiable: no two users can book the same room for the same dates).
- **Eventual consistency is fine for search/discovery.**
- **Concurrency:** thousands of users on the same hotel at once without breaking.
- **Latency:** search **< 500 ms**; booking confirmation **2-3 s** acceptable.
- **Availability:** ~three nines (99.9%) uptime.
- **Scale horizontally** without redesigning later.

## 4.2 Capacity estimation `[58:39]`

Think of a **funnel**: all visitors (100%) -> ~10% browse/search -> ~10% of those actually book. Most visitors are tire-kickers but they still add **QPS**.

Two ways to estimate. Get **one** anchor number from the interviewer, then compute the rest:
- **Top-down:** monthly visits (e.g., ~500M/month from a traffic tool).
- **Bottom-up:** reservations per year (e.g., ~1.1B room-nights/year).

**Writes (transactions per second):**
```
1.1B room-nights/yr / 2 nights per booking = 550M bookings/yr
550M / 365 = ~1.5M bookings/day
1.5M / 86,400 s = ~17 bookings/s   => ~17 TPS
```

**Reads (queries per second):**
```
500M visits/month / 30 = ~17M visits/day
17M / 86,400 = ~200 visits/s
~5 backend reads per visit (search, availability checks; ~8 pages/visit)
200 x 5 = ~1,000 QPS
```
(The video says "divide by 40" but the result of 17M/day matches dividing by **30**.)

**Conclusion:** ~**1,000 QPS reads vs ~17 TPS writes** = strongly **read-heavy**. Optimize reads while keeping writes consistent to avoid double booking.

## 4.3 High-level architecture `[1:04:15]`

**Service-oriented architecture with clear boundaries, but tightly coupled data in the same database.**

```
 Web / Mobile --> CDN (images, JS, media)
      |
      v
 +-----------------------------------------------+
 | API Gateway: auth, rate limiting, routing, LB |
 +--+--------+--------+---------+-----------+-----+
    |        |        |         |           |
 Hotel    Reservation Rate    Payment   Notification
 service  service     service service   service
 (static, (booking,   (dynamic (Stripe,  (email, push)
  cacheable) inventory, pricing) PayPal,
             concurrency)        Apple/Google Pay)
    |        |                              ^
    +---+----+                              |
        v                                   | events
  PostgreSQL (ACID)  <-- Redis cache     Kafka / RabbitMQ
        |
        | CDC (Debezium -> Kafka)
        v
  Elasticsearch  <-- Search service  (GET /hotels, GET /search)
```

**Component roles**
- **CDN:** static assets, images, JS.
- **API gateway:** single entry point; authentication, rate limiting, routing, load balancing across service instances.
- **Hotel service:** hotel and room info; mostly static, highly cacheable.
- **Reservation service:** booking logic, inventory checks, create/cancel; **concurrency is handled here**.
- **Rate service:** price by date; applies **dynamic pricing** by demand.
- **Payment service:** integrates providers (Stripe, PayPal, Apple Pay, Google Pay).
- **Notification service:** emails and push; talks to reservation and payment services.
- **PostgreSQL:** ACID guarantees, transactions.
- **Redis:** cache for data that does not change often.
- **Search service + Elasticsearch:** PostgreSQL can't handle ~1,000 QPS of search well. ES filters by location (e.g., within 5 km), price range, ratings, and does full-text on names/descriptions. **Eventually consistent.**
- **Message queue (Kafka/RabbitMQ):** when payment succeeds, publish an event -> notifications go out AND availability updates flow to the search service -> ES updates (eventually). Right after payment, search may still show the hotel as available, but **booking is blocked because the booking path double-checks the DB.**

**Monolith vs microservices vs pragmatic `[1:09:56]`**
| Option | Pros | Cons |
|---|---|---|
| Single monolith | Simple transactions | Hard to scale/deploy independently with hundreds of developers |
| Full microservices, DB per service | Independent scaling/deploy | **Distributed transactions** and consistency problems |
| **Pragmatic (chosen):** service separation at app layer, **shared DB for tightly coupled data** (reservations + hotels) | **Local ACID transactions**, still modular | Shared schema coupling |

Optional: a **GraphQL layer** in front hides routing between many services from the front end. The video keeps direct REST calls for simplicity.

## 4.4 API design and idempotency `[1:11:08]`

**Hotels (admin: login required; users can browse without login but must sign in to reserve)**
| Action | Request | Notes |
|---|---|---|
| List/filter hotels | `GET /hotels?filters...` | Served from **Elasticsearch** |
| One hotel | `GET /hotels/{id}` | From **PostgreSQL** |
| Add hotel | `POST /hotels` | Insert SQL, return **201**; ES updated eventually |
| Update | `PUT/PATCH /hotels/{id}` | |
| Delete | `DELETE /hotels/{id}` | Removed from ES later |

**Rooms (sub-resource):** `GET /hotels/{id}/rooms`, `GET /hotels/{id}/rooms/{roomId}`, `POST /hotels/{id}/rooms`, `PUT /hotels/{id}/rooms/{roomId}`, `DELETE ...`.

**Reservations**
| Action | Request |
|---|---|
| History | `GET /reservations` |
| Details + invoice | `GET /reservations/{id}` |
| Create | `POST /reservations` |
| Cancel | `DELETE /reservations/{id}` (check cancellation policy, e.g., a week before is fine, last minute may not be) |
| Change dates | `PUT /reservations/{id}` |

**Search:** `GET /search?location=&checkin=&checkout=&guests=&beds=&price_min=&price_max=&filters...` -> queried from **Elasticsearch**, not the DB.

### Double-booking scenario 1: same user submits twice `[1:17:46]`

- Cause: user double-clicks submit.
- **Solution: client-generated reservation ID (UUID)** sent in the `POST /reservations` body.
  - Server checks if that reservation ID already exists.
  - **Not found:** insert, then publish a Kafka message (notify user and hotel owner).
  - **Found (duplicate):** return the **existing reservation** ("already reserved").
- Enforced with a **unique key** on reservation ID (duplicate-key error -> return existing).
- Alternative (backend hash of hotel + room + user + date) is more complex and can block legitimate future bookings.

> **➕ Added:** This is the **idempotency key** pattern (often sent as an `Idempotency-Key` header). Store key with result, return stored result on retry, expire after ~24h, and pass an idempotency key to the payment provider too.

Scenario 2 (two different users) is solved in [4.6](#46-concurrency-and-race-conditions-13308).

## 4.5 Data model `[1:22:35]`

**Key idea: you book a room type, not a specific room.** Room numbers are assigned at check-in.

```
hotels            (id PK, name, address, location, description)
                  -- brands: separate hotel IDs per legal entity, or a hotel_groups table
room_types        (id PK, name, description, capacity, hotel_id FK)      -- hotel 1:N room_types
room_type_rates   (room_type_id FK, package, rate)    -- basic 100, better view 150, breakfast 200 ...
rooms             (id PK, floor, available, room_type_id FK, hotel_id FK) -- physical room, assigned at check-in
room_type_inventory (hotel_id, room_type_id, date, total_inventory, total_reserved)
reservations      (id PK, room_type_id FK, hotel_id FK, guest_id FK,
                   start_date, end_date, status[reserved|cancelled])
guests            (id PK, first_name, last_name, email, ...)
admins/brokers    (users who publish hotels)
transactions      (id PK, reservation_id FK, type[new|change|refund], amount +/-)
```

**Why a separate `transactions` table?** One reservation can have **multiple transactions**: initial payment, a date change (price difference to pay or refund), or cancellation (negative amount = refund).

**`room_type_inventory` design**
- **One row per hotel + room type + date** (not per physical room).
- Enables **bulk availability updates**, scalable pricing/search, and **simple overbooking logic**.
- **Prepopulate the next 2 years**; a **daily cron job** adds new dates as time moves.

**Availability check (SQL)**
```sql
SELECT date, total_inventory, total_reserved
FROM room_type_inventory
WHERE hotel_id = ? AND room_type_id = ?
  AND date BETWEEN ? AND ?;
-- allow booking if: total_reserved + rooms_to_book <= total_inventory * overbooking_factor
```
Example: 100 inventory, 92 reserved, booking 1 -> 93 <= 100 OK. With **10% overbooking**, a 100-room inventory allows up to **110** bookings.

**Why relational:** need **ACID/consistency** (prevent double booking); read-heavy handled well and scaled further with Elasticsearch; clear **relationships**; complex joins and aggregations over inventory, pricing, dates.

## 4.6 Concurrency and race conditions `[1:33:08]`

**Scenario 2: two different users, one room left.** PostgreSQL default isolation is **read committed**. There is a **gap between the availability check and the write**. User A reads "1 left", user B also reads "1 left", both checks pass, both book -> **double booking**. So **read committed alone is unsafe for bookings.**

| Approach | How | Pros | Cons |
|---|---|---|---|
| **Serializable isolation** | Set isolation level to serializable when beginning the transaction | Simple; fine at **low TPS** | Transactions block each other at high TPS; **deadlocks** if rows are locked in different orders. Mitigate with lock timeouts and short transactions. |
| **Read committed + row locking** (`SELECT ... FOR UPDATE`) | Lock the inventory rows while checking and booking | Second transaction **waits its turn**; transactions touching **other rows run in parallel** | Must keep transactions short; lock rows in consistent order |

**Preferred:** read committed + `SELECT ... FOR UPDATE` row locking. Serializable is acceptable when TPS is low.

> **➕ Added:** Another common alternative is an **atomic conditional update**: `UPDATE room_type_inventory SET total_reserved = total_reserved + 1 WHERE ... AND total_reserved < total_inventory * factor`, then check affected rows equals number of nights. Also mention **optimistic locking** (version column + retry) for low-contention cases. Lock inventory rows in **date order** to avoid deadlocks on multi-night stays.

## 4.7 Sharding, caching and scaling `[1:34:44]`

**Sharding**
- **Shard key = `hotel_id`.** Most queries filter by hotel first. **All data for one hotel lives on the same shard**, so **no cross-shard joins** and **local transactions** within a shard.
- Initial approach **modulo sharding:** `shard = hash(hotel_id) mod N`. Example: **16 shards** -> ~**62 read QPS** and ~**1 TPS per shard** (from ~1,000 QPS / 17 TPS). Fine when shard count rarely changes.
- Problem: adding shard #17 with modulo **reshuffles almost all data**.
- **Consistent hashing:** hash ring where servers and keys map to points on a circle; each key goes to the **next server clockwise**. Adding a server only moves keys between it and its predecessor (~**1/17** of data).
- Rule: few shards and rarely changing -> **modulo**; frequently adding shards -> **consistent hashing**.

**Caching (Redis / Memcached)**
- Cache hot inventory.
- **Key:** `hotel_id + room_type_id + date`. **Value:** total inventory and total reserved. The **overbooking rate** is a fixed config, not stored in cache.
- **TTL ~5 minutes.**
- **Flow:**
  1. Search -> check Redis. **Hit:** show immediately.
  2. **Miss or expired:** read from DB, populate Redis, show.
  3. **Booking:** validate against the **DB (source of truth)**, update DB, **invalidate affected cache keys** so the next search is up to date.

## 4.8 Search at scale with Elasticsearch `[1:40:41]`

**What it is:** a search and analytics engine, specialized for search, **not** for transactions.

**Why not SQL `LIKE`:**
- `LIKE '%text%'` causes a **full table scan**, even with indexes.
- Cannot **rank by relevance**.
- Cannot handle **typos** (e.g., "Pari" vs "Paris").
- Poor at **geographic distance** queries.
- **Aggregations** are expensive.

**What ES gives you**
- **Full-text search with relevance ranking:** tokenize query ("luxury hotels in Paris"), find matching docs, rank.
- **Fuzzy matching** (edit distance of 1 catches missing letters).
- **Geo search:** e.g., hotels within 5 km, so a cheaper hotel in a nearby town can surface.
- Fast **filtering** (price 80-180, stars, location) across millions of docs.

**Data is duplicated and denormalized:** one JSON document per hotel with **nested** location and room types (no joins).

**Keeping ES in sync (CDC)** `[1:47:46]`
```
PostgreSQL (source of truth)
   -> Write-Ahead Log (WAL)
   -> Debezium (open-source CDC connector)
   -> Kafka (one topic per table)
   -> ES consumer updates the index
   => Elasticsearch is EVENTUALLY consistent
```
- **Transactions are validated against PostgreSQL**, not ES.
- The transaction log also helps with crash recovery.

## 4.9 Replication and global routing `[1:45:19]`

- **Primary (write) DB** replicated to **read replicas** in different **AZs/regions**. Users connect to the **closest** replica.
- **Read-your-writes:** use a flag for "recent write". If the user just wrote, read from the **primary** (replica may lag); otherwise read from the **nearest replica**.
- **Load balancing:** use **geo/latency-based** routing. Europe users mostly book Europe hotels, US users US hotels.
- **Regional** Redis caches and **regional** Elasticsearch clusters hold the data most relevant to that region.
- **Keep one primary shard** (e.g., shard 7 stays a single primary) to **avoid write conflicts between regions**. Since TPS is low, one primary shard + multiple regional replicas is enough.

## 4.10 Trade-offs to mention

| Decision | Trade-off |
|---|---|
| Strong consistency on bookings, eventual on search | Correctness where money is involved; search can show stale availability (re-checked at booking) |
| Shared DB for tightly coupled services | Local ACID transactions; less service independence |
| Row locks vs serializable | Parallelism for unrelated rows vs simplicity at low TPS |
| Modulo vs consistent hashing | Simplicity vs minimal data movement |
| Cache TTL ~5 min | Fewer DB reads; short staleness (booking still verifies DB) |
| Single primary | No cross-region write conflicts; extra latency for writes from far regions |

---

# Part 5: System 3, WhatsApp-Style Chat

*(Most in-depth case study.)* Forces you to handle **stateful connections, real-time delivery, consistency, scale, failures, disaster recovery, multi-device sync.** The plan: start simple (few thousand users), scale to billions, find where it breaks, fix it. `[1:50:45]`

## 5.1 Requirements `[1:54:36]`

**Functional**
- **One-on-one** messaging.
- **Group chats up to 100** participants.
- Delivery to **online and offline** users.
- **Media:** images, video, voice notes.
- **Message status:** sent, delivered, read.
- **Online/offline status** and **last seen**.

**Non-functional**
- **Low latency**: < **500 ms** when online.
- **Guaranteed delivery** (messages can't just disappear).
- Scale: **billions of users**, high throughput.
- **Don't store messages on servers longer than necessary.**
- **Resilient** when components fail (message must still eventually arrive).

## 5.2 Back-of-the-envelope `[1:56:23]`

| Metric | Value |
|---|---|
| Registered users | 1 billion |
| Daily active users (DAU) | 500 million |
| Peak concurrent connections | 50 million |
| Messages per user per day | 10-20 (use 20 max) |
| Messages per day | 500M x 20 = **10 billion** (= 10B writes/day) |
| Average rate | 10B / 86,400 = **~115,000 messages/s** |
| Peak (3-5x) | **350,000-500,000 writes/s** |
| Message size (with metadata) | ~1 KB |
| Storage per day | 10B x 1 KB = **10 TB/day** (**~3.6 PB/year**) |
| Peak bandwidth | 50M connections x 10 KB/s = **~500 GB/s** |

**Retention:** most messages are delivered and cleared within seconds. Users store history on devices and optionally back up (e.g., Google Drive). Requirements: **30-day retention for undelivered messages** and maybe ~**10%** of users keeping history on servers. This lets WhatsApp run on minimal infrastructure.

## 5.3 Architecture (start simple, add complexity as you scale) `[1:59:04]`

```
 Mobile app                                          Mobile app
 (WebSocket, local storage, retries)                  (receiver)
     |                                                    ^
     v                                                    |
 +--------+      +---------------+   route       +----------------+
 |  L4 LB |----->| Chat servers  |-------------->| Chat server(s) |
 +--------+      | (WebSocket)   |  via Redis    | of receiver    |
                 +---+-------+---+  pub/sub      +----------------+
        ack "sent"   |       |
                     |       +--> Redis: user connection cache
                     v            (user -> chat server, last activity)
              +--------------+
              | Message queue|  (Kafka / RabbitMQ)
              +------+-------+
                     v
            +------------------+      +-----------------------+
            | Message storage  |----->| NoSQL (Cassandra /    |
            | service          |      | DynamoDB)             |
            +------------------+      +-----------------------+

 Media:  App -> pre-signed URL -> Blob storage (S3/GCS) -> CDN
 Offline: Chat server -> Notification service -> APNs / FCM
 Presence: Presence service (heartbeats, Redis, publishes changes)
```

| Component | Role |
|---|---|
| **Mobile app (client)** | Maintains a persistent connection, handles UI, local message storage, **retries** on failures |
| **Load balancer** | Routes traffic across chat servers. **Layer 4** because of WebSockets |
| **Chat servers** | Hold **WebSocket** connections, route messages between users, track who's online, handle persistence flow |
| **Message queue (Kafka/RabbitMQ)** | **Decouples message writing from delivery**; helps with failures. Chat server **acks "sent" immediately**, then pushes async to the queue |
| **Message storage service** | Consumes from queue and persists to DB |
| **NoSQL DB (Cassandra / DynamoDB)** | **High write throughput**, simple query patterns (by user, conversation, timestamp) |
| **User connection cache (Redis)** | Which user is on which chat server, last activity. **Microseconds** vs milliseconds for DB, matters at scale |
| **Blob storage + CDN** | Media files; CDN caches popular files near users |
| **Notification service** | **APNs** (Apple) and **FCM** (Firebase/Android) push for offline users |
| **Presence service** | Heartbeats, updates Redis, publishes presence changes to interested subscribers |

### Communication method `[2:03:39]`

| Option | How | Problem |
|---|---|---|
| **Polling** | Client periodically asks "any new messages?" | Wastes bandwidth, adds latency (message arrives right after a poll = wait), huge resource waste at millions of users |
| **Long polling** | Server holds request until message or timeout | Constantly opening/closing connections, repeated HTTP handshakes, idle connections consume memory and thread slots |
| **WebSocket (chosen)** | One persistent **bi-directional** connection (HTTP handshake once, then persistent) | Needs **L4** load balancers; more memory per connection. Worth it for real-time |

### Service discovery `[2:07:40]`

- Chat servers crash, redeploy, scale up/down. The LB needs to know who is healthy.
- Use a **service registry** (Consul, AWS Cloud Map): servers **register** at boot; **health checks every ~10 s**; the LB **queries healthy servers** and updates its **routing table** (e.g., only servers 1 and 2, drop unhealthy server 3).

## 5.4 Data model and API flows `[2:09:28]`

**Tables**
```
users          (SQL)       id PK, phone UNIQUE, username, created_at, last_seen
messages       (Cassandra) message_id PK, sender_id, receiver_id, content, media_url,
                           status[sent|delivered|read], timestamp, deleted
groups         (Postgres)  id PK, name, created_at, created_by
group_members  (Postgres)  group_id, user_id, role, joined_at
connections    (Redis)     user_id -> server_id, connection_id, last_active, device_type
                           (refreshed on heartbeat, e.g., every ~300 s)
```

**WebSocket flows**
1. **Authenticate:** client sends user ID, token, device info. Server replies with auth success: **session ID, server time, online count**.
2. **Send message:** receiver, content, media, timestamp. Server **acks**.
3. **Receive:** server pushes message ID, sender, content, media. Client **acks** receipt.
4. **Read receipt:** client sends message ID + timestamp when opened. Server updates and forwards to the sender.
5. **Heartbeat** every ~5-10 s (both directions) so dead connections are closed (e.g., app or tab closed).

**Media upload via REST**
1. `POST /api/media` with file type, size, conversation.
2. Server returns **upload URL, media ID, expiry** (pre-signed).
3. Client uploads to the pre-signed URL (blob storage). Storage responds with success.
4. Client sends the chat message with the media reference. Server acks, or sends "message delivery failed" with an error.

## 5.5 Multi-device sync `[2:13:10]`

Example: user A has **3 devices** (phone, tablet, desktop), user B has **2** (phone, laptop).

1. A's phone sends to **chat server A**, which acks **"sent"**.
2. Chat server queries **Redis**: B has 2 devices connected to servers B and E; A's other devices are on servers C and D.
3. Servers C and D **display the sent message** on A's tablet and desktop (sync).
4. Server B delivers to B's phone, server E to B's laptop.
5. B's phone **acks** -> chat server B tells server A -> A sees **delivered**.
6. B reads -> **read receipt** -> server A updates to **read** -> server B marks it read on **B's laptop** as well.

**Conflicts:** **last-write-wins with version numbers** (not ideal, but simple).

**Optimization: incremental sync.** Don't send the full history to a new device. Send the **last ~30 days**, fetch older on demand (scroll up), archive very old messages in **cold storage** and load lazily.

**Trade-off:** roughly **doubles the delivery workload** (each message goes to multiple devices), but users expect it.

> **➕ Added:** In production, WhatsApp-like systems keep a **per-device mailbox/cursor**, so each device resumes from its last acknowledged message, and use **E2EE per device** (sender encrypts for each recipient device).

## 5.6 Core delivery flows `[2:20:16]`

**Both online**
1. A sends over WebSocket to chat server A.
2. Server **generates message ID** (unique, **timestamp-based**), **acks "sent"**.
3. Pushes message to the **message queue** for persistence.
4. Looks up B in the **Redis connection cache** (e.g., B is on server B).
5. Routes via **Redis pub/sub** to server B -> WebSocket to B -> B acks.
6. Server B notifies server A -> status **delivered**.
7. B opens message -> **read**. Optionally enqueue for analytics/archival.
8. Queue consumer writes to **Cassandra**.

**Recipient offline**
1. Same first steps. Redis says B is **offline**.
2. Add message to B's **inbox in Redis** and to the **inbox table in the DB**.
3. **Notification service** -> **APNs/FCM push**.
4. Enqueue -> persisted to DB.
5. When B comes online and connects to a chat server (e.g., C): **fetch pending from Redis inbox + any missed from DB inbox**, deliver, and notify A on read.

**Group messages (50-100 members)** `[2:25:07]`
1. A sends to the chat server with the group ID.
2. Chat server asks the **group service** for the member list.
3. Check each member's status via **Redis cache** (say 30 online, 20 offline).
4. For online users: look up their servers and **publish via Redis pub/sub** in parallel (e.g., 15 on server B, 15 on server C) -> WebSocket delivery -> acks forwarded back to sender's server.
5. For the **20 offline**: write **20 inbox records** + send **push notifications** via the notification service.
6. **Message stored once** (one write to queue -> DB). Status shows delivered to 30, pending for 20.

## 5.7 Media storage, pre-signed URLs, CDN `[2:28:56]`

**Wrong way:** upload to chat server -> store in DB -> recipient downloads from chat server. Chat servers become **bandwidth-limited** moving large blobs instead of routing messages. A server holding ~100K connections could be crushed by ~100 users uploading small files.

**Right way: pre-signed URLs**
1. A taps send -> app requests a **pre-signed upload URL** from the chat server.
2. Chat server asks blob storage (S3) to generate it and returns it.
3. A **uploads directly to S3**; S3 confirms and returns the media URL.
4. A sends the **chat message including the media URL**.
5. B receives the message + URL; on tapping, B requests a **pre-signed download URL**.
6. B downloads via **CDN** (scalable, replicated near users) **or directly from storage** (simplest).

**Trade-off:** CDN adds an extra request to get a URL and more complexity, but performance and cost benefits are **massive at this scale**.

- Chat servers only route **small messages and URLs**.
- URLs **expire** (TTL). **Media TTL = 30 days**, aligned with message retention, with automatic deletion.

## 5.8 Event-driven architecture and batch processing `[2:31:44]`

- Decouple using **Kafka**. Assign **partitions** to message-storage service instances (e.g., instance 1 gets partitions 0-9, instance 2 gets 10-19, ...). Stateless instances scale out.
- Each instance: **Kafka consumer -> batch write to DB cluster**. **Retry** on failure. If still failing, **do not delete from the queue**, or send to a **dead letter queue (DLQ)**.

**Detailed processing pipeline**
1. Batch size e.g. **500 messages**; consumer **polls every ~100 ms**; process with a **thread pool (e.g., 20 workers)**.
2. Per message: **validate schema**, metadata checks (e.g., file size limit), then **validate and enrich**:
   - compute **message hash for deduplication**
   - extract **@mentions**
   - detect **links for preview generation**
   - generate **indexes** for later queries
3. Queue validated messages into a **batch writer**. Accumulate e.g. **1,000 messages or every 5 seconds**; **sort and group by conversation**; prepare the batch.
4. **Atomic batch insert** into Cassandra. Update **search/analytics indexes** (index updater). Mark processed.
5. **Failures:** e.g., 10 of 1,000 fail -> **retry** (maybe a rate limit) -> if still failing, **DLQ** with escalating retry steps before notifying the user that delivery failed.

**Why batch:** a single insert ~**5 ms**. 1,000 individual inserts = **5 s**. One batched insert of 1,000 ~ **50 ms** = **~100x throughput**.

> **➕ Added (Cassandra caveat):** Batching helps most when a batch targets **one partition** (e.g., same conversation). Large **multi-partition logged batches** are an anti-pattern in real Cassandra. That's why grouping by conversation is the right instinct.

## 5.9 Encryption and rate limiting `[2:39:18]`

### End-to-end encryption (Signal protocol)

- Messages are encrypted on the sender's device and decrypted only on the recipient's. **Servers cannot read content.**
- **Flow**
  1. Each user generates an **identity key pair** at registration. **Public key** uploaded to the **key server**; **private key never leaves the device**.
  2. User B also uploads **prekeys** (e.g., ~100 signed prekeys); the server stores them for future initiators.
  3. When A wants to message B, A fetches B's **public identity key + one prekey**.
  4. A performs a **key exchange** to establish a **shared session key**.
  5. Messages are encrypted with session keys. A **message counter** advances with each message, so even a stolen session key can't easily decrypt other messages.

> **➕ Added (precision):** In the Signal protocol this is **X3DH** (key agreement using identity key + signed prekey + one-time prekey, works even if the recipient is **offline**) followed by the **Double Ratchet** (new key per message: **forward secrecy** and **post-compromise security**). Messages are encrypted with derived **symmetric** keys, not directly "with the public key". Group chats typically use **Sender Keys**. Trade-offs: server can't search or moderate content; metadata (who talks to whom, when) is still visible to the server.

### Rate limiting and abuse prevention `[2:45:00]`

Without limits, attackers can scrape data or DDoS. Apply limits **per action type**:

| Action | Example limit |
|---|---|
| Send messages | **100 messages/min** (token bucket) |
| Create groups | > **5 groups/hour** looks suspicious |
| Upload media | e.g., **10-100 files/min** |
| Connections | e.g., **5 different IPs in 1 minute** is suspicious (check IP connection counter) |
| Group invites | Group-invite counter |

(Numbers are examples; clarify in requirements.)

**Enforcement flow:** check limit -> **exceeded: block** (show "too many requests, try again in 60 s") **or require CAPTCHA** -> otherwise allow and write to DB.

**Progressive penalties:** warning -> **1-hour restriction** -> **24-hour restriction** -> **account suspended for human review** (contact user; tagged suspicious) -> repeated abuse = **permanent deletion**.

**Token bucket:** initialize with **capacity** and **refill rate** (different per action). Each request checks tokens: if **> 1**, allow and deduct one (return true); else return false and block. Refill based on time since **last refill**.

**Centralized store:** use **Redis** with **atomic operations** (data kept ~24 h), because users connect to different servers. This prevents **server-hopping** to bypass limits.

## 5.10 Sharding and cleanup jobs `[2:49:45]`

**Sharding options for messages**
| Shard by | Pros | Cons |
|---|---|---|
| **User ID** | All messages for a user on one shard, so reads hit one database | **Hotspots / imbalanced shards** (heavy users create huge shards) |
| **Message ID** | Even distribution | Reading a user's messages **hits all shards** (scatter-gather), slow and expensive |
| **Timestamp** | Alternative strategy | Similar trade-offs (understand them) |

**Good approach:** shard by **user ID** **plus duplicate writes**. If A (shard A) talks to B (shard B), write a copy into **both** shards.
- **Downside:** **doubles write volume**.
- **Upside:** **blazingly fast reads**; each user reads only their own shard.
- Add **replicas** per shard: users read from **replicas** (A reads replica A, B reads replica B), primaries handle writes.

**Rule:** **sharding scales writes; replication scales reads.**

**Replication details:** per shard, primary + e.g. **3 replicas** with **asynchronous** replication (**eventually consistent**). Read queries (opening chats, unread messages) go **only to replicas**; writes go to the **primary**.

**In the interview:** clarify early whether **reads or writes** are more important. Then pick: user-ID sharding + replication, message-ID sharding, or timestamp sharding. The key is stating the trade-offs.

**Cleanup jobs (garbage collection)** `[2:57:54]`
- Clean **expired media**, **old undelivered messages**, **stale connections**.
- **Job scheduler** (Airflow / cron; daily or weekly) -> **job queue** (separate from the message queue) -> **worker pool** (containerized executors) -> executes cleanup tasks (deletes expired media by date, removes undelivered messages that failed DB writes but linger in memory, stale connections) -> **submit results to monitoring**.
- **Trade-off:** adds operational complexity (another thing to monitor/debug), but the alternative is garbage accumulating until it forces you to do it anyway.

## 5.11 Failure handling and global scaling `[3:00:32]`

### Cache (Redis) failure
- Redis holds **undelivered-message inboxes**. If the Redis cluster dies?
- **Write path:** write undelivered messages to the **Redis inbox** (fast) **and** to the **database** (durable backup).
- **Read path:** try Redis; on failure, **log it** and **fall back to the DB** (slower but works). Either way the message is delivered.
- **Recovery:** a **background job** syncs from the DB to Redis (select inbox messages where not delivered), then normal Redis flow resumes.

### Regions and global distribution `[3:04:28]`
- Each **region** (e.g., US East, Singapore, Ireland) has a **full stack**: load balancer, chat servers, Redis clusters, DB shards, so it can operate independently.
- User in Tokyo: **DNS lookup** -> nearest region (e.g., Singapore) -> LB -> chat server (WebSocket).
- **Redis lookup** finds where user B is so the message routes to B's region; B (e.g., London) connects to the nearest region via DNS.
- **Capacity varies per region** (e.g., many chat servers in Ireland, fewer in Singapore).

### Kafka failure `[3:06:37]`
- Kafka scales well (topics split into **partitions**, each with a **leader** and **replicas**; producers write to leaders).
- **One broker fails:** replace and rejoin; Kafka **rebalances partitions**.
- **Entire cluster down (rare):** chat servers can't publish -> messages not persisted -> data loss risk. **Mitigation: local in-memory buffer** in the chat server.
  - `try publish to Kafka; catch -> log the error + push to local buffer`.
  - Buffer limited (e.g., **100,000 messages**); if it fills, **drop the oldest** rather than failing everything.
  - **Retry after an interval** longer than the time needed to spin up a new Kafka.
  - If the **chat server also fails** (both down), keep an **append-only log file on local disk**; **replay** it on recovery, **delete after successful publish**.
  - **Recovery:** request messages since last ID from the storage service, query processed messages in the DB, find which are still unpublished, **republish** them, then resume normal consumption.

## 5.12 Feature flags and rollouts `[3:09:08]`

- Usually you **don't release features to all users at once**; do **gradual rollout** using a **feature flag store** (LaunchDarkly, Azure/AWS native tools).
- Flags are **key-value pairs** (e.g., `new_theme_enabled = true/false`), controlled **per environment** (true on staging, false on production).
- **Gradual enablement:** e.g., **20% of users** get the beta feature; watch failure logs; if good, enable for everyone. Can be per user or per group.
- The **chat server checks** the flag store and returns enabled features. It can send new-feature messages to **client versions above e.g. 2.2** and **strip unsupported fields** for older clients.

> **➕ Added:** Also mention **kill switches**, **canary/ring deployments**, **dark launches (shadow traffic)**, backward-compatible protocol versions (old clients live for months), and **expand/contract DB migrations**. Clean up stale flags.

## 5.13 Trade-offs to mention

| Decision | Trade-off |
|---|---|
| WebSocket over polling | Real-time, efficient; more memory per connection, needs L4 LB |
| Queue between chat server and DB | Decoupling, failure handling, batching; added complexity and latency to persistence |
| NoSQL (Cassandra/DynamoDB) | Write throughput; limited joins/flexible queries |
| Redis connection cache | Microsecond routing; must handle Redis failure (DB fallback) |
| Duplicate writes + replicas | Fast reads; 2x write volume |
| Multi-device sync | Expected feature; roughly doubles delivery work |
| Pre-signed URL + CDN | Chat servers stay light; extra requests and complexity |
| Cleanup jobs | Keeps storage healthy; operational overhead |
| Local buffer + disk log for Kafka outage | Minimizes data loss; bounded memory and replay complexity |

---

# Part 6: Concepts Reference

> **➕ Added:** These are the building blocks the video uses repeatedly (and deeper explanations of the same). Keep this section for concept-round interviews.

## 6.1 Scaling and load balancing
- **Vertical vs horizontal:** bigger machine vs more machines. Horizontal needs **stateless** services.
- **L4 vs L7 LB:** L4 routes by IP/port (use for WebSockets/TCP); L7 routes by URL/header (path routing, TLS termination).
- **Algorithms:** round robin, weighted, **least connections**, IP hash, consistent hash. **Health checks** remove bad nodes.

## 6.2 Caching
- **Patterns:** cache-aside (most common), read-through, write-through, write-behind, refresh-ahead.
- **Eviction:** **LRU** (default), LFU, TTL.
- **Problems:** **stampede** (hot key expires, fix with single-flight / early refresh / jittered TTL), **penetration** (non-existent keys, fix with negative caching / Bloom filter), **avalanche** (many keys expire together, fix with randomized TTL), **hot key**.

## 6.3 Databases
| | SQL | NoSQL |
|---|---|---|
| Use for | Money, inventory, bookings, relational data, joins | Huge write throughput, simple access patterns (messages, events) |
| Gives | ACID, joins, strong schema | Horizontal scale, flexible schema |
- **Families:** key-value (Redis, DynamoDB), document (MongoDB), wide-column (Cassandra), search (Elasticsearch).
- **Indexes:** speed reads, slow writes. B-tree (read-optimized), LSM (write-optimized).

## 6.4 Replication, CAP, consistency
- **Leader-follower:** writes to leader, reads from followers; **replication lag** means stale reads. Use **read-your-writes** routing for recent writes.
- **CAP (video says "CP theorem"):** under a partition choose **Consistency** or **Availability**. Money/inventory favors **C**; search, presence, feeds favor **A**.
- **Quorum:** R + W > N for strong-ish reads.

## 6.5 Sharding
- **Range / hash / geo / directory.** **Modulo** is simple but reshuffles everything when N changes; **consistent hashing** moves only ~1/N.
- Pick a **shard key** that spreads load and keeps transactions single-shard. Beware **hot shards** and cross-shard queries.
- **Sharding scales writes; replication scales reads.**

## 6.6 Messaging
- **Queue** (task distribution) vs **log** (Kafka: replay, partitions, consumer groups, offsets).
- **Delivery semantics:** at-most-once, **at-least-once** (default), exactly-once (practically = at-least-once + **idempotent consumers**).
- **Patterns:** **DLQ**, retries with backoff, **outbox**, **CDC** (WAL -> Debezium -> Kafka), backpressure.

## 6.7 APIs and real time
- **REST** default; **gRPC** internal; **GraphQL** aggregation layer; **WebSocket** bidirectional; **SSE** one-way push; long polling fallback.
- **Pagination:** offset vs **cursor** (preferred for large/changing lists).
- **Idempotency key** for any side-effecting `POST`.

## 6.8 Blob storage and CDN
- **Object storage (S3):** cheap, durable, effectively unlimited. Flat namespace; "folders" are key prefixes.
- **Pre-signed/signed URLs:** clients upload/download directly; never proxy big files through app servers.
- **CDN:** caches near users; control with TTL and **invalidation**.

## 6.9 Search
- **Inverted index** (term -> documents), relevance ranking, fuzzy, geo. **Not source of truth**; sync via CDC; **eventually consistent**.

## 6.10 Rate limiting
| Algorithm | Note |
|---|---|
| **Token bucket** | Allows bursts; most common (used in the video) |
| Leaky bucket | Smooth constant output |
| Fixed window | Simple; boundary bursts |
| Sliding window (log/counter) | More accurate |
- Store counters in **Redis** (atomic). Return **429** + `Retry-After`.

## 6.11 Resilience and operations
- **Timeouts**, **retries with exponential backoff + jitter** (only for idempotent calls), **circuit breaker**, **bulkhead**, **graceful degradation** (shed non-critical features first).
- **Multi-AZ**, health checks, failover, RPO/RTO.
- **Observability:** metrics (p95/p99 latency, error rate, saturation), logs, traces, alerts on thresholds/SLO burn.
- **Deployment safety:** feature flags, canary, rollback.

---

# Part 7: Capacity Math Reference

> **➕ Added:** Numbers and templates to make estimation fast.

**Time:** 1 day = 86,400 s ~ 10^5 s. **Peak** = 2-5x average (video uses 3-5x).

| Requests/day | ~QPS |
|---|---|
| 1 million | ~12 |
| 10 million | ~120 |
| 100 million | ~1,200 |
| 1 billion | ~12,000 |

**Units:** KB 10^3, MB 10^6, GB 10^9, TB 10^12, PB 10^15 bytes.

**Latency (order of magnitude):** memory ~100 ns; SSD ~100 us; same-datacenter RTT ~0.5 ms; HDD seek ~10 ms; cross-continent RTT ~100-150 ms.

**Formulas**
```
avg QPS        = (DAU x actions/user/day) / 86,400
peak QPS       = avg x 2-5
storage        = events/day x bytes/event x 365 x years x replication (2-3)
bandwidth      = QPS x payload size
servers        = peak QPS / QPS-per-server (+ headroom)
```

**Template (say out loud):** assumptions -> avg and peak QPS -> storage (with replicas) -> bandwidth -> **conclusion** ("so audio dominates, we need CDN", or "so it's read-heavy, we need search + cache + replicas").

**Video's worked numbers to remember**
| System | Key numbers |
|---|---|
| Music (small) | 500K users, 30M songs, 3 MB/song -> 90 TB audio, 3 GB metadata, 0.5 GB users |
| Music (scaled) | 50M users, 200M songs -> 600 TB audio, 20 GB metadata, 50 GB users |
| Hotel | 1.1B room-nights/yr, 2 nights/booking -> ~17 TPS; 500M visits/month x 5 reads -> ~1,000 QPS; 16 shards -> ~62 QPS + ~1 TPS each |
| Chat | 500M DAU x 20 msgs = 10B/day = ~115K/s avg, 350-500K/s peak; 1 KB/msg = 10 TB/day = 3.6 PB/yr; 50M concurrent x 10 KB/s = 500 GB/s |

---

# Part 8: How to Actually Get Better `[3:11:14]`

From the video:
1. **Start simple.** Clarify requirements and estimate first; use the estimates to decide on storage, services, and whether you even need Kafka. Don't jump to Redis pub/sub or sharding for a few thousand users.
2. **Build it up naturally** by talking with the interviewer, then **identify bottlenecks** (Kafka can break, rate limiting issues, DB can go down).
3. **You can't memorize it.** Know the steps, then for each system walk the flow and **discuss trade-offs**.
4. **Every decision has costs.** WebSockets use more memory than HTTP but give real-time delivery. For another system, polling might be the better trade-off.
5. **Practice back-of-the-envelope calculations**; compute examples, then check them.
6. **Think about failures only after the main components exist.**
7. **You are not expected to design a perfect system.** Interviewers want to see how you think, discuss trade-offs, and whether you pick tools just because you like them.
8. **Say your thinking out loud.** The interviewer will correct you and give hints toward the right trade-off.

> **➕ Added (practice plan)**
> - 1-2 timed (45-60 min) designs per week using the 5 steps.
> - Redraw each system from memory 2-3 days after studying it.
> - Do mock interviews; ask for feedback on structure and trade-offs.
> - Build mini versions: rate limiter, WebSocket chat, job queue, URL shortener.
> - Read postmortems and engineering blogs; "Designing Data-Intensive Applications" for depth.
> - Be ready to explain your **own projects** through the framework (project walkthrough rounds).

---

# Part 9: Interview Questions and Answers

**Q1. How do you start any system design question?**
Clarify functional and non-functional requirements, define what's out of scope, confirm scale (DAU, read/write ratio, latency, availability), then ask "does this scope make sense?" before designing.

**Q2. What are the five things interviewers score?**
Communication/collaboration, structured thinking, technical judgment, practical depth, time management.

**Q3. How do you pick between SQL and NoSQL?**
By requirements. SQL for ACID, joins, relationships, consistency (payments, bookings, music metadata). NoSQL (Cassandra/DynamoDB) for massive write throughput and simple query patterns (messages). Always state why not the other.

**Q4. Why signed/pre-signed URLs for audio and media?**
Keep large files out of API/chat servers (bandwidth bottleneck), let clients talk directly to blob storage, and make links expire for security.

**Q5. Why a CDN?**
Lower latency and less origin load/egress cost by caching content near users (LRU eviction, fetch from origin on miss). Remember CDN invalidation when content changes.

**Q6. How do you scale the database?**
First read/write split (leader + followers). Then shard (geo, ID ranges, or by hotel/user key). Sharding scales writes, replication scales reads.

**Q7. Modulo sharding vs consistent hashing?**
Modulo is simple but adding a shard reshuffles almost all data. Consistent hashing (hash ring, next server clockwise) moves ~1/N. Use modulo for few, stable shards; consistent hashing when you add shards often.

**Q8. How do you prevent a user from double-booking by double-clicking?**
Client-generated reservation ID (UUID), unique key in DB; duplicate returns the existing reservation (idempotency).

**Q9. How do you prevent two users booking the last room?**
Don't rely on read committed alone. Use `SELECT ... FOR UPDATE` row locks (preferred) or serializable isolation (OK at low TPS). Keep a hotel's data on one shard so the transaction is local.

**Q10. Why Elasticsearch instead of SQL for search?**
`LIKE` forces full scans, no relevance ranking, no typo tolerance, poor geo and aggregations. ES has inverted indexes, ranking, fuzzy, geo queries. Keep PostgreSQL as source of truth and sync via CDC (WAL -> Debezium -> Kafka -> consumer); ES is eventually consistent.

**Q11. How do you handle stale search results?**
Accept eventual consistency for search; the booking path re-validates availability in the DB (and invalidates cache keys on booking).

**Q12. Why one inventory row per room type per date?**
Bulk availability updates, scalable pricing and search, simple overbooking logic (`total_reserved <= total_inventory x factor`).

**Q13. Why WebSockets for chat?**
Persistent bidirectional channel for real-time push; polling wastes bandwidth and adds latency; long polling churns connections. Trade-off: L4 LB and more memory per connection.

**Q14. How does a message reach an offline user?**
Redis says offline -> write to inbox (Redis + DB) -> push via APNs/FCM -> on reconnect, fetch pending from Redis inbox and missed from DB inbox -> deliver -> read receipt to sender.

**Q15. How do group messages work?**
Get members from group service, check statuses (Redis cache), publish to online users' servers in parallel via Redis pub/sub, write inbox + push for offline users. Message stored once.

**Q16. How do you handle multi-device sync?**
Redis tracks all devices/servers per user; fan out to each device; acks/read receipts propagate to the sender and sync across devices; last-write-wins with version numbers; incremental sync (last ~30 days, older on demand).

**Q17. Why batch writes to the database?**
One 1,000-message batch (~50 ms) vs 1,000 single inserts (~5 s) is ~100x throughput. Group by conversation; retry; DLQ on failure.

**Q18. How does E2EE work at a high level?**
Identity key pairs per user (private never leaves device), prekeys uploaded to key server, key exchange establishes session key, message counter/ratchet protects against key theft. Servers see only ciphertext.

**Q19. How do you design rate limiting?**
Token bucket per action type (messages, group creation, media, connections, invites), centralized in Redis with atomic ops, block or CAPTCHA, progressive penalties up to account review.

**Q20. How do you shard messages?**
Options: by user ID (hotspots), by message ID (scatter-gather reads), by timestamp. Good choice: user ID + duplicate writes + replicas (2x writes, very fast reads). Ask whether reads or writes matter more.

**Q21. What happens if Redis goes down?**
Write inbox to both Redis and DB; read from Redis, on failure fall back to DB; background job resyncs Redis from the DB after recovery.

**Q22. What happens if Kafka goes down?**
Replace failed brokers (rebalance). If the whole cluster is down: local buffer in chat servers (bounded, drop oldest), retry after interval, append-only disk log if the chat server dies too, republish missing messages after recovery.

**Q23. How do you roll out a new feature safely?**
Feature flag store, per-environment flags, gradual rollout (e.g., 20%), watch errors, then enable for all; handle old client versions by stripping unsupported fields.

**Q24. Monolith vs microservices?**
Monolith: simple transactions but hard to deploy independently at large team size. Full microservices: distributed transaction problems. Pragmatic: separate services at app layer, share DB for tightly coupled entities for local ACID.

**Q25. What do you cover in the last 5 minutes?**
Summarize key decisions and what you sacrificed (with why), then extensions: what you'd do with more time or 10x load.

---

# Part 10: Cheat Sheet

## 10.1 Framework in 30 seconds

| Step | Do | Time (60 min) |
|---|---|---|
| 1 | **Clarify requirements** (FR, NFR, out-of-scope) + estimates. Check in. | 5-10 |
| 2 | **Data model and API** (entities, keys, indexes, consistency; endpoints, pagination, errors, rate limit, versioning; DB choice + why) | 15-20 |
| 3 | **High-level design** (API servers + LB + DB + cache, user journey, read/write flows, TTL/invalidation, CAP) | ~15 |
| 4 | **Bottlenecks, scale, reliability** (SPOFs, fallbacks, multi-AZ, monitoring/alerts) | 5-10 |
| 5 | **Trade-offs and extensions** (decisions, sacrifices, 10x plan) | last few |

Steps 2 and 3 can be swapped.

## 10.2 The five scored pillars
**Communication** · **Structured thinking** · **Technical judgment (why X, not Y)** · **Practical depth (failures, scale)** · **Time management**

## 10.3 Always do / never do
- **Do:** clarify first · think out loud · treat interviewer as a colleague · suggest multiple approaches · explain trade-offs (even for obvious SQL) · check in after each phase.
- **Don't:** jump into solutions · stay silent · monologue · pick tech without justification · forget failures/monitoring.

## 10.4 Four default components
**API servers + Load balancer + Database + (Cache)**. Then add queues, CDN, blob storage, search as requirements demand.

## 10.5 Key numbers
```
1 day = 86,400 s   |  peak = 3-5x avg
Music:  30M songs x 3MB = 90 TB;  200M x 3MB = 600 TB;  metadata 100B/song
Hotel:  ~17 TPS writes vs ~1,000 QPS reads;  16 shards -> ~62 QPS + ~1 TPS each
Chat:   10B msgs/day = ~115K/s avg, 350-500K/s peak;  1KB/msg = 10 TB/day
        50M concurrent x 10KB/s = 500 GB/s;  batch insert 1000 = 50ms vs 5s (100x)
Latency: search <500ms | booking 2-3s | chat <500ms | memory ns, SSD us, DB ms
```

## 10.6 System 1: Music streaming (one screen)
- Client -> **LB** -> **stateless REST API (JWT)** -> **SQL** (metadata) + **blob storage** (audio, **signed URLs**, range requests for chunks).
- Upload: validate format/size -> blob -> extract metadata -> SQL -> hierarchical key -> **CDN invalidation**.
- Schema: users, playlists, playlist_items, songs, artists, artist_songs. **SQL because of joins.**
- Scale: **CDN per region (LRU)**, multi-region LBs, stateless API, **leader/follower DB**, shard by geo + artist ID ranges, regional blob replication.
- Insight: **audio dominates storage and bandwidth**; start simple.

## 10.7 System 2: Hotel booking (one screen)
- Read-heavy (~1,000 QPS vs ~17 TPS). Strong consistency for booking, eventual for search.
- Gateway -> hotel / reservation / rate / payment / notification services; **PostgreSQL** (shared DB for coupled data, local ACID); **Redis** (key = hotel+roomtype+date, TTL ~5 min); **Elasticsearch** via **CDC** (WAL -> Debezium -> Kafka); Kafka for payment/notification/availability events.
- **Inventory:** per hotel + room type + date (`total_inventory`, `total_reserved`); overbooking factor (e.g., 10%); prepopulate 2 years + daily cron.
- **Idempotency:** client-generated reservation UUID (same-user double click).
- **Concurrency:** read committed + `SELECT ... FOR UPDATE` (or serializable at low TPS).
- **Shard by hotel_id** (modulo for few shards, consistent hashing if adding shards often).
- **Replication:** primary + regional replicas; recent write -> primary; geo-latency LB; regional Redis/ES; **single primary** to avoid write conflicts.

## 10.8 System 3: WhatsApp-style chat (one screen)
- Client <-> **L4 LB** <-> **chat servers (WebSocket)**; **Redis** connection cache + pub/sub routing; **Kafka** -> message storage -> **Cassandra/DynamoDB**; **blob + CDN** for media (**pre-signed URLs**); **APNs/FCM** for offline; presence service (heartbeats).
- Flow: generate timestamp-based message ID -> ack **sent** -> queue -> route (Redis lookup) -> **delivered** -> **read**.
- Offline: inbox in **Redis + DB** -> push -> fetch on reconnect.
- Groups (<=100): group service -> parallel pub/sub to online servers, inbox + push for offline, **store once**.
- Multi-device: fan out to all devices, last-write-wins + versions, **incremental sync (30 days)**.
- Kafka consumers: partitions per instance, **batch 500 / write batch 1,000 or 5 s**, dedup hash, retry -> **DLQ**.
- **E2EE (Signal):** identity keys + prekeys on key server, session key, message counter/ratchet.
- **Rate limit:** token bucket per action in Redis, CAPTCHA, progressive penalties.
- **Sharding:** user ID + **duplicate writes** + replicas (2x writes, fast reads).
- **Cleanup:** scheduler -> job queue -> worker pool; TTL 30 days.
- **Failures:** Redis down -> DB fallback + resync job; Kafka down -> bounded local buffer + disk log + replay; regions with full stacks + DNS routing.
- **Feature flags:** per-env flags, 20% gradual rollout, strip fields for old clients.

## 10.9 Decision rules ("if X then Y")

| If you need... | Use... |
|---|---|
| Money / inventory / strong correctness | SQL, ACID, row locks |
| Massive write throughput, simple queries | Cassandra / DynamoDB |
| Fuzzy/geo/filter search | Elasticsearch (sync via CDC), never the source of truth |
| Large files | Blob storage + pre-signed URLs + CDN |
| Real-time bidirectional | WebSocket + L4 LB |
| Decouple writes from delivery, absorb spikes | Message queue (Kafka) + DLQ |
| Reduce DB reads | Cache (Redis) with TTL + invalidation on write |
| Scale reads / scale writes | Replicas / sharding |
| Add shards often | Consistent hashing |
| Few stable shards | Modulo |
| Safe retries / double-click | Client-generated idempotency key (UUID) |
| Abuse control | Token bucket in Redis |
| Privacy from the server | E2EE (Signal protocol) |
| Gradual release | Feature flags (percentage rollout) |
| Healthy server list for LB | Service registry + health checks |

## 10.10 Trade-off phrases
- "I'm choosing **A** over **B** because **R**; the downside is **D**, mitigated by **M**."
- "Strong consistency for bookings (non-negotiable); eventual consistency for search is fine because we re-validate in the DB."
- "Sharding scales writes; replication scales reads."
- "WebSockets use more memory than HTTP, but real-time delivery makes that worth it here."
- "Start simple, add Kafka/sharding only when the numbers justify it."

## 10.11 Red flags
No clarifying questions · no numbers · no "why" behind tech · ignoring failures/monitoring · SPOF left unaddressed · treating search index/cache as source of truth · monologue or silence · running out of time before scale and trade-offs.

## 10.12 Closing script (30 seconds)
> "To summarize: we scoped the system to A, B, C and assumed N users with X read/write traffic. The core flow is ... The critical correctness issue is ..., solved by ... We scale reads with ... and writes with ... Failures are handled by ... The main trade-offs were ..., and with more time or 10x load I'd add ... (monitoring, multi-region, recommendations, abuse detection)."

---

*End of handbook.*
