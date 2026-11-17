# Recon map: Sónar 2026 by Meetmaps (Android → clone as mobile web PWA)

Scope: festival-goer core — line-up, schedule by day and stage, personal
favourites / "my plan", reminders before a set, venue map, offline use.
Out of this slice: Sónar+D professional networking, ticketing.
For: the user's own event (single festival, one organiser).
Date: 2026-11-17

> **Recon limits.** The Google Play page, the App Store page and AppGoblin
> could not be fetched from this session (network allowlist). This map is
> built from web search results about the official app and from the standard
> patterns of festival apps. Rows marked `guess` must be checked against the
> real app on your own phone (screenshots go in `replica/screens/`, reference
> only). To fetch the store listings, allow `play.google.com`,
> `apps.apple.com` and `sonar.es` in the environment's network settings.

## Sources

| # | source | URL | notes |
| --- | --- | --- | --- |
| 1 | Play Store listing (target) | https://play.google.com/store/apps/details?id=com.meetmaps.sonar2026 | not reachable from session |
| 2 | Official app info page 2026 | https://sonar.es/en/practical-info/sonarapp | "real-time guide to Sónar and Sónar+D"; networking for +D ticket holders |
| 3 | App info page (2024 edition) | https://production-admin-portal.sonar.es/en/practical-info/sonarapp | stage maps for By Day / By Night, works offline |
| 4 | Sónar+D app page | https://sonar.es/en/practical-info/sonarapp-sonard | networking section (out of scope) |
| 5 | Meetmaps success story 2023 | https://welcome.meetmaps.com/en/success-stories/success-story-sonar-2023/ | personalised agenda built from favourites |
| 6 | Meetmaps blog | https://welcome.meetmaps.com/en/?p=18521 | vendor context |
| 7 | App Store listing (2018 id) | https://apps.apple.com/gb/app/s%C3%B3nar/id1378948981 | multilingual (EN, CA, ES, FR, DE, IT, PT, RU) |
| 8 | AppGoblin, previous package | https://appgoblin.info/apps/com.meetmaps.sonar2018 | ~16.7K installs, Meetmaps S.L. |
| 9 | Sónar 2026 venue press | https://xceed.me/blog/en/sonar-2026-one-venue-six-redesigned-stages/ | 2026: one venue, six stages |

## Core loop

A festival-goer opens the app, browses who plays when and where, stars the
sets they want, and gets their own timetable plus a reminder before each set
— even without mobile data.

## Screens

| ID | screen | route / how to reach | purpose | key components | states seen |
| --- | --- | --- | --- | --- | --- |
| S01 | Splash / language pick | first launch | choose language, load data for offline | logo, language list | first run, loading, offline-first-run error |
| S02 | Home | `/` tab bar | entry point: now playing, next up, news highlights | hero card, "now/next" list, quick links | filled, before-festival (countdown), during, after |
| S03 | Line-up (artists A–Z) | tab "Line-up" | browse all artists | search bar, filter chips (day, stage, genre), artist row | filled, no search results, loading |
| S04 | Artist detail | tap artist | bio, photo, set time, stage, links | header image, set card(s), favourite star, share | filled, artist with 2+ sets, no bio |
| S05 | Schedule / timetable | tab "Programme" | sets per day, by stage | day tabs, stage columns or list, time ruler, "now" line | filled, current-time highlight, clash markers, empty day |
| S06 | My plan (favourites) | tab "My Sónar" | personal timetable | same timetable filtered to favourites, clash warning | empty ("star sets to build your plan"), filled, clashes |
| S07 | Venue map | tab "Map" | find stages and services | zoomable map image, POI pins, legend, By Day / By Night toggle | filled, offline, POI selected |
| S08 | Stage detail | tap stage on map or in schedule | what plays on this stage | stage header, set list for day | filled, nothing now |
| S09 | Practical info | menu | hours, access, transport, FAQ, rules | accordion list, rich text | filled |
| S10 | News / updates | menu or home | last-minute changes, cancellations | news list, push-linked article | empty, filled |
| S11 | Settings | menu | language, notifications, reminder lead time | toggles, select | default |
| S12 | Notification permission prompt | after first favourite | ask to allow push | dialog | allowed, denied (fallback copy) |

States *guessed* for S05/S06 clash markers and S12 timing — confirm on device.

## Flows

