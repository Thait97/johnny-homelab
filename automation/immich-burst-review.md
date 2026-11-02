# Immich Burst Review Automation

## Project Overview

I built a custom automation workflow to identify likely burst-photo sequences in my self-hosted Immich library and place them into a review album.

The goal was not to delete photos automatically.

The goal was to identify likely burst groups, collect them for review, remember which photos I intentionally kept, and avoid repeatedly surfacing the same images.

This project combines:

- Bash
- SQL
- Python
- REST API usage
- database reasoning
- automation logic
- state tracking
- safety controls

## Main Script

The main script is:

```text
~/burst-review-load-all.sh
```

The script queries the Immich PostgreSQL database to identify candidate photos.

## Detection Logic

The current detection logic looks for:

```text
3 or more photos
same rapid sequence
new session if gap > 6 seconds
exclude trashed assets
use face metadata as a safety filter
```

The exact implementation has evolved as I tested the workflow against real photo-library behavior.

## Review Album

Candidate assets are added to an Immich album named:

```text
Burst Review
```

The purpose of this album is to create a controlled review queue.

Instead of deleting photos automatically, the automation surfaces likely burst groups so I can make the final decision manually.

## Why I Did Not Automate Deletion

Immich manages photo assets through its database.

Directly deleting Immich-managed files with commands such as:

```bash
rm
```

could leave the filesystem and database out of sync.

For that reason, deletion should happen through Immich itself, such as:

- Immich UI
- Trash workflow
- supported API operations

The automation is designed around review rather than destructive filesystem operations.

## Asset IDs Instead of Filenames

The workflow uses Immich asset IDs rather than relying on filenames.

This matters because duplicate filenames can exist.

For example:

```text
IMG_1234.JPG
IMG_1234.JPG
```

may refer to different assets.

A database-backed asset ID provides a more reliable identifier for automation.

Conceptually:

```text
Filename
   |
   +-- may not be unique

Asset ID
   |
   +-- unique Immich record
```

## KEEP-Memory System

One of the most useful additions to the project was a small state-tracking system.

The workflow uses:

```text
~/burst-review-keep-ids.txt
~/burst-review-seen-ids.txt
```

These files help the automation remember previous review decisions.

## Review Logic

The workflow uses a rule similar to:

```text
previously seen
+ still active
+ still qualifies
+ removed from Burst Review
= intentionally kept
```

The idea is that if an asset was previously identified as a burst candidate but I manually removed it from the review album without deleting it, that action should be treated as an intentional KEEP decision.

The automation should not keep adding that photo back to the review queue forever.

## Workflow

Conceptually, the process looks like:

```text
Immich Database
      |
      v
Query photo metadata
      |
      v
Find rapid sequences
      |
      +-- fewer than 3 photos --> ignore
      |
      +-- 3+ photos
              |
              v
      Apply safety filters
              |
              v
      Compare seen/keep state
              |
              v
     Candidate asset IDs
              |
              v
       Immich REST API
              |
              v
       Burst Review album
              |
              v
        Manual decision
```

## Database Reasoning

The project required understanding that Immich is more than a folder of image files.

The database tracks application state such as:

- assets
- metadata
- albums
- trash state
- relationships
- other application-managed information

That means automation has to respect the application's data model instead of treating the library like a normal directory tree.

## API Usage

The automation uses the Immich API to interact with application-managed assets.

This is safer than manipulating files directly because Immich remains aware of the changes.

One important security rule for the project is:

```text
Never publish the real API key.
```

Public examples should use placeholders such as:

```text
IMMICH_API_KEY=YOUR_API_KEY_HERE
```

## Safety Controls

The workflow was intentionally designed to reduce the risk of accidental data loss.

Safety decisions include:

- no automatic destructive deletion
- exclude trashed assets from candidate processing
- use asset IDs instead of filenames
- maintain seen-state history
- maintain KEEP-state history
- use a review album before taking action
- avoid publishing credentials

## Why State Tracking Matters

Without state tracking, an automation can repeatedly rediscover the same result.

For example:

```text
Run 1
Photo A added to review

Manual decision
Photo A removed from review because I want to keep it

Run 2
Photo A still matches burst logic
Photo A added again

Run 3
Photo A added again
```

That creates a frustrating loop.

The KEEP-memory logic changes this behavior:

```text
Photo previously reviewed
        |
        v
Still exists and still qualifies
        |
        v
Removed from Burst Review manually
        |
        v
Treat as intentional KEEP
        |
        v
Do not repeatedly re-add
```

This turned the script from a simple detection tool into a state-aware automation workflow.

## What I Learned

### Automation Needs Memory

Some workflows cannot be solved correctly by evaluating only the current state.

Previous human decisions can be part of the logic.

### Databases Matter

A self-hosted application such as Immich should not be treated like a normal folder of files.

Understanding the database and application state is important when building automation around it.

### Unique IDs Are Better Than Filenames

Filenames can collide.

Using application-level asset IDs makes the automation more reliable.

### Safe Automation Is Often Better Than Full Automation

Automatically deleting duplicate-looking photos would be risky.

Creating a review queue gives the automation useful power while keeping the final destructive decision with the user.

### APIs Preserve Application State

Using the supported application API is generally safer than making direct filesystem changes behind the application's back.

## Technologies Used

This project gave me practical experience with:

- Bash
- SQL
- PostgreSQL queries
- Python
- REST APIs
- text-file state tracking
- application databases
- conditional logic
- automation safety
- debugging

## Result

The workflow now automatically identifies likely burst-photo sequences and places them into a controlled Immich review album.

It also remembers previously reviewed assets so intentionally kept photos are not repeatedly surfaced.

The project evolved from a simple photo-detection script into a state-aware automation system built around database records
