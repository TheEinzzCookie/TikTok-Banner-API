# TikTok Profile Banner API

This document explains the architecture, flow, and endpoints used by TikTok to handle user profile covers (banners) based on internal API handlers and reverse-engineered client logic.
You need a Singer to use these API endpoints.

---

## Architecture & Message Handling

In the TikTok Android client, profile synchronization and state management are handled by the core handler class `C0OWJ`[cite: 1].

* **Message Type `10`**: When the client receives a profile update event, action code `10` specifically designates **Profile Cover / Banner** updates[cite: 1].
* **Response Processing**: Upon receiving a response, the client processes `UserResponse` and extracts cover image locations using `user.getCoverUrls()`[cite: 1].

---

## Two-Step Execution Flow

Updating or assigning a profile banner consists of two sequential operations:

### 1. Image Upload Phase
Raw image files cannot be directly attached in a single profile commit request. They must first be ingested by TikTok's storage infrastructure.

* **Purpose**: Upload the raw binary image data to TikTok's storage servers.
* **Result**: The server stores the image on ByteDance Cloud Storage (TOS) and returns a unique resource identifier known as a **`cover_uri`** (e.g., `tos-maliva-p-0068/...`).

### 2. Profile Commit Phase
Once the `cover_uri` is generated, the user's profile state must be updated to reference the new asset.

* **Endpoint**: The primary update route is `/aweme/v1/commit/user/`[cite: 1].
* **Parameters**: The request maps key-value parameters including the newly acquired `cover_uri` and context metadata such as `page_from`[cite: 1].
* **Result**: The server updates the user's record and links the cover image to the profile[cite: 1].

---

## API Variations: Mobile vs. Web

### Mobile API (`api-va.tiktokv.com`)
* Primary mobile endpoint defined in internal app presenters (`C351010DpN`)[cite: 1].
* Protected by proprietary client-side signature headers (`X-Gorgon`, `X-Khronos`, `X-MS-STUB`).
* Unsigned HTTP requests sent to these endpoints are rejected by the server (e.g., `status_code: 7`, `Permission Denied`).

### Web API (`www.tiktok.com`)
* Web-facing endpoints (e.g., `/api/v1/web/project/post/image/upload/`).
* Relies on standard browser authentication mechanisms using session cookies (`sessionid`) and anti-CSRF tokens (`passport_csrf_token` passed via the `X-Secsdk-Csrf-Token` header).
* Bypasses proprietary mobile signature requirements when accompanied by standard web parameters (`aid`, `device_id`, and standard HTTP browser headers).