```
F01 Plan my festival
    S02 home -> S03 line-up -> S04 artist -> star -> S06 my plan
    happy path clicks: 4 per artist (open tab, tap artist, star, back)
    edge: artist with two sets, set time changed after starring,
          two starred sets overlap, data not yet downloaded

F02 What's on now / next
    S02 home -> "now playing" card -> S08 stage
    happy path clicks: 1-2
    edge: between sets, festival not started, after closing, wrong device clock

F03 Get reminded before a set
    S04 star -> S12 allow notifications -> (push at T-15 min) -> S04 artist
    happy path clicks: 2 + push tap
    edge: permission denied, app killed, time zone of device ≠ Europe/Madrid,
          set cancelled (push must say so)

F04 Find a stage
    S05 schedule -> tap stage -> S07 map with stage highlighted
    happy path clicks: 2
    edge: offline, By Day vs By Night venue

F05 Browse by day and stage
    S05 schedule -> day tab -> scroll stages
    happy path clicks: 1-2
    edge: sets after midnight belong to the previous "festival day"

F06 Read a last-minute change
    push -> S10 news article -> linked S04 artist
    edge: offline, stale cache
```

## Components

| component | variants | states | used on |
| --- | --- | --- | --- |
| Bottom tab bar | 4–5 tabs | active, inactive, badge | all |
| Artist row | with photo, text only | default, favourited, pressed | S03, S06, S08 |
| Set card | compact (timetable), full (artist page) | upcoming, live now, finished, cancelled, clash | S04, S05, S06, S08 |
| Favourite star | icon toggle | off, on, animating | S03, S04, S05 |
| Day tabs | segmented | active, inactive, today | S05, S06 |
| Filter chips | day, stage, genre | on, off | S03 |
| Search bar | — | empty, typing, no results | S03 |
| Timetable grid | by stage columns, list | now-line, scroll-to-now | S05, S06 |
| Map viewer | pinch-zoom image + pins | POI selected, highlighted stage | S07 |
| Accordion | — | open, closed | S09 |
| Dialog | permission, confirm | — | S12 |
| Toast / banner | info, offline, updated | — | all |
| Countdown | — | before festival | S02 |

## Inferred data model

```
Event     id, name, edition_year, timezone (Europe/Madrid), start_date, end_date,
          day_cutoff_hour (e.g. 06:00, sets before belong to previous day)
          evidence: source 2, 9   confidence: high

Day       id, event_id, label (Thu/Fri/Sat), date, part (by_day | by_night)
          evidence: source 3 (By Day / By Night)   confidence: medium

Venue     id, event_id, name, map_image_url, part (by_day | by_night)
          evidence: source 3, 9   confidence: medium

Stage     id, venue_id, name, color, map_x, map_y, sort_order
          evidence: source 3, 9 (six stages in 2026)   confidence: high

Artist    id, event_id, name, slug, bio (per language), image_url, country,
          genres[], links {web, instagram, spotify, ...}
          evidence: standard line-up pages   confidence: high

Performance (Set)  id, artist_id, stage_id, day_id, start_at, end_at,
          status (scheduled | changed | cancelled), type (concert | dj | talk)
          evidence: schedule pattern   confidence: high

Poi       id, venue_id, kind (bar, wc, first_aid, exit, lockers, ...), name, map_x, map_y
          evidence: source 3 (maps)   confidence: guess

NewsItem  id, event_id, title, body (per language), published_at, linked_artist_id?
          evidence: "updated in real time" source 2   confidence: medium

InfoPage  id, event_id, title, body (per language), sort_order
          evidence: practical info section   confidence: medium

Favourite  device_id (or user_id), performance_id, created_at
          evidence: source 5   confidence: high — stored locally, no account (guess)

PushSubscription  device_id, endpoint, keys, language, reminder_minutes
          evidence: reminders pattern   confidence: guess
```

Relationships: Event 1-n Day, Event 1-n Venue, Venue 1-n Stage, Venue 1-n Poi,
Artist 1-n Performance, Stage 1-n Performance, Day 1-n Performance,
Device 1-n Favourite → Performance, Event 1-n NewsItem, Event 1-n InfoPage.

Translations: 8 languages on the original; for your own event start with 2–3
(e.g. FR + EN), stored per field.

## Feature matrix

See `features.csv`. Must: 13, should: 9, could: 6, skip: 5.

## Out of scope (cannot or should not be cloned)

- Sónar's line-up, artist bios, photos, logo, maps and texts — they are the
  festival's content. Your app uses **your event's** content.
- Sónar+D professional networking and meetings (separate slice, Meetmaps'
  core product).
- Ticketing / ticket wallet (partner ticketing system) — link out instead.
- Cashless / RFID wristband top-up (hardware + payment partner).
- Sponsor integrations and partner offers.

## Size

Screens 12, flows 6, entities 11. Hard parts:
1. **Offline-first PWA** — service worker cache of all data + map + images, and
   update without breaking the user's favourites.
2. **Web push reminders** — scheduled server-side per favourite; iOS only
   delivers web push once the PWA is installed to the home screen.
3. **Time logic** — festival days that run past midnight, Europe/Madrid vs
   device time zone, live "now" state, clash detection, set changes after a
   user starred them.

Size: **M** (a few weeks) for one event with an admin to enter the programme;
**S** if the programme is loaded from a JSON/CSV file and no admin UI is built.
