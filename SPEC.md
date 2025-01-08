The first version of this specification was written via a conversation with OpenAI's o1 model.

---

## 1. Project Overview

**App Name (Placeholder)**  
“WordlyWeeks” (or any preferred name)

**Purpose**  
- Let parents track their child’s “word (or phrase) of the week” starting from birth.  
- Optionally add a description and milestone tags (with small icons).  
- (Premium) Optionally add one photo per week.  
- Handle multiple children, aligned by their “week of life.”  
- Provide a way to export or share a summary (image export, plus optional JSON backup).  

**Platform & Tools**  
- **Language**: Swift (latest version).  
- **UI Framework**: SwiftUI.  
- **Data Layer**: Core Data / CloudKit or a combination for iCloud sync.  
- **Notifications**: Local notifications (UNUserNotificationCenter).  
- **Target iOS Version**: iOS 15+ or 16+ (depending on modern SwiftUI features).

---

## 2. User Experience & Features

### 2.1 Onboarding
1. **Splash/Welcome Screen**  
   - Explains the app concept briefly.  
   - Prompts to create the first child’s profile (name, birthdate, color theme).
2. **Child Creation**  
   - **Name**: Text field.  
   - **Birthdate**: Date picker.  
   - **Color**: Choose from preset color swatches or a color picker.  
   - **Active / Inactive**: Defaults to active upon creation.

### 2.2 Children Management
1. **Free/Paid Tiers**  
   - **Free**: 1 child only, no photo uploads.  
   - **Premium** (“Unlock Everything” in-app purchase): Multiple children + ability to add 1 photo per week.
2. **View Children**  
   - A list or grid of children, showing name, color tag, last entered week, and active/inactive status.  
   - Tapping a child goes to their dedicated timeline screen (see below).

### 2.3 Week-Oriented Entry Screen
- **Goal**: Quickly enter the word/phrase for all **active** children in the current week.  
- **Weekly Scrolling**:
  - By default, shows the “current week” (based on Monday start, child’s birth date is Week 1).  
  - A button to navigate to previous weeks or future weeks if needed.  
- **Per-Child Section**:
  - Child’s color theme (background or accent).  
  - Text field for “word/phrase.”  
  - Optional text field for “description.”  
  - (If premium) Option to add one photo.  
  - List of milestone tags (checkbox or toggle style, with associated icons or emojis).  
- **Save/Submit**:
  - Writes to the local data model and syncs to iCloud in the background.

### 2.4 Individual Child Timeline View
1. **Graphical Table View**  
   - Strict layout with the following for each week:  
     - **Week #**  
     - **Word/Phrase** (color-coded to differentiate weeks).  
     - Optional **photo thumbnail** (for premium).  
     - **Milestone icons**.  
     - (Expandable) **description** if user taps.
   - Scroll vertically through the weeks (1…N).
2. **Paragraph View**  
   - Text blocks concatenated with minimal styling.  
   - Displays each week’s word, preceded by a small “Week X” label.  
   - Each “Week X” label can use the child’s color theme.  
   - Optional icons for milestones inline or listed after the text.

### 2.5 Milestone Tags
- **Global Milestone List** (shared across all children).  
- User can:
  1. Add new custom tags (with an icon — can be an emoji or a small uploaded image).  
  2. Delete or modify existing tags.
- Assign zero or more milestones to each week’s entry.

### 2.6 Notifications
- **Weekly Reminder**  
  - Local notification each week, e.g., Sunday night or Monday morning.  
  - Customizable day/time.  
  - “Don’t forget this week’s word for ChildName!”

### 2.7 Export & Sharing
1. **Image Export**  
   - Export a single continuous view (likely the Paragraph View or Graphical Table View) as an image.  
   - Use the standard iOS share sheet.
2. **JSON Export**  
   - Exports text data only (no photos).  
   - Structure includes child info, week entries, word, description, milestone tags, etc.  
   - Also shareable via share sheet (e.g., saving to Files, emailing, etc.).

---

## 3. Technical Details

### 3.1 Data Model

**Entities** (in Core Data or an equivalent structure):  
1. **Child**  
   - **id** (UUID)  
   - **name** (String)  
   - **birthdate** (Date)  
   - **colorTheme** (String or stored color representation)  
   - **isActive** (Bool)  
2. **WeekEntry**  
   - **id** (UUID)  
   - **childId** (Relationship to Child)  
   - **weekNumber** (Int)  
   - **word** (String)  
   - **description** (String, optional)  
   - **photo** (URL or binary data, optional for premium)  
   - **milestones** (Relationship to “Milestone” or store milestone IDs in an array)
3. **Milestone**  
   - **id** (UUID)  
   - **label** (String, e.g., “First Steps”)  
   - **icon** (String or Data; can store emoji or an image)  
   - Might have a global scope or belong to the user’s “settings” entity.

