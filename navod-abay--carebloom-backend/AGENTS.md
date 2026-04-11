# CareBloom Backend - Developer Instructions

## Notification Service Usage

Use `NotificationDispatchService` for sending push notifications:

```java
@Autowired
private NotificationDispatchService notificationService;

// Send field visit notification
notificationService.sendFieldVisitNotification(motherId, "2024-12-15", "10:30 AM");

// Send health tip
notificationService.sendHealthTip(motherId, "Tip Title", "Tip content...");

// Send workshop notification
notificationService.sendWorkshopNotification(motherId, "Workshop Title", "2024-12-20", "Location");

// Send appointment reminder
notificationService.sendAppointmentReminder(motherId, "clinic checkup", "2024-12-18", "2:00 PM");

// Send emergency alert to multiple mothers
notificationService.sendEmergencyAlert(List.of("id1", "id2"), "Alert Title", "Alert message");

// Send custom notification
notificationService.sendCustomNotification(motherId, "Title", "Body", Map.of("key", "value"));
```

**Note:** Mother acceptance notifications are sent automatically when MOH accepts registration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/navod-abay)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/navod-abay)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
