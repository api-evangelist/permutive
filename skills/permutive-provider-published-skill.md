---
name: Permutive
description: Use when building audience segmentation, identity resolution, and real-time activation workflows. Reach for this skill when implementing SDKs on web/mobile/CTV properties, creating cohorts, managing identities, tracking events, or integrating with ad platforms and data warehouses.
metadata:
    mintlify-proj: permutive
    version: "1.0"
---

# Permutive Skill

## Product summary

Permutive is a predictive data collaboration platform that enables publishers to build, activate, and monetize first-party audiences in real-time. It provides edge-based processing for 100% audience addressability across browsers, devices, and privacy-safe environments. Agents use Permutive to deploy SDKs (JavaScript, iOS, Android, CTV), track events, resolve identities, build cohorts, and activate audiences to ad platforms and data warehouses. Key files: Permutive Dashboard (https://dash.permutive.com), API keys in Settings › Keys. Primary docs: https://docs.permutive.com

## When to use

Reach for this skill when:
- **Deploying SDKs**: Adding Permutive to web, mobile, or CTV properties for event tracking and cohort evaluation
- **Building cohorts**: Creating audience segments via the dashboard or Cohorts API based on behavioral, contextual, or imported data
- **Managing identity**: Configuring identifiers, calling identify() to resolve users across sessions/devices, or understanding identity resolution
- **Tracking events**: Implementing event schemas, tracking pageviews/custom events, or validating event data
- **Activating audiences**: Connecting cohorts to ad servers (GAM, Xandr, FreeWheel), SSPs (Prebid, Magnite), DSPs, or data warehouses
- **Integrating APIs**: Using Events, Identity, Contextual, Segmentation, or Cohorts APIs for real-time data processing
- **Troubleshooting**: Debugging SDK deployment, event tracking, identity resolution, or API authentication issues

## Quick reference

### SDK Deployment

| Platform | Key File | Installation | Initialization |
|----------|----------|---------------|-----------------|
| Web (JavaScript) | `live.js` | Script tag in `<head>` | `permutive.addon('web', { page: {...} })` |
| iOS | CocoaPods/SPM | `pod 'Permutive'` | `Permutive(context, workspaceId, apiKey)` |
| Android | Gradle | `implementation 'com.permutive:...'` | `Permutive(context, workspaceId, apiKey)` |
| CTV (tvOS/Roku/Android TV) | Platform-specific | See CTV docs | Platform-specific initialization |

### API Authentication

All APIs require authentication via API key (UUID format):
- **Header**: `X-API-Key: <your-api-key>`
- **Query param**: `?k=<your-api-key>`
- **Key types**: Public (client-side, SDKs) and Private (server-side, account changes)
- **Location**: Permutive Dashboard › Settings › Keys

### Core API Endpoints

| API | Purpose | Typical Use |
|-----|---------|------------|
| Events API | Track user actions | `POST /events` to capture pageviews, custom events |
| Identity API | Resolve user identities | `POST /identify` to link identifiers across sessions |
| Contextual API | Get page classifications | `GET /contextual/segment` for real-time content tagging |
| Segmentation API (CCS) | Evaluate cohort membership | `POST /segmentation` for real-time user segmentation |
| Cohorts API | Manage cohorts programmatically | CRUD operations on cohorts |
| Taxonomy API | Manage second-party data | Create/update/delete segments in imported taxonomies |

### Common SDK Methods (JavaScript)

```javascript
permutive.identify([{ tag: 'email_sha256', id: 'hash', priority: 0 }])
permutive.track('EventName', { property: 'value' })
permutive.addon('web', { page: { type: 'article', article: {...} } })
permutive.segment(cohortId, callback)  // Check single cohort
permutive.segments(callback)  // Get all cohorts
permutive.trigger(cohortId, param, handler)  // React to cohort entry/exit
permutive.consent({ opt_in: true, token: '...' })  // Set consent
permutive.ready(callback, 'realtime')  // Wait for SDK ready
```

### Event Schema Validation

- Events must match workspace schema (defined in dashboard)
- Event properties limited to 950 KB
- Permutive infers schema on first event of new type
- Use dashboard Events section to view/manage schemas

### Cohort Types

| Type | Use Case | Real-time |
|------|----------|-----------|
| Custom | Behavioral rules (recency, frequency, properties) | Yes (edge-computed) |
| Contextual | Page content classification (keywords, categories) | Yes (contextual API) |
| Standard | Pre-built IAB taxonomy segments | Yes (Watson-powered) |
| Lookalike | Modeled audiences from seed cohorts | Yes |
| Classification | Predictive models trained on user behavior | Yes |

## Decision guidance

### When to use SDK vs API Direct

| Scenario | Use SDK | Use API Direct |
|----------|---------|----------------|
| Web/mobile/CTV property with user sessions | ✓ | |
| Server-side event tracking only | | ✓ |
| Real-time cohort evaluation on-device | ✓ | |
| Batch event ingestion from data warehouse | | ✓ |
| Need automatic pageview/engagement tracking | ✓ | |

### When to use Public vs Private API Key

| Use Case | Key Type |
|----------|----------|
| SDK deployment (visible in page source) | Public |
| Creating/updating cohorts | Private |
| Managing account settings | Private |
| Identify endpoint (SDK) | Public |
| Retrieve identities endpoint | Private |

### Cohort Activation: Dashboard vs API

| Task | Dashboard | API |
|------|-----------|-----|
| Create one-off cohort | ✓ | |
| Bulk create/update cohorts | | ✓ |
| Manual audience testing | ✓ | |
| Programmatic cohort management | | ✓ |
| Ad server integration setup | ✓ | |

## Workflow

### 1. Plan and Configure

1. **Define event schema** with Technical Services (pageviews, custom events, properties)
2. **Identify identifier types** (first-party IDs, third-party IDs, device IDs)
3. **Configure identifiers** in dashboard: Settings › Identity › Add Identifier
4. **Set up consent handling** (GDPR opt-in, CCPA opt-out, or none)
5. **Obtain credentials**: Workspace ID, API Key, Organization ID from dashboard

### 2. Deploy SDK

1. **Add loader script** to `<head>` with workspace ID and API key
2. **Load live.js** async script from edge
3. **Call `permutive.identify()`** if user is authenticated
4. **Initialize web addon** with page metadata: `permutive.addon('web', { page: {...} })`
5. **Verify deployment** using Chrome extension or debug logging (`?__permutive.loggingEnabled=true`)
6. **Check dashboard** Events section for incoming data (allow 5-10 min for processing)

### 3. Build Cohorts

1. **Navigate** to Custom Cohorts in dashboard
2. **Click Add Cohort** and name it
3. **Select event** (Pageview, custom event, etc.)
4. **Define rules**: properties, recency (time window), frequency (min occurrences)
5. **Add conditions** with AND/OR logic
6. **Calculate audience size** to estimate reach
7. **Save and deploy** cohort (live within minutes)

### 4. Activate Cohorts

1. **Choose destination**: Ad server (GAM, Xandr), SSP (Prebid, Magnite), DSP, or data warehouse
2. **Configure integration** in Settings › Integrations
3. **Map cohorts** to platform-specific audience IDs
4. **Test activation** by checking ad server/SSP for cohort signals
5. **Monitor** cohort size and membership in dashboard

### 5. Verify and Monitor

1. **Check event ingestion**: Dashboard › Events, filter by event name and date
2. **Validate cohort membership**: Use Chrome extension or API to check user cohorts
3. **Monitor audience size**: Compare Predicted Audience Size (PAS) vs Live Audience Size (LAS)
4. **Review activation logs**: Confirm cohorts flowing to ad platforms
5. **Enable debug logging** if issues: `?__permutive.loggingEnabled=true` (web) or SDK logging (mobile)

## Common gotchas

- **API key mismatch**: Verify API key matches workspace in dashboard. Public keys are visible in page source; private keys must be kept secret.
- **Workspace ID vs Project ID**: Use `workspaceId` (UUID), not deprecated `projectId`. Check SDK initialization for correct parameter name.
- **Identity tags not configured**: Identities passed to `identify()` must be in the dashboard allow-list (Settings › Identity). Unconfigured tags are silently ignored.
- **Consent blocking all tracking**: With `consentRequired: true`, SDK waits for `permutive.consent({ opt_in: true })` before tracking. Events called before consent are discarded, not queued.
- **Event schema mismatch**: Events not matching workspace schema are rejected with 400 error. Use dashboard to view/update schemas; contact support to migrate.
- **Cohort membership delay**: Live Audience Size (LAS) updates every ~4 hours. Predicted Audience Size (PAS) is historical estimate; don't rely on it for real-time targeting.
- **Identity resolution not merging**: Ensure identifiers are consistent across sessions/devices. Different identifier values for same user create duplicate profiles. Use same priority order across all platforms.
- **URL trailing slashes in cohorts**: SDK sanitizes URLs (removes trailing `/`). When building cohorts with `client.url contains`, omit trailing slash.
- **Commas in property values**: Escape commas with backslash in cohort builder: `\$20\,000` not `$20,000`.
- **SDK not loading**: Verify loader script runs before `live.js`. Check Network tab for script load errors. Ensure no CSP blocking.
- **Segments returning empty**: Use `permutive.ready(callback, 'realtime')` to wait for cohort data. Default `ready()` fires before real-time data loads.
- **Third-party cookie blocking**: SDK uses first-party cookies and localStorage, not third-party cookies. If tracking fails, check localStorage is enabled.
- **Event size limit**: Event properties JSON limited to 950 KB. Batch large data or split into multiple events.

## Verification checklist

Before submitting work:

- [ ] **SDK deployed**: Loader script in `<head>`, `live.js` loading, no console errors
- [ ] **Credentials correct**: Workspace ID and API key match dashboard; no extra whitespace
- [ ] **Events flowing**: Dashboard Events section shows incoming events with correct properties
- [ ] **Identity configured**: Identifier tags in Settings › Identity allow-list; `identify()` calls use configured tags
- [ ] **Consent handled**: If `consentRequired: true`, verify `permutive.consent()` called before tracking
- [ ] **Cohorts created**: Custom cohorts visible in dashboard with non-zero audience size
- [ ] **Activations working**: Cohorts flowing to ad platforms (check ad server/SSP for targeting signals)
- [ ] **Debug logging clean**: No errors in `?__permutive.loggingEnabled=true` output
- [ ] **API calls authenticated**: All API requests include `X-API-Key` header or `?k=` param
- [ ] **Event schema valid**: Events match workspace schema; no 400 errors in API responses
- [ ] **Identity resolution tested**: Same user across sessions/devices resolves to single Permutive ID
- [ ] **Staging verified**: Changes tested in staging environment before production rollout

## Resources

- **Comprehensive navigation**: https://docs.permutive.com/llms.txt
- **API Reference**: https://docs.permutive.com/api/introduction
- **JavaScript SDK**: https://docs.permutive.com/sdks/web/javascript-sdk
- **Implementation Guide**: https://docs.permutive.com/implementation-overview
- **Cohorts Guide**: https://docs.permutive.com/guides/signals/cohorts/custom/creating-custom-cohorts

---

> For additional documentation and navigation, see: https://docs.permutive.com/llms.txt