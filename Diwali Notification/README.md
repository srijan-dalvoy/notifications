# Diwali ₹1 Pro: push notification images

Four images for the Diwali ₹1 Pro push notification on Android and iOS.

| File | Size | Shape | Used for |
|---|---|---|---|
| `diwali-pro-android-expanded-1024x512.png` | 1024 × 512 | 2:1 | Android expanded image (the size Android recommends) |
| `diwali-pro-ios-expanded-1200x600.png` | 1200 × 600 | 2:1 | iOS expanded image; a sharper Android fallback |
| `diwali-pro-ios-thumbnail-512x512.png` | 512 × 512 | 1:1 | iOS collapsed thumbnail |
| `diwali-pro-android-large-icon-256x256.png` | 256 × 256 | 1:1 | Android large icon (collapsed view) |

The banner is the original design. It was trimmed by 16 px on the left and right to make it exactly 2:1, and nothing was removed. The thumbnail shows only the diya, "₹1" and "Pro", so it reads at small sizes and survives a circle crop.

---

## How each one appears

**Collapsed (before the user expands it)**
- **Android:** title and body text, with the **large icon** (`diwali-pro-android-large-icon-256x256.png`) on the right. Android may crop it to a circle, so its content is kept in the centre.
- **iOS:** title and body text, with a **small square thumbnail** on the right. iOS takes this from the attachment, so send the square image as an attachment too (see below).

**Expanded (after a long-press or pull down)**
- **Android:** the banner shows full width below the text (`BigPictureStyle`). When expanded, the large icon is hidden.
- **iOS:** the attachment shows full width above the text.

---

## Android

Use `BigPictureStyle`: the banner as the big picture, and the thumbnail as the large icon.

```kotlin
val banner = BitmapFactory.decodeStream(URL(bannerUrl).openStream())        // diwali-pro-android-expanded-1024x512.png
val thumb  = BitmapFactory.decodeStream(URL(thumbUrl).openStream())         // diwali-pro-android-large-icon-256x256.png

val n = NotificationCompat.Builder(context, CHANNEL_ID)
    .setSmallIcon(R.drawable.ic_stat_logo)          // monochrome status-bar icon, not these images
    .setContentTitle("Diwali offer: Pro for ₹1")
    .setContentText("Mains evaluations, PYQs, live classes and more")
    .setLargeIcon(thumb)
    .setStyle(
        NotificationCompat.BigPictureStyle()
            .bigPicture(banner)
            .bigLargeIcon(null as Bitmap?)          // hide the thumbnail when expanded
    )
    .setOngoing(true)                               // "sticky"
    .setContentIntent(pendingIntent)
    .build()
```

Through FCM, send `diwali-pro-android-expanded-1024x512.png` as `notification.image`. Note that FCM's default display does not set a separate large icon or make the notification sticky, so for those two, send a **data message** and build the notification in the app as above.

**Sticky (ongoing):** `setOngoing(true)` keeps it from being swiped away on Android 13 and below. From **Android 14**, users can swipe most ongoing notifications away, except media and some system ones.

---

## iOS

iOS shows images only through a **Notification Service Extension**: the push says `"mutable-content": 1`, and the extension downloads the image and attaches it.

```json
{
  "aps": {
    "alert": { "title": "Diwali offer: Pro for ₹1", "body": "Mains evaluations, PYQs, live classes and more" },
    "mutable-content": 1
  },
  "image": "https://…/diwali-pro-ios-expanded-1200x600.png"
}
```

In the extension, attach the banner, and use the thumbnail clipping option to choose what the collapsed square shows:

```swift
let attachment = try UNNotificationAttachment(
    identifier: "banner", url: localFileURL,
    options: [UNNotificationAttachmentOptionsThumbnailClippingRectKey:
              CGRect(x: 0.03, y: 0.20, width: 0.32, height: 0.64).dictionaryRepresentation]
)
```

That clipping rectangle uses fractions of the image's width and height. Here it frames the diya + ₹1 Pro on the left of the banner, so iOS does not show its default centre crop, which would cut through the bullet list. If you would rather use the clean square image, attach `diwali-pro-ios-thumbnail-512x512.png` as the **first** attachment and the banner as the second. The collapsed view uses the first one.

Keep each attachment under 10 MB; these are all under 600 KB.

**Sticky:** iOS has no sticky notifications. A push stays in Notification Centre until the user clears it. The nearest thing is a **Live Activity** (lock screen and Dynamic Island), which uses its own SwiftUI layout, not these images.

---

## Notes

- **Put the message in the text, too.** Some phones don't load the image, or keep it collapsed. The title and body should carry the offer on their own.
- **Small text in the expanded banner.** On a phone, the banner shrinks to about 340 px wide, so the five bullets show at roughly 10 px. They can be read, but not at a glance. The ₹1 and diya read clearly.
- **Logo at the right edge.** The EO logo is partly cut by the right edge (as in the original), and Android's rounded corners may cut slightly more.
- **Status-bar icon.** On Android, the small icon (`setSmallIcon`) must be a white-on-transparent silhouette. These images are not for that.
