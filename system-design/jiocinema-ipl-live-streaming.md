# JioCinema - IPL Live Streaming

Podcast: [How JioCinema live streams IPL to 20 million concurrent devices w/ Prachi Sharma | Ep 7](https://www.youtube.com/watch?v=36N1Bz7qW0A) on [@AsliEngineering](https://www.youtube.com/@AsliEngineering).

### Notes

* Feature flags: P0, P1, P2.
  * P0 features must work - app open, streaming, monetization
  * P1 and P2 features can go through graceful degradation - failures should not be visible to customers - personalization.
  * In-house feature config service with several funnels, e.g., OS, platform, geography.
* Retry mechanism - exponential backoff.
* Request flow: client -> multi-CDN -> load balancer -> origin server -> DB.
  * Autoscaling is not enough in realtime scenarios, can take several minutes to trigger and finish.
  * Pre-scale instead based on back-of-the-envelope calculations.
    * API response as per Kubernetes pod config.
    * How many requests are likely to reach the origin?
* Brace-on-impact: spikes during the match based on interesting events.
  * Dhoni comes to play -> playback spikes
  * Dhoni is out -> homepage spikes.
  * Mechanisms:
    * Increase cache TTL
    * Keep a snapshot of pre-match API response in static storage and serve from there -> mitigation for panic mode situations.
* Keep buffers as per the match, e.g., MI vs CSK tends to have more traffic.
* No scaling down during the match, only after the match.
  * Ladder stepdown (number of customers).
* Multi-CDN optimizer - only for live streaming. Different CDNs have different capacities.
  * Tunings: cache TTL, cache offload < 90%, maximize the number of requests handled by CDN.
  * Backend decides which CDN to switch to - switch can be every few minutes to hours.
* Async processing - lots of Kafka with local storage as backup - view counter is backed by Kafka.
* User behaviour understanding is important - design decisions are based on this.
* Ad injection in live stream: short vs long ads.
  1. Static ad insertion: Every language commentary is a different stream - has one human director and operator. Operator inserts ads on director's cue. Can lead to overdelivery of ads.
  2. Personalized ad insertion - very difficult.
  3. Client-side ad insertion - based on [SCTE markers](https://en.wikipedia.org/wiki/SCTE-35) - [ABR](https://www.dacast.com/blog/adaptive-bitrate-streaming/) shifts can affect.
  4. Server-side ad insertion - form cohorts of people for targeted ads, cohorts can be based on factors such as geography.
* Interesting problems:
  * Image CDN got hot - because of IPL stickers.
    * Possible ways: [image sprite](https://www.w3schools.com/css/css_image_sprites.asp), base64 encoding. Sent base64 encoded images instead - already had those from WhatsApp stickers.
  * JioCinema app did not open due to stale DNS in the ISPs - mitigated from the backend - client switched to 8.8.8.8 for DNS resolution errors.
