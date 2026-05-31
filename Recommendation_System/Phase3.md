# Phase 3: Notification System for Recommendations

## 1. Existing Recommendation APIs

We already have recommendation APIs.

```text
GET /api/recommendations/today
GET /api/recommendations/plan
GET /api/recommendations/topic-practice
GET /api/recommendations/weakness-plan
```

These APIs return recommended problems to the user.

The notification system will not replace these APIs.

The notification system will only tell the user:

```text
You have recommended problems available.
```

---

## 2. Basic Notification Flow

```text
User logs in
     ↓
System checks if recommendations exist
     ↓
If recommendations exist, create notification
     ↓
Frontend shows notification bell
```

Example:

```text
🔔 You have 5 new recommended problems
```

---

## 3. Two Types of Notification Delivery

There are two notification delivery cases.

```text
1. SSE Notification
2. Web Push Notification
```

---

## 4. SSE Notification

SSE means Server-Sent Events.

SSE is used when:

```text
User is logged in
AND
User browser tab is open
```

Example:

```text
User is using the website
     ↓
Backend creates notification
     ↓
Backend sends notification immediately through SSE
     ↓
Frontend receives notification without refreshing page
     ↓
Notification bell updates instantly
```

SSE is useful for real-time in-app notifications.

---

## 5. Web Push Notification

Web Push is used when:

```text
User is logged in
BUT
Browser tab is not open
```

Example:

```text
User allowed browser notifications
     ↓
User closes website tab
     ↓
Backend creates recommendation notification
     ↓
Backend sends Web Push notification
     ↓
User receives browser notification
```

Example browser notification:

```text
New problems recommended
You have 5 new LeetCode problems to practice.
```

---

## 6. Final Notification Delivery Logic

Whenever a recommendation notification is created:

```text
1. Store notification in database.

2. Check if user has active SSE connection.

3. If user is online through SSE:
   Send notification using SSE.

4. If user is not online through SSE:
   Send notification using Web Push.

5. If Web Push subscription does not exist:
   Only store notification in database.
```

Final flow:

```text
Recommendation exists
        ↓
Create notification in database
        ↓
Is user connected through SSE?
        ↓
Yes → send SSE notification
No  → send Web Push notification
```

---

# Database Setup

## 7. Create user_notifications Table

This table stores all notifications for users.

```sql
CREATE TABLE user_notifications (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,

    notification_type TEXT NOT NULL,
    title TEXT NOT NULL,
    message TEXT NOT NULL,

    status TEXT DEFAULT 'unread',

    channel TEXT DEFAULT 'in_app',

    metadata JSONB DEFAULT '{}'::jsonb,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    read_at TIMESTAMP,
    clicked_at TIMESTAMP,
    dismissed_at TIMESTAMP,
    expires_at TIMESTAMP
);
```

---

## 8. Explanation of user_notifications Table

### id

```sql
id SERIAL PRIMARY KEY
```

This is the unique ID of each notification.

Example:

```text
notification_id = 1
notification_id = 2
notification_id = 3
```

---

### user_id

```sql
user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE
```

This tells which user owns the notification.

Example:

```text
user_id = 10
```

Means the notification belongs to user 10.

`ON DELETE CASCADE` means if the user is deleted, all their notifications are also deleted.

---

### notification_type

```sql
notification_type TEXT NOT NULL
```

This tells what type of notification it is.

For recommendation notifications, possible values are:

```text
new_recommendations
pending_recommendations
weak_topic_practice
expiring_recommendations
inactive_recommendation_reminder
```

---

### title

```sql
title TEXT NOT NULL
```

This is the short heading of the notification.

Example:

```text
New problems recommended
```

---

### message

```sql
message TEXT NOT NULL
```

This is the full notification message.

Example:

```text
You have 5 new important LeetCode problems recommended for you.
```

---

### status

```sql
status TEXT DEFAULT 'unread'
```

This tells the current status of the notification.

Possible values:

```text
unread
read
clicked
dismissed
expired
```

Meaning:

| Status    | Meaning                                  |
| --------- | ---------------------------------------- |
| unread    | User has not seen the notification       |
| read      | User has opened or seen the notification |
| clicked   | User clicked the notification            |
| dismissed | User removed the notification            |
| expired   | Notification is old and no longer useful |

