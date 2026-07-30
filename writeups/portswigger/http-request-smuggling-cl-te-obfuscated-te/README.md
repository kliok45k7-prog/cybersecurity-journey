# HTTP Request Smuggling – CL.TE via Obfuscated TE Header

**Platform:** PortSwigger Web Security Academy  
**Vulnerability:** HTTP Request Smuggling (CL.TE)  
**Technique:** Obfuscated `Transfer-Encoding` header

---

## Summary

This lab demonstrates a classic **CL.TE** request smuggling vulnerability. The front-end and back-end servers disagree on how to parse an obfuscated `Transfer-Encoding` header. The front-end ignores the malformed header and uses `Content-Length`, while the back-end accepts it and treats the request as chunked. This desynchronization allows an attacker to smuggle a second request.

---

## Core Concept

- **Front-end (FE):** Sees the obfuscated `Transfer-Encoding` header as invalid and falls back to `Content-Length`.
- **Back-end (BE):** Accepts the same header and processes the body as `Transfer-Encoding: chunked`.

Because the two servers calculate the request length differently, leftover data from the first request is treated as the start of the next request.

---

## Detection

To detect this issue, send a request with a slightly malformed `Transfer-Encoding` header and observe differences in behavior.

Common obfuscation techniques:

- `Transfer-Encoding : chunked` (space before colon)
- `Transfer-Encoding:  chunked` (double space after colon)
- `Transfer-Encoding: xchunked`
- `Transfer-Encoding: chunked` (leading space)
- Adding a second `Transfer-Encoding` header after another header

---

## Exploit Payload

```http
POST / HTTP/1.1
Host: [LAB-ID].web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 60
Transfer-Encoding : chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

---

## Step-by-Step Solution

1. Capture a request to the home page and send it to **Repeater**.
2. Add the obfuscated header:  
   `Transfer-Encoding : chunked` (note the space before the colon).
3. Craft the body:
   - Start with the chunk size in hex (`5c`).
   - Follow it with the smuggled request (`GPOST ...`).
   - End with `0` to terminate the chunked body.
4. Ensure **Update Content-Length** is enabled in Repeater so the front-end receives the full body.
5. Send the request **twice**:
   - The first request poisons the back-end’s buffer.
   - The second request triggers the smuggled `GPOST` method.
6. A successful exploit returns an error such as **“Unrecognized method GPOST”**.

---

## Key Takeaways

- The front-end rejects the malformed `Transfer-Encoding` header and relies on `Content-Length`.
- The back-end normalizes the header and processes the request as chunked.
- The desynchronization causes the smuggled data to be prepended to the next request.
- Always test for differences in header parsing between front-end and back-end servers.

---

## Notes

This is a fundamental request smuggling technique. Understanding how each server interprets the same headers is critical when testing for desync vulnerabilities.
