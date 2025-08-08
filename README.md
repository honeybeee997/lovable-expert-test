## Overview

This document outlines the major bugs that were discovered and resolved in the
Scheduling App

---

## Critical Fixes Implemented

### 1. Timezone Display Errors in Appointments

**File**: `src/utils/dateFormatter.ts`
**Severity**: Critical
**Status**: Fixed

#### Problem

Appointment times displayed inconsistently across users in different timezones. A
booking at 4:00 PM EST appeared as 1:00 PM PST, leading to:

- Missed meetings
- Confusion in customer support
- Frustration and loss of trust

#### Root Cause

Timestamps were rendered using `new Date().toLocaleString()` without setting a
consistent server-side timezone.

#### Fix

Replaced local formatting with a UTC-standardized formatter using `date-fns-tz`:

```typescript
format(utcToZonedTime(appointmentTimeUTC, userTimeZone), 'hh:mm a zzz');
```

#### Impact

- ✅ Accurate appointment display in all user timezones
- ✅ Fewer missed appointments
- ✅ Time consistency across platforms

---

### 2. Duplicate Bookings on Retry

**File**: `src/hooks/useBookAppointment.ts`
**Severity**: High
**Status**: ✅ Fixed

#### Problem

Users experiencing network issues and retrying caused **duplicate bookings**, which
cluttered the database and overbooked slots.

#### Root Cause

No idempotency token was implemented to recognize retries of the same booking.

#### Fix

Introduced a unique `x-request-id` in each booking attempt and deduplicated on the
server:

```typescript
// Frontend
axios.post('/api/book', payload, {
  headers: {'x-request-id': uuidv4()},
});
```

```ts
// Backend
if (hasAlreadyProcessed(requestId)) return;
```

---