---

### channel

```sql
channel TEXT DEFAULT 'in_app'
```

This tells the notification channel.

For this system, possible values can be:

```text
in_app
sse
web_push
```

Meaning:

| Channel  | Meaning                                |
| -------- | -------------------------------------- |
| in_app   | Stored and shown inside app            |
| sse      | Sent live while tab is open            |
| web_push | Sent through browser push notification |

Important:

Even if notification is delivered using SSE or Web Push, it should still be stored in `user_notifications`.

---

### metadata

```sql
metadata JSONB DEFAULT '{}'::jsonb
```

This stores extra information.

Example:

```json
{
  "recommendation_count": 5,
  "redirect_to": "/recommendations/today"
}
```

This helps frontend know where to redirect user after click.

---

### created_at

```sql
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
```

Stores when notification was created.

---

### read_at

```sql
read_at TIMESTAMP
```

Stores when user read the notification.

---

### clicked_at

```sql
clicked_at TIMESTAMP
```

Stores when user clicked the notification.

---

### dismissed_at

```sql
dismissed_at TIMESTAMP
```

Stores when user dismissed the notification.

---

### expires_at

```sql
expires_at TIMESTAMP
```

Stores when notification should expire.

Example:

```text
Notification created today
Expires after 7 days
```

---

## 9. Create notification_events Table

This table stores notification history.

```sql
CREATE TABLE notification_events (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    notification_id INTEGER REFERENCES user_notifications(id) ON DELETE CASCADE,

    event_type TEXT NOT NULL,
    delivery_channel TEXT,

    metadata JSONB DEFAULT '{}'::jsonb,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 10. Purpose of notification_events Table

This table tracks what happened to each notification.

Example events:

```text
notification_created
notification_sent_sse
notification_sent_web_push
notification_read
notification_clicked
notification_dismissed
notification_expired
web_push_failed
```

Example:

| id | user_id | notification_id | event_type            | delivery_channel |
| -: | ------: | --------------: | --------------------- | ---------------- |
|  1 |      10 |               5 | notification_created  | in_app           |
|  2 |      10 |               5 | notification_sent_sse | sse              |
|  3 |      10 |               5 | notification_clicked  | sse              |

---

## 11. Create web_push_subscriptions Table

For Web Push, browser gives a subscription object.

You need to store that subscription in database.

```sql
CREATE TABLE web_push_subscriptions (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,

    endpoint TEXT NOT NULL,
    p256dh TEXT NOT NULL,
    auth TEXT NOT NULL,

    user_agent TEXT,
    is_active BOOLEAN DEFAULT true,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    UNIQUE (user_id, endpoint)
);
```

---

## 12. Explanation of web_push_subscriptions Table

### user_id

```sql
user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE
```

This tells which user owns this push subscription.

---

### endpoint

```sql
endpoint TEXT NOT NULL
```

This is the browser push endpoint.

It is provided by the browser when user allows notifications.

---

### p256dh

```sql
p256dh TEXT NOT NULL
```

This is a public encryption key from browser push subscription.

It is needed to send push notification securely.

---

### auth

```sql
auth TEXT NOT NULL
```

This is another secret key from browser push subscription.

It is also required for Web Push.

---

### user_agent

```sql
user_agent TEXT
```

This stores browser/device information.

Example:

```text
Chrome on Windows
Chrome on Android
Safari on iPhone
```

---

### is_active

```sql
is_active BOOLEAN DEFAULT true
```

If push fails permanently, mark subscription inactive.

---

## 13. Add Indexes

Indexes make notification queries faster.

```sql
CREATE INDEX idx_user_notifications_user_status
ON user_notifications(user_id, status);

CREATE INDEX idx_user_notifications_user_type_status
ON user_notifications(user_id, notification_type, status);

CREATE INDEX idx_user_notifications_expires_at
ON user_notifications(expires_at);

CREATE INDEX idx_notification_events_user_id
ON notification_events(user_id);

CREATE INDEX idx_notification_events_notification_id
ON notification_events(notification_id);