**Week Calculation**  
- For each child:  
  - Calculate “Week X” by `(currentDate - birthdate) / 7 days`, rounding appropriately, always starting the week on Monday.  
  - The child’s birth week is labeled as Week 1.

### 3.2 iCloud Sync
- Use **CloudKit** or **Core Data with CloudKit** for seamless background syncing.  
- The app should function offline, storing data locally, and automatically sync changes to iCloud when online.  
- Store images in the user’s iCloud (either as Core Data Binary or CloudKit assets).

### 3.3 In-App Purchases
- **Single Non-Consumable “Unlock Everything”**  
  - Allows adding multiple children.  
  - Enables photo uploads.  
- Implement via **StoreKit**.  
- If user does not own the purchase:
  - Limit 1 active child profile.  
  - Hide/disable the photo field in weekly entries.

### 3.4 User Interface
- **SwiftUI** for all screens.  
- **Navigation**:
  1. **Tab or Sidebar** with “Home” (Week-Oriented Entry), “Children List,” “Settings.”  
  2. **Child Timeline** is accessed by tapping a child from the list.  
- **Design**:
  - Modern, minimal aesthetic with subtle color usage.  
  - Each child’s color theme in the timeline or data-entry screen.  
  - Strict layout for the Graphical Table View, e.g., text on the left, optional photo on the right, minimal row height.

### 3.5 Notifications
- **Local Notifications** via **UNUserNotificationCenter**.  
- On app launch or in Settings, user can configure:
  - **Day of week** (or “weekly”)  
  - **Time**  
  - Fire the local notification reminding them to add a new entry.

### 3.6 Handling Photos (Premium)
- Upon adding a photo, the app:
  1. Resizes/compresses large images locally to a standard max dimension (e.g., 1200x1200) before saving to Core Data or as a CloudKit asset.  
  2. Stores the reference for offline access and iCloud sync.  
  3. Photos do **not** appear in JSON exports.

### 3.7 Data Export
1. **Image Export**  
   - Render SwiftUI view into an image (using `UIViewRepresentable` or SwiftUI’s native snapshot APIs in iOS 16+).  
   - Share via standard iOS share sheet.
2. **JSON Export**  
   - Provide an option in Settings or Child Timeline to “Export Data.”  
   - The JSON file includes:
     ```json
     {
       "child": {
         "name": "...",
         "birthdate": "...",
         "colorTheme": "...",
         "entries": [
           {
             "weekNumber": ...,
             "word": "...",
             "description": "...",
             "milestones": [...]
           },
           ...
         ]
       }
     }
     ```
   - Milestone icons could be omitted or represented by text (emoji) if feasible.

### 3.8 Offline Functionality
- **Core Data** stores everything locally.  
- **Sync** occurs in the background via CloudKit.  
- User can continue adding/editing words even without internet; changes push when network is available.

### 3.9 Settings & Other Details
- **Milestone Management**:  
  - A “Milestones” screen to add/edit/remove global milestones.  
  - Each milestone has a name and icon.  
- **Child Management**:  
  - Mark child “inactive” if finished logging weekly words.  
  - Possibly reorder child list or simply show inactive separately.  
- **Privacy & Security**:  
  - Provide a Privacy Policy to comply with App Store guidelines.  
  - Possibly integrate with Apple’s “App Privacy” to explain how data (photos, local entries) is handled.

---

## 4. Development Roadmap (MVP & Future Enhancements)

1. **MVP**  
   - Single child for free, multiple children with IAP.  
   - Word/phrase + description + milestones.  
   - Weekly data entry (offline + iCloud).  
   - Basic SwiftUI UI for table view and paragraph view.  
   - Local notifications once per week.  
   - Image export and JSON export (text only).  
   - Simple color selection.  
   - Minimal settings for reminders and milestone customization.

2. **Future Enhancements**  
   - Photo uploads for premium users.  
   - More robust milestone icons (custom image uploads, emoji picker).  
   - Enhancements to the paragraph view or layout templates.  
   - Collaboration/sharing with multiple caregivers (if Apple’s iCloud sharing approach is simple enough).

---

## 5. Summary

This specification outlines a **family-focused iOS app** using **SwiftUI** and **native Apple frameworks** (Core Data, CloudKit, UNUserNotificationCenter) to track weekly words, milestone tags, and optional photos for each child. A single, non-consumable in-app purchase unlocks multiple child profiles and photo-upload capabilities. The design approach is **simple and modern**, relying on color themes per child and a user-friendly weekly entry system.

With these guidelines in place, a developer (or another AI system) should have a solid starting point to **build and iterate** on the MVP, then layer on premium and future enhancements. If any detail needs deeper exploration, they can refer back to each section’s outline.
