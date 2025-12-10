---
name: Add Minigames Feature
overview: Add minigame system with drawing games for waiting room and optional mid-game breaks. Host can trigger minigames, and they're just for fun (no scoring).
todos:
  - id: backend_minigame_state
    content: Add minigame state fields to GameRoom class and update STATES list
    status: pending
  - id: backend_minigame_handlers
    content: Implement minigame message handlers (start_minigame, end_minigame, minigame_submit)
    status: pending
    dependencies:
      - backend_minigame_state
  - id: player_minigame_ui
    content: Create minigame screen HTML with canvas element in index.html
    status: pending
  - id: player_drawing_canvas
    content: Implement canvas drawing functionality with touch/mouse support in player.js
    status: pending
    dependencies:
      - player_minigame_ui
  - id: player_minigame_logic
    content: Add showMinigame() and minigame submission handling in player.js
    status: pending
    dependencies:
      - player_drawing_canvas
  - id: host_minigame_controls
    content: Add minigame start/end controls and type selector in host UI
    status: pending
    dependencies:
      - backend_minigame_handlers
  - id: host_minigame_display
    content: Implement display of player submissions on host screen
    status: pending
    dependencies:
      - host_minigame_controls
  - id: styling
    content: Add CSS styling for minigame screens and canvas controls
    status: pending
    dependencies:
      - player_minigame_ui
      - host_minigame_controls
---

# Add Minigames Feature

Add a minigame system to provide entertainment during waiting room and optional breaks between questions. Focus on drawing/creative games that don't award points.

## Architecture Overview

- Add new game state: `"minigame"` to `GameRoom.STATES`
- Minigames are host-controlled and can be triggered from lobby or after reveal
- Players participate via WebSocket messages
- Results are displayed but don't affect quiz scores

## Implementation Plan

### 1. Backend: Minigame Infrastructure (`server.py`)

**Add to GameRoom class:**

- Add `minigame_state` field to track current minigame (type, start time, submissions)
- Add `minigame_submissions: dict[str, dict]` to store player drawings/responses

**New message handlers:**

- `handle_host_message`: Add `start_minigame` and `end_minigame` cases
- `handle_player_message`: Add `minigame_submit` case for player submissions
- Create `start_minigame(room, minigame_type)` function
- Create `end_minigame(room)` function to broadcast results

**Minigame types to implement:**

- `draw_prompt`: Players draw based on a prompt (e.g., "Draw a cat")
- `draw_freestyle`: Free drawing (for waiting room)

### 2. Frontend: Player Minigame UI (`player.js` + `player.html`)

**Add new screen:**

- `minigameScreen` div in `index.html` with canvas for drawing
- Drawing controls: color picker, brush size, clear button, submit button

**JavaScript functions:**

- `showMinigame(data)` - displays minigame UI based on type
- `initDrawingCanvas()` - sets up HTML5 canvas with touch/mouse support
- `submitMinigameAnswer()` - sends drawing data to server
- Handle `minigame_start` and `minigame_end` WebSocket messages

**Update lobby screen:**

- Show minigame UI overlay when minigame is active in lobby
- Allow players to participate while waiting

### 3. Frontend: Host Minigame Controls (`host.js` + `host.html`)

**Add controls:**

- "Start Minigame" button in lobby and reveal screens
- Minigame type selector dropdown
- Display submitted drawings/responses on host screen
- "End Minigame" button to return to normal flow

**New functions:**

- `showMinigameControls()` - displays minigame UI
- `displayMinigameSubmissions(data)` - shows all player submissions
- Handle `minigame_submission` messages to update display in real-time

### 4. Drawing Game Implementation

**Canvas-based drawing:**

- Use HTML5 Canvas API
- Support touch events for mobile
- Basic tools: brush, color picker, clear
- Convert canvas to base64 image for transmission
- Compress image data before sending

**Backend storage:**

- Store base64 image data in `minigame_submissions[player_id]`
- Include player name and timestamp
- On minigame end, broadcast all submissions to host

### 5. State Management

**Game flow integration:**

- Lobby state: Host can start waiting room minigame
- After reveal: Host can optionally start break minigame before next question
- Minigame state: Temporary state that doesn't block game progression
- Return to previous state (lobby/reveal) when minigame ends

## Files to Modify

- `LibraryQuiz/server.py`: Add minigame state, handlers, and logic
- `LibraryQuiz/static/index.html`: Add minigame screen HTML
- `LibraryQuiz/static/player.js`: Add minigame UI and canvas handling
- `LibraryQuiz/static/player.css`: Style minigame screen and canvas
- `LibraryQuiz/static/host.html`: Add minigame controls UI
- `LibraryQuiz/static/host.js`: Add host minigame controls
- `LibraryQuiz/static/host.css`: Style minigame display

## Technical Details

**Drawing data format:**

```json
{
  "type": "minigame_submit",
  "minigame_type": "draw_prompt",
  "data": "data:image/png;base64,...",
  "prompt": "Draw a cat"
}
```

**Minigame state structure:**

```python
room.minigame_state = {
    "type": "draw_prompt",
    "prompt": "Draw a cat",
    "start_time": time.time(),
    "duration": 60  # seconds
}
```

## Future Extensibility

Structure allows easy addition of more minigame types:

- Word games (typing challenges)
- Reaction games (tap timing)
- Trivia light games

Each minigame type will have its own UI component and submission handler.