CREATE INDEX idx_web_push_subscriptions_user_active
ON web_push_subscriptions(user_id, is_active);
```

---

# Backend API Endpoints

## 14. Create Notifications Router

Create a new router:

```text
/api/notifications
```

Final endpoints:

```text
GET     /api/notifications
GET     /api/notifications/unread-count
PATCH   /api/notifications/read-all
PATCH   /api/notifications/{notification_id}/read
PATCH   /api/notifications/{notification_id}/click
PATCH   /api/notifications/{notification_id}/dismiss

GET     /api/notifications/stream

POST    /api/notifications/web-push/subscribe
DELETE  /api/notifications/web-push/unsubscribe

POST    /api/notifications/recommendations/check
```

---

## 15. Endpoint: GET /api/notifications

Purpose:

```text
Get active notifications for logged-in user.
```

SQL:

```sql
SELECT
    id AS notification_id,
    notification_type,
    title,
    message,
    status,
    channel,
    metadata,
    created_at,
    read_at,
    clicked_at,
    dismissed_at,
    expires_at
FROM user_notifications
WHERE user_id = :user_id
  AND status IN ('unread', 'read')
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  )
ORDER BY created_at DESC;
```

Frontend uses this to show notification dropdown.

---

## 16. Endpoint: GET /api/notifications/unread-count

Purpose:

```text
Get unread notification count for notification bell.
```

SQL:

```sql
SELECT COUNT(*) AS unread_count
FROM user_notifications
WHERE user_id = :user_id
  AND status = 'unread'
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  );
```

Frontend shows:

```text
🔔 3
```

---

## 17. Endpoint: PATCH /api/notifications/read-all

Purpose:

```text
Mark all notifications as read.
```

SQL:

```sql
UPDATE user_notifications
SET
    status = 'read',
    read_at = NOW()
WHERE user_id = :user_id
  AND status = 'unread'
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  );
```

Use this when user opens the notification dropdown.

---

## 18. Endpoint: PATCH /api/notifications/{notification_id}/read

Purpose:

```text
Mark one notification as read.
```

SQL:

```sql
UPDATE user_notifications
SET
    status = 'read',
    read_at = NOW()
WHERE id = :notification_id
  AND user_id = :user_id
  AND status = 'unread';
```

Then insert event:

```sql
INSERT INTO notification_events (
    user_id,
    notification_id,
    event_type,
    delivery_channel,
    metadata
)
VALUES (
    :user_id,
    :notification_id,
    'notification_read',
    'in_app',
    '{}'::jsonb
);
```

---

## 19. Endpoint: PATCH /api/notifications/{notification_id}/click

Purpose:

```text
Mark notification as clicked and return redirect path.
```

SQL:

```sql
UPDATE user_notifications
SET
    status = 'clicked',
    clicked_at = NOW()
WHERE id = :notification_id
  AND user_id = :user_id
RETURNING metadata;
```

Then insert event:

```sql
INSERT INTO notification_events (
    user_id,
    notification_id,
    event_type,
    delivery_channel,
    metadata
)
VALUES (
    :user_id,
    :notification_id,
    'notification_clicked',
    'in_app',
    '{}'::jsonb
);
```

Example returned metadata:

```json
{
  "redirect_to": "/recommendations/today",
  "recommendation_count": 5
}
```

Frontend redirects user to `redirect_to`.

---

## 20. Endpoint: PATCH /api/notifications/{notification_id}/dismiss

Purpose:

```text
Dismiss notification from notification list.
```

SQL:

```sql
UPDATE user_notifications
SET
    status = 'dismissed',
    dismissed_at = NOW()
WHERE id = :notification_id
  AND user_id = :user_id;
```

Then insert event:

```sql
INSERT INTO notification_events (
    user_id,
    notification_id,
    event_type,
    delivery_channel,
    metadata
)
VALUES (
    :user_id,
    :notification_id,
    'notification_dismissed',
    'in_app',
    '{}'::jsonb
);
```

---

# SSE System

## 21. Endpoint: GET /api/notifications/stream

Purpose:

```text
Open live SSE connection when user tab is open.
```

Frontend will call this after login.

```text
GET /api/notifications/stream
```

Flow:

```text
User logs in
     ↓
Frontend opens SSE connection
     ↓
Backend stores user's active connection in memory
     ↓
