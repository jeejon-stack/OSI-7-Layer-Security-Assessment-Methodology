# OSI-7-Layer-Security-Assessment-Methodology

This assessment was assigned as part of Johnson's day-to-day role at Bincom Dev Center. It's published here to demonstrate the OSI-layer assessment methodology behind Kyvronix's security practice  Bincom is the employer and assigning organization for this work, not a Kyvronix client.

---

## Overview

This project walks the Bincom Mentoring Platform through every layer of the OSI model, from the moment a browser opens the page down to the network path packets take to reach it” using only what's visible to an anonymous, unauthenticated visitor. No credentials, no internal access, no source code: just a browser, its developer tools, and standard network reconnaissance (DNS lookups and a traceroute).

The goal was to answer a simple question: **what does the platform's security posture look like from the outside, before a user ever logs in?** The same layer-by-layer approach shown here is the one behind Kyvronix's security assessment practice.

## Methodology

- **Layers 7,6,5 (Application, Presentation, Session):** browser developer tools ” Network panel for requests/responses, Application panel for cookies and storage (Local Storage, Session Storage, IndexedDB).
- **Layer 4 (Transport):** TLS/QUIC handshake details surfaced via the browser's security panel and response headers.
- **Layer 3 (Network):** DNS resolution and a traceroute run from Nigeria to map the network path to the platform's edge.
- **Layers 2â€“1 (Data Link, Physical):** out of scope â€” not observable from an external, browser-based vantage point.

## Key Findings at a Glance

| Area | Observation |
|---|---|
| Transport security | HTTP/3 (QUIC) on UDP 443, TLS with **ML-KEM-768 post-quantum key exchange**, AES-128-GCM traffic encryption |
| Edge / CDN | Fronted by Cloudflare (Anycast IPs, `CF-Ray` / `CF-Cache-Status` headers); origin server fully masked |
| Session hygiene | No cookies, tokens, or auth headers set during anonymous browsing â€” nothing persisted client-side |
| Compression | Content negotiation across gzip, deflate, br, and zstd |
| Frontend | "Become a Mentor" / "Become a Mentee" buttons produced no visible navigation or network activity â€” flagged for follow-up |

---

## Layer 7 ” Application

Opening the platform triggers a standard `HTTPS GET` to the Bincom web server, which returns the page successfully. The browser then pulls in JavaScript bundles (e.g. `818c0.js`) via additional `GET` requests, each answered with `200 OK`, before the frontend initializes.


**Observation:** the **Become a Mentor** and **Become a Mentee** buttons didn't trigger any visible navigation, network request, or other client-side action during testing. No new entries appeared in the Network panel when they were clicked. This could point to a missing event handler, an unimplemented feature, or a client-side JavaScript issue â€” it needs a closer look at the browser console and the buttons' underlying HTML.


## Layer 6 ” Presentation

The connection runs over HTTPS on port 443, so TLS encryption is protecting data in transit. Response headers show:

```
Content-Type: text/html; charset=UTF-8
Content-Encoding: zstd
```

Zstandard (zstd) compression is applied server-side, and the browser decompresses it automatically before rendering ” a Presentation-layer optimization that keeps payloads small without altering the underlying content.


Content negotiation is also in play: requests advertise support for multiple encodings and the server picks whichever the browser supports, which helps both compatibility and performance.

```
Accept-Encoding: gzip, deflate, br, zstd
```



## Layer 5 ” Session

Inspecting an anonymous session end to end:

- No `Cookie` header on outgoing requests, and no `Set-Cookie` returned by the server.
- No `Authorization` header â€” protected session tokens aren't required to view the public platform.
- Local Storage, Session Storage, and IndexedDB were all clear: no auth tokens, session identifiers, or user data.
- Refreshing the page created no new client-side session artifacts.


**Takeaway:** the public-facing content is fully reachable without ever establishing an authenticated session, and the platform doesn't leave anything behind in the browser for anonymous visitors.

## Layer 4 ” Transport

This is where the platform stands out. Observed transport security includes:

- **HTTP/3 (QUIC)** over UDP port 443 ” advertised via `Alt-Svc: h3=":443"` and confirmed in the handshake.
- **TLS** with a valid, trusted certificate, establishing an authenticated encrypted connection.
- **ML-KEM-768 post-quantum key exchange** as part of the TLS handshake ” this is notable, since it means the key exchange is already designed to resist future quantum-computing attacks, well ahead of where most production platforms are today.
- **AES-128-GCM** encrypting all traffic after the handshake completes.
- **Cloudflare** acting as the edge proxy/CDN, confirmed via `CF-Ray` and `CF-Cache-Status` headers.
- No mixed-content issues ” every resource loads securely.


## Layer 3 ” Network

DNS resolution returns two public IPv4 addresses:

```
172.**.***.68
104.**.**.4
```

Both belong to Cloudflare's Anycast network, meaning the origin infrastructure sits behind Cloudflare's reverse proxy rather than being directly exposed. Response headers (`Server: cloudflare`, `CF-Ray`, `CF-Cache-Status`) confirm the same thing from the application side.


A traceroute from Nigeria showed traffic passing through the local ISP before reaching Cloudflare's autonomous system (AS13335) via the Amsterdam Internet Exchange (AMS-IX) ” geographically sensible routing to a nearby edge location. The origin server's real IP, host, and underlying infrastructure couldn't be determined, since Cloudflare masks it. No IPv6 addresses were observed.


## Layers 2 & 1 ” Data Link & Physical

Out of scope for this assessment ” not observable from an external, browser-based testing position.

---

## Conclusion

This end-to-end assessment used the OSI 7-layer model to evaluate the Bincom Mentoring Platform from an external, unauthenticated perspective. The platform uses HTTPS, HTTP/3, TLS with post-quantum (ML-KEM-768) key exchange, Cloudflare protection, and multi-format compressed delivery to provide secure, efficient communication ” a genuinely strong network and transport security posture for public access.

No authentication tokens, session cookies, or browser-stored session data were observed during anonymous use, confirming that public resources are reachable without establishing a session, and that nothing sensitive is left behind client-side.

**Limitation:** authenticated functionality (login, mentor/mentee dashboards, backend services) was outside the scope of this test and could not be evaluated. A complete assessment would need authenticated access to verify session management, authorization, and backend controls.

## Recommendations

1. **Investigate the non-responsive Mentor/Mentee buttons** ” check the browser console and the buttons' event handlers to confirm whether this is a bug, an incomplete feature, or intentional.
2. **Run a follow-up authenticated assessment** covering login flow, session/token management, authorization boundaries, and backend API security â€” none of which are visible from an anonymous, external vantage point.

---

*This assessment reflects only what's observable from an external, unauthenticated perspective as of July 24, 2026. It is not a substitute for an authenticated penetration test or a full application security review.*

---

## About Kyvronix

Kyvronix Cybersecurity Solutons Limited is a pan-African, AI-native cybersecurity practice founded by Johnson Oni. This write-up reflects the OSI-layer, black-box methodology used across Kyvronix's security assessment work.

