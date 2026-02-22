# Scenario 06: Course Certificate Locked Despite 100% Video Progress

## Scenario
An online learning platform tracks video progress and unlocks certificates when 100% of the course is completed. Many users report that their videos show fully watched, but certificates remain locked even after refreshing and re-logging.

---

## Assumptions
- Video progress is tracked via events (e.g., play, pause, seek, complete).
- A backend service calculates overall course completion percentage.
- Certificate unlocking is triggered by a completion event or periodic job.
- Progress data may be stored in both a database and a cache.

---

## Test Ideas

### Event Tracking Validation
- Verify that video completion events are correctly sent from the client to the backend.
- Check if the "video complete" event fires at 100% or at a threshold (e.g., 95%).
- Test what happens if the user skips to the end of a video — is it marked complete?
- Confirm events are not lost when the network is interrupted during viewing.

### Progress Calculation Testing
- Query the backend directly for a user's course completion percentage.
- Compare the UI-displayed progress vs the actual backend-stored value.
- Test with a course that has multiple modules — is all module progress aggregated correctly?
- Verify that re-watching a completed video does not reset the completion status.

### Completion Trigger Validation
- Test the certificate unlock trigger: is it event-driven or polled (cron job)?
- If cron-based, check the job schedule and whether it processes all pending completions.
- Manually trigger the completion check for an affected user and verify certificate unlocks.
- Verify the certificate unlock API/logic handles edge cases (e.g., user completed in one session).

### Database & Cache Consistency
- Check if the progress shown in the UI comes from cache while the backend calculation uses the DB.
- Verify the cache is updated when the completion event is processed.
- Test the scenario where cache shows 100% but DB has a slightly lower value.

---

## Edge Cases
- User watches a video, closes the browser at 99% — is it marked complete?
- Video has chapters and the user skips a short chapter — is it still counted as complete?
- User completes course on mobile but checks certificate on web — cross-device sync issue.
- Backend calculates completion based on number of videos watched, but a new video was added to the course.
- Certificate service is down — completion triggers are queued but not processed.

---

## Risks
- **User Trust Risk:** Users who completed the course feel cheated if certificates are withheld.
- **Business Risk:** Platform credibility suffers; users may abandon the course or platform.
- **Data Integrity Risk:** Progress data inconsistency makes it hard to audit completion.
- **Scalability Risk:** Cron-based triggers may not scale during high completion volumes.
- **Regression Risk:** A video library update (new content added) may break completion logic.