If notification is created while tab is open
     ↓
Backend sends notification through SSE
```

---

## 22. SSE Backend Connection Store

Backend should maintain an in-memory map:

```text
active_sse_connections = {
    user_id: connection
}
```

Example:

```text
user 10 has open tab
active_sse_connections[10] = response_stream
```

If user closes tab:

```text
Remove user_id from active_sse_connections
```

---

## 23. SSE Message Format

When backend sends SSE notification, send this data:

```json
{
  "type": "notification",
  "notification": {
    "notification_id": 15,
    "notification_type": "new_recommendations",
    "title": "New problems recommended",
    "message": "You have 5 new important LeetCode problems recommended for you.",
    "metadata": {
      "recommendation_count": 5,
      "redirect_to": "/recommendations/today"
    },
    "created_at": "2026-05-31T10:00:00"
  }
}
```

---

## 24. SSE Frontend Logic

After login:

```javascript
const eventSource = new EventSource("/api/notifications/stream");

eventSource.onmessage = function (event) {
  const data = JSON.parse(event.data);

  if (data.type === "notification") {
    showToast(data.notification.title, data.notification.message);
    updateNotificationBell();
    addNotificationToDropdown(data.notification);
  }
};
```

What happens here:

```text
1. Frontend opens SSE connection.
2. Backend sends notification.
3. Frontend receives notification immediately.
4. Frontend shows toast.
5. Frontend updates notification bell.
```

---

# Web Push System

## 25. Web Push Requirement

Web Push needs:

```text
1. Service worker on frontend
2. Browser notification permission
3. Push subscription from browser
4. Store subscription in backend
5. Backend sends push when tab is closed
```

---

## 26. Endpoint: POST /api/notifications/web-push/subscribe

Purpose:

```text
Store browser push subscription for logged-in user.
```

Frontend sends subscription object to backend.

Request body example:

```json
{
  "endpoint": "https://fcm.googleapis.com/fcm/send/abc",
  "keys": {
    "p256dh": "browser_public_key",
    "auth": "browser_auth_secret"
  }
}
```

Backend stores:

```text
endpoint
p256dh
auth
user_id
user_agent
```

SQL:

```sql
INSERT INTO web_push_subscriptions (
    user_id,
    endpoint,
    p256dh,
    auth,
    user_agent,
    is_active,
    updated_at
)
VALUES (
    :user_id,
    :endpoint,
    :p256dh,
    :auth,
    :user_agent,
    true,
    NOW()
)
ON CONFLICT (user_id, endpoint)
DO UPDATE SET
    p256dh = EXCLUDED.p256dh,
    auth = EXCLUDED.auth,
    user_agent = EXCLUDED.user_agent,
    is_active = true,
    updated_at = NOW();
```

---

## 27. Endpoint: DELETE /api/notifications/web-push/unsubscribe

Purpose:

```text
Disable browser push subscription.
```

SQL:

```sql
UPDATE web_push_subscriptions
SET
    is_active = false,
    updated_at = NOW()
WHERE user_id = :user_id
  AND endpoint = :endpoint;
```

---

## 28. Web Push Frontend Service Worker

Create service worker file:

```text
/public/service-worker.js
```

Service worker handles push notification.

Example:

```javascript
self.addEventListener("push", function (event) {
  const data = event.data ? event.data.json() : {};

  const title = data.title || "New notification";

  const options = {
    body: data.message || "",
    data: data.metadata || {}
  };

  event.waitUntil(
    self.registration.showNotification(title, options)
  );
});

self.addEventListener("notificationclick", function (event) {
  event.notification.close();

  const redirectTo = event.notification.data.redirect_to || "/";

  event.waitUntil(
    clients.openWindow(redirectTo)
  );
});
```

---

## 29. Web Push Frontend Subscription Flow

After user logs in:

```text
1. Register service worker.
2. Ask notification permission.
3. Subscribe user to push manager.
4. Send subscription to backend.
```

Pseudo-code:

```javascript
async function setupWebPush() {
  const registration = await navigator.serviceWorker.register("/service-worker.js");

  const permission = await Notification.requestPermission();

  if (permission !== "granted") {
    return;
  }

  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: VAPID_PUBLIC_KEY
  });

  await fetch("/api/notifications/web-push/subscribe", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(subscription)
  });
}
```

---

## 30. Backend Web Push Send Logic

When user is not connected through SSE, backend sends Web Push.

Steps:

```text
1. Get active web push subscriptions for user.
2. Send push payload to each subscription.
3. If push fails permanently, mark subscription inactive.
4. Insert notification event.
```

SQL to get subscriptions:

```sql
SELECT
    id,
    endpoint,
    p256dh,
    auth
