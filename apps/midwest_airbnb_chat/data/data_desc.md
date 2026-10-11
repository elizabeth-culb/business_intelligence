# Midwest Airbnb Listings: Data Dictionary

**Dataset:** `listings` table in `midwest_airbnb.db` (SQLite), 14,887 rows and 29 columns
**Source:** Inside Airbnb (https://insideairbnb.com/get-the-data/), the detailed `listings.csv.gz` file for each of three regions: Chicago (snapshot 2026-07-20), Columbus (snapshot 2026-07-23), and Twin Cities MSA (snapshot 2026-07-21). Column meanings follow Inside Airbnb's data dictionary and assumptions (https://insideairbnb.com/data-assumptions/).
**Course:** ISA 401, Miami University

> One row is one listing that showed a nightly price on the snapshot date; listings with no price were dropped. Empty cells are stored as SQL `NULL`.

---

## Field Definitions

| Field | Type | Description |
|---|---|---|
| `city` | text | Which Inside Airbnb region the listing came from: `Chicago` (7,439 rows), `Columbus` (2,587), or `Twin Cities` (4,861). The Twin Cities file covers the Minneapolis-St. Paul metro area, not just the two cities. |
| `snapshot_date` | text | Date Inside Airbnb compiled the file, stored as an ISO text string, not a date: `2026-07-20` for Chicago, `2026-07-23` for Columbus, `2026-07-21` for Twin Cities. Every row of a city shares the same value. |
| `id` | text | Airbnb's listing id. Unique across the table (14,887 distinct values). Stored as text even though it looks numeric, so compare it to a quoted string. |
| `name` | text | Listing title as shown on Airbnb (for example "Tiny Studio Apartment 94 Walk Score"). Never empty. |
| `price` | real | Nightly price in U.S. dollars on the snapshot date, with the dollar sign and commas removed. Ranges from 2.56 to 11,412; never `NULL` (rows without a price were dropped). |
| `room_type` | text | Airbnb's four listing categories: `Entire home/apt` (11,652 rows), `Private room` (2,951), `Hotel room` (246), or `Shared room` (38). |
| `host_id` | text |	Airbnb's unique identifier for the listing's host. One host can own several listings, so this value repeats across rows — cross-reference with host_name to see a host's full portfolio. [confirm distinct count vs. total rows — a big gap tells you how concentrated hosting is]
| `host_name` | text | The host's first name (or team/business name) as shown on their public profile. Not unique or verified — self-reported, and multiple hosts can share a name. [check for NULLs]
| `host_since` | text | Date the host's Airbnb account was created, stored as an ISO text string (e.g. 2015-03-02), not a date type. Reflects account age, not when they started hosting this specific listing.
| `host_is_superhost` | text | Whether the host held Superhost status on the snapshot date. Airbnb's raw scrape stores this as the single characters t/f, not a true boolean — check whether your import converted it. [confirm actual stored values]
| `neighbourhood` | text |This is Inside Airbnb's neighbourhood_cleansed field, not the host's free-text neighbourhood. Inside Airbnb assigns it itself by plotting each listing's latitude/longitude against official city/metro boundary files, so values are standardized and complete, unlike the free-text version (which is often empty).
| `latitude` | real | Geographic coordinate of the listing, published by Airbnb. Airbnb jitters these slightly for guest privacy before a booking is confirmed, so treat them as approximate, not a street address.
| `longitude` | real | Same caveat as latitude; use the pair together for mapping, not alone.
| `property_type` | text | A finer-grained category than room_type — e.g. "Entire rental unit", "Private room in home", "Entire condo." Airbnb maintains dozens of labels, so expect far more than 4 distinct values. [run COUNT(DISTINCT ...)]
| `accommodates` | integer | Maximum guest count the host sets for the listing. Required by Airbnb, so it should never be NULL.
| `bedrooms` | integer | Host-entered bedroom count. Can be NULL — common for studio-style listings where Airbnb doesn't force an entry.
| `beds` | integer | Host-entered bed count, independent of bedrooms (a studio can have 2 beds, 0 separate bedrooms). Can also be NULL.
| `bathrooms_text` | text | Free-text bathroom description, e.g. "1 bath", "1.5 shared baths", "0 baths" — Airbnb replaced the old numeric bathrooms column with this. You'd need to parse out the leading number to use it quantitatively.
| `minimum_nights` | integer | Shortest stay the host allowed on the snapshot date. Hosts change this over time, so it's a point-in-time policy, not a fixed rule.
| `availability_365` | integer | Of the 365 days following the snapshot, how many the calendar showed open (not booked or host-blocked). A listing can show 0 because the host paused hosting, not because of low demand — don't read it as a popularity score on its own.
| `number_of_reviews` | integer | Total reviews ever received as of the snapshot. 0 is valid and common for new listings.
| `number_of_reviews_ltm` | integer | Reviews in the 12 months before the snapshot ("ltm" = last twelve months). A better short-term activity signal than the lifetime total.
| `first_review	text`	| text | Date of the listing's earliest review, ISO text string. NULL for listings with zero reviews.
| `last_review	text` | text | Date of the most recent review, same format and same NULL rule as first_review.
| `review_scores_rating` | real | Average overall guest rating. Airbnb's scale is effectively compressed near the top (most ratings cluster at 4.8–5.0 of 5, i.e. ~95–100 on the internal 100-point scale), so small differences are noisy. NULL for unreviewed listings.
| `reviews_per_month` | real | Calculated by Inside Airbnb, not scraped directly: total reviews divided by the number of months the listing has been active. Used as a rough stand-in for booking frequency, since most guests never leave a review. NULL for zero-review listings.
| `instant_bookable	text`	| text | Whether a guest can book without host approval. Raw Airbnb flag, stored as t/f. [confirm stored format]
| `estimated_revenue_l365d` | real | Calculated by Inside Airbnb: roughly price × an estimated occupancy figure, where that occupancy estimate is itself built mostly from number_of_reviews_ltm. Treat it as a model, not verified income — it systematically undercounts bookings where guests skip leaving a review. NULL where Inside Airbnb couldn't compute it.
| `amenities_count` | integer | Not an Inside Airbnb column. Computed for this course as the number of items in each listing's raw amenities list. Depends entirely on how thoroughly the host filled out the form, not on independent verification — two identical units can differ here just because one host listed more amenities.

Continue the table for the remaining 23 columns (Assignment 05): `host_id`, `host_name`, `host_since`, `host_is_superhost`, `neighbourhood`, `latitude`, `longitude`, `property_type`, `accommodates`, `bedrooms`, `beds`, `bathrooms_text`, `minimum_nights`, `availability_365`, `number_of_reviews`, `number_of_reviews_ltm`, `first_review`, `last_review`, `review_scores_rating`, `reviews_per_month`, `instant_bookable`, `estimated_revenue_l365d`, `amenities_count`.

Two hints: `neighbourhood` is Inside Airbnb's `neighbourhood_cleansed` column, and `amenities_count` is not an Inside Airbnb column; it was computed for this course as the number of items in each listing's `amenities` list. Everything else keeps its Inside Airbnb name, so the data dictionary linked above explains it.
