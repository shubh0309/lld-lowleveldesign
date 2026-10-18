# Meeting Scheduler — Low Level Design

Schedules meetings into rooms based on capacity and time-slot availability, keeps per-user and per-room calendars, sends invite/cancellation notifications and tracks participant responses.

**Package:** `com.conceptcoding.interviewquestions.meetingscheduler`

## Entry Point

`Demo.main()` — creates three rooms (capacity 4, 8, 12) and seven users, schedules two meetings for the same time slot (the second gets a different room), prints user and room calendars, then cancels both meetings and prints the now-empty calendars.

Run `Demo` directly from your IDE.

## Folder Structure

```
meetingscheduler/
├── Demo.java                 # Entry point (main method)
├── MeetingScheduler.java     # Finds rooms, schedules/cancels meetings, notifies users
├── Meeting.java              # Meeting details and attendee responses
├── MeetingRoom.java          # Room capacity, booked slots and calendar
├── Calendar.java             # Meetings keyed by TimeInterval (for users and rooms)
├── TimeInterval.java         # Start/end time with overlap check
├── User.java                 # User with calendar; responds to invites
├── Notification.java         # Sends invite/cancellation messages
├── ParticipantResponse.java  # ACCEPTED, DECLINED, TENTATIVE, NOT_RESPONDED
└── ParticipantType.java      # ORGANIZER, ATTENDEE
```

## Classes

| Class | Responsibility |
|-------|----------------|
| `MeetingScheduler` | Holds all `MeetingRoom`s. `scheduleMeeting()` finds the first room with enough capacity and no overlapping slot, creates a `Meeting`, adds it to organiser/participant/room calendars, books the slot, sends invites and simulates all attendees accepting. `cancelMeeting()` releases the room slot, removes the meeting from calendars and notifies accepted participants. Rooms can be added/removed. |
| `Meeting` | ID, subject, `TimeInterval`, organiser, `MeetingRoom` and a `Map<User, ParticipantResponse>` of attendees; helpers return accepted, declined and tentative participants. |
| `MeetingRoom` | ID, name, capacity, booked `TimeInterval`s and a `Calendar`. `isAvailableFor(interval, capacity)` checks capacity and overlaps. |
| `Calendar` | `Map<TimeInterval, Meeting>` used by both users and rooms; add, remove and list meetings. |
| `TimeInterval` | Start and end `LocalDateTime`; `overlapsWith()` checks for a clash. |
| `User` | Name, email, `ParticipantType` and `Calendar`. `respondToInvitation()` records the response and adds/removes the meeting from the calendar. |
| `Notification` | Prints invite and cancellation notifications to users. |
| `ParticipantResponse` | Enum: `ACCEPTED`, `DECLINED`, `TENTATIVE`, `NOT_RESPONDED`. |
| `ParticipantType` | Enum: `ORGANIZER`, `ATTENDEE`. |

## Design Notes

- **Shared `Calendar` abstraction** for users and rooms.
- **Room allocation:** first-fit on capacity and non-overlapping time slot.
- **Notifications** are console output; can be replaced with an Observer/notification service.