FROM web_push_subscriptions
WHERE user_id = :user_id
  AND is_active = true;
```

Payload:

```json
{
  "title": "New problems recommended",
  "message": "You have 5 new important LeetCode problems recommended for you.",
  "metadata": {
    "notification_id": 15,
    "redirect_to": "/recommendations/today",
    "recommendation_count": 5
  }
}
```

---

# Recommendation Notification Logic

## 31. Endpoint: POST /api/notifications/recommendations/check

Purpose:

```text
Check if recommendation notifications should be created.
```

This endpoint should check:

```text
1. New recommendations
2. Pending recommendations
3. Weak topic recommendations
4. Expiring recommendations
5. Inactive recommendation reminder
```

For first version, start only with:

```text
new_recommendations
```

---

## 32. Check New Recommendations

SQL:

```sql
SELECT COUNT(*) AS pending_count
FROM user_problem_recommendations
WHERE user_id = :user_id
  AND status = 'pending'
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  );
```

Meaning:

```text
Count recommendations that are generated but not shown yet.
```

If count is greater than `0`, create notification.

---

## 33. Avoid Duplicate Notification

Before creating notification, check if same unread notification already exists.

```sql
SELECT id
FROM user_notifications
WHERE user_id = :user_id
  AND notification_type = 'new_recommendations'
  AND status = 'unread'
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  )
LIMIT 1;
```

If this returns a row:

```text
Do not create duplicate notification.
```

If no row is returned:

```text
Create notification.
```

---

## 34. Create New Recommendation Notification

```sql
INSERT INTO user_notifications (
    user_id,
    notification_type,
    title,
    message,
    status,
    channel,
    metadata,
    expires_at
)
VALUES (
    :user_id,
    'new_recommendations',
    'New problems recommended',
    'You have new important LeetCode problems recommended for you.',
    'unread',
    'in_app',
    jsonb_build_object(
        'recommendation_count', :pending_count,
        'redirect_to', '/recommendations/today'
    ),
    NOW() + INTERVAL '7 days'
)
RETURNING id;
```

Then insert event:

```sql
INSERT INTO notification_events (
    user_id,
    notification_id,
    event_type,
    delivery_channel,
    metadata
)
VALUES (
    :user_id,
    :notification_id,
    'notification_created',
    'in_app',
    jsonb_build_object(
        'source', 'recommendation_system'
    )
);
```

---

## 35. Deliver Notification After Creating It

After creating notification:

```text
Check if user has active SSE connection.
```

### Case 1: User is online

```text
If active_sse_connections has user_id:
    send notification using SSE
    insert notification_sent_sse event
```

Event:

```sql
INSERT INTO notification_events (
    user_id,
    notification_id,
    event_type,
    delivery_channel,
    metadata
)
VALUES (
    :user_id,
    :notification_id,
    'notification_sent_sse',
    'sse',
    '{}'::jsonb
);
```

---

### Case 2: User is offline

```text
If active_sse_connections does not have user_id:
    send notification using Web Push
    insert notification_sent_web_push event
```

Event:

```sql
INSERT INTO notification_events (
    user_id,
    notification_id,
    event_type,
    delivery_channel,
    metadata
)
VALUES (
    :user_id,
    :notification_id,
    'notification_sent_web_push',
    'web_push',
    '{}'::jsonb
);
```

---

### Case 3: User has no Web Push subscription

```text
If user is not connected through SSE
AND user has no web push subscription:
    do not send live notification
    only keep notification stored in database
