# URL Shortener with Rate Limiting: Requirements

## 1. Problem statement

A service that converts long URLs into short links and redirects users to the original URL quickly. It must handle far more reads (redirects) than writes (link creation), and protect itself from abuse using rate limiting.

## 2. Functional requirements

### In v1

| ID | Requirement |
|----|-------------|
| FR1 | User can submit a long URL and get a short URL back |
| FR2 | Opening the short URL redirects to the original long URL |
| FR3 | The same short code must never point to two different URLs |
| FR4 | User can set an optional expiry time. Expired links return 410 Gone |
| FR5 | User can choose an optional custom alias (like `/my-link`). If it is taken, return 409 Conflict |
| FR6 | User can see click stats for a link (total clicks, last clicked time) |
| FR7 | The create-URL API is rate limited per client IP. When the limit is crossed, return 429 with a `Retry-After` header |
| FR8 | Invalid URLs are rejected with a clear error (400) |
| FR9 | User can delete a short link |

## 3. Non-functional requirements

| ID | Requirement | Why it matters |
|----|-------------|----------------|
| NFR1 | Redirect must be fast (target: under 50 ms with cache) | Redirect is the main feature, and users feel delay |
| NFR2 | Read-heavy: about 100 redirects for every 1 link created | This is why a Redis cache is added |
| NFR3 | Short codes must be unique even with many requests at the same time | A wrong redirect means a broken system |
| NFR4 | Highly available: redirects still work if analytics fails | Analytics is less important than redirect |
| NFR5 | Scalable: app can run as many copies, so no state is stored inside app memory | Rate limit counters and cache must live in Redis |
| NFR6 | Short code length: 7 characters, Base62 (a-z, A-Z, 0-9) | About 3.5 trillion combinations |
| NFR7 | Secure basics: validate input, block dangerous URLs | Prevents abuse |
| NFR8 | Easy to run: whole project starts with `docker-compose up` | Anyone can try it in minutes |

## 4. Out of scope :

- No frontend or UI (Swagger UI is our “UI”)
- No multi-region deployment
- No payments or paid plans
- No QR codes, no link previews
- No advanced analytics (location, device, referrer)
- No microservices split (one Spring Boot app is the right size here)
- No Kafka in v1

## 5. Assumptions :

- Anyone can create links without login in v1, and rate limiting is by client IP
- Long URLs can be up to 2048 characters
- Short links do not expire unless the user sets an expiry
- Redirect uses HTTP 302 (so every click reaches our server and can be counted)