```

User will see it next time they open the app.

---

# Notification Types for Recommendations

## 36. Notification Type 1: new_recommendations

When:

```text
User has pending recommendations.
```

SQL:

```sql
SELECT COUNT(*) AS count
FROM user_problem_recommendations
WHERE user_id = :user_id
  AND status = 'pending'
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  );
```

Notification:

```text
Title: New problems recommended
Message: You have new important LeetCode problems recommended for you.
Redirect: /recommendations/today
```

---

## 37. Notification Type 2: weak_topic_practice

When:

```text
User has high-score recommendations from weak topics.
```

SQL:

```sql
SELECT COUNT(*) AS count
FROM user_problem_recommendations
WHERE user_id = :user_id
  AND status IN ('pending', 'shown', 'clicked')
  AND score >= 110
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  );
```

Notification:

```text
Title: Practice your weak topics
Message: You have recommended problems from your weak topics.
Redirect: /recommendations/topic-practice
```

---

## 38. Notification Type 3: expiring_recommendations

When:

```text
Recommendations expire within 24 hours.
```

SQL:

```sql
SELECT COUNT(*) AS count
FROM user_problem_recommendations
WHERE user_id = :user_id
  AND status IN ('pending', 'shown', 'clicked')
  AND expires_at IS NOT NULL
  AND expires_at > NOW()
  AND expires_at <= NOW() + INTERVAL '24 hours';
```

Notification:

```text
Title: Recommendations expiring soon
Message: Some of your recommended problems will expire soon.
Redirect: /recommendations/today
```

---

## 39. Notification Type 4: inactive_recommendation_reminder

When:

```text
User has not clicked shown recommendations for 2 days.
```

SQL:

```sql
SELECT COUNT(*) AS count
FROM user_problem_recommendations
WHERE user_id = :user_id
  AND status = 'shown'
  AND clicked_at IS NULL
  AND recommended_at <= NOW() - INTERVAL '2 days'
  AND (
      expires_at IS NULL
      OR expires_at > NOW()
  );
```

Notification:

```text
Title: Continue your practice
Message: You have not opened your recommended problems yet.
Redirect: /recommendations/today
```

---

# Service Layer

## 40. Create Notification Service

Create a backend service file:

```text
notification_service.py
```

This service should have these functions:

```text
create_notification()
create_notification_event()
check_duplicate_notification()
get_unread_count()
expire_old_notifications()
deliver_notification()
send_sse_notification()
send_web_push_notification()
check_recommendation_notifications()
```

---

## 41. create_notification()

Purpose:

```text
Insert notification into user_notifications table.
```

Input:

```text
user_id
notification_type
title
message
metadata
expires_at
```

Output:

```text
created notification
```

---

## 42. check_duplicate_notification()

Purpose:

```text
Prevent same unread notification from being created again.
```

It checks:

```text
same user_id
same notification_type
status = unread
not expired
```

---

## 43. deliver_notification()

Purpose:

```text
Decide whether to send SSE or Web Push.
```

Logic:

```text
if user has active SSE connection:
    send_sse_notification()
else:
    send_web_push_notification()
```

---

## 44. send_sse_notification()

Purpose:

```text
Send notification instantly to open browser tab.
```

Used when:

```text
user is logged in
tab is open
SSE connection exists
```

---

## 45. send_web_push_notification()

Purpose:

```text
Send browser push notification.
```

Used when:

```text
user is logged in
tab is not open
web push subscription exists
```

---

## 46. check_recommendation_notifications()

Purpose:

```text
Check recommendation table and create notifications if needed.
```

Logic:

```text
1. Check new recommendations.
2. Avoid duplicate notification.
3. Create notification.
4. Deliver notification using SSE or Web Push.
```

---

# Frontend Flow

## 47. On User Login

After user logs in:

```text
1. Open SSE connection.
2. Register service worker.
3. Ask notification permission.
4. Subscribe for Web Push.
5. Send subscription to backend.
6. Check recommendation notifications.
7. Fetch unread count.
8. Fetch notifications.
```

---

## 48. Frontend Login Flow

```text
User logs in
     ↓
Frontend calls GET /api/notifications/stream
     ↓
Frontend registers service worker
     ↓
Frontend asks notification permission
     ↓
Frontend sends push subscription to backend
     ↓
Frontend calls POST /api/notifications/recommendations/check
     ↓
Frontend calls GET /api/notifications/unread-count
     ↓
Frontend shows notification bell
```

---

## 49. When Notification Arrives by SSE

```text
Backend sends SSE event
     ↓
Frontend receives event
     ↓
Show toast
     ↓
Update notification bell count
     ↓
Add notification to dropdown
```

---

## 50. When Notification Arrives by Web Push

```text
Backend sends Web Push
     ↓
Browser service worker receives push
     ↓
Browser shows notification
     ↓
User clicks notification
     ↓
Service worker opens recommendation page
```

---

# Final Complete Flow

## 51. Complete End-to-End Flow

```text
1. User logs in.

2. Frontend opens SSE connection:
   GET /api/notifications/stream

3. Frontend registers Web Push subscription:
   POST /api/notifications/web-push/subscribe

4. Frontend checks recommendation notifications:
   POST /api/notifications/recommendations/check

5. Backend checks user_problem_recommendations.

6. If pending recommendations exist:
   Backend creates row in user_notifications.

7. Backend creates notification_created event.

8. Backend checks active SSE connection.

9. If user tab is open:
   Backend sends notification through SSE.

10. If user tab is not open:
   Backend sends notification through Web Push.

11. Frontend shows notification bell.

12. User opens notification dropdown:
   PATCH /api/notifications/read-all

13. User clicks notification:
   PATCH /api/notifications/{notification_id}/click

14. Frontend redirects user to:
   /recommendations/today

15. Frontend loads recommendation data using:
   GET /api/recommendations/today
```

---

# Final New Endpoints

## 52. Add These Notification Endpoints

```text
Notifications
The notifications router is mounted with prefix /api/notifications.

Method    Endpoint                                           Purpose
GET       /api/notifications                                 Get active notifications
GET       /api/notifications/unread-count                    Get unread notification count
PATCH     /api/notifications/read-all                        Mark all notifications as read
PATCH     /api/notifications/{notification_id}/read           Mark one notification as read
PATCH     /api/notifications/{notification_id}/click          Mark notification as clicked
PATCH     /api/notifications/{notification_id}/dismiss        Dismiss notification

GET       /api/notifications/stream                          Open SSE notification stream

POST      /api/notifications/web-push/subscribe              Save Web Push subscription
DELETE    /api/notifications/web-push/unsubscribe            Disable Web Push subscription

POST      /api/notifications/recommendations/check           Check and create recommendation notifications
```

---

# Build Order

## 53. Step-by-Step Build Order

Build in this order:

```text
1. Create user_notifications table.

2. Create notification_events table.

3. Create web_push_subscriptions table.

4. Add indexes.

5. Create notification service.

6. Create basic notification APIs:
   GET /api/notifications
   GET /api/notifications/unread-count
   PATCH /api/notifications/read-all
   PATCH /api/notifications/{notification_id}/click

7. Create recommendation check API:
   POST /api/notifications/recommendations/check

8. Add duplicate notification check.

9. Add SSE endpoint:
   GET /api/notifications/stream

10. Add active SSE connection storage.

11. Add SSE delivery function.

12. Add Web Push subscription APIs.

13. Add service worker on frontend.

14. Add Web Push delivery function.

15. Connect notification creation with delivery:
    SSE if tab open,
    Web Push if tab closed.

16. Add notification bell in frontend.

17. Add click redirect to recommendation pages.
```

---

# Simplest Version to Build First

## 54. Version 1

Start with only:

```text
1. user_notifications table
2. notification_events table
3. GET /api/notifications
4. GET /api/notifications/unread-count
5. POST /api/notifications/recommendations/check
6. SSE endpoint
```

Only support:

```text
new_recommendations
```

---

## 55. Version 2

Then add:

```text
1. Web Push subscription table
2. Web Push subscribe API
3. Service worker
4. Web Push delivery
```

---

## 56. Version 3

Then add more notification types:

```text
weak_topic_practice
expiring_recommendations
inactive_recommendation_reminder
```

---

# Final Summary

Phase 3 notification system does this:

```text
Phase 1:
Standard important problems are stored.

Phase 2:
Recommended problems are generated.

Phase 3:
User is notified about those recommendations.
```

Delivery logic:

```text
If user tab is open:
    Send SSE notification.

If user tab is closed:
    Send Web Push notification.

Always:
    Store notification in database.
```
