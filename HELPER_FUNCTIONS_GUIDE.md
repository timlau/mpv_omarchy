# ModernZ Helper Functions Quick Reference Guide

## Overview

This guide documents the helper functions added to `modernz.lua` to improve code readability and reduce duplication.

---

## Constants

### UI Thresholds and Dimensions

```lua
-- Mouse and hover zone thresholds (in pixels)
MOUSE_HOVER_TOP_THRESHOLD = 40           -- Y-position threshold for top hover detection
DEFAULT_BOTTOM_HOVER_ZONE = 130          -- Default height of bottom hover zone

-- Element visibility offset values
ELEMENT_HIDDEN_OFFSET = 100              -- Offset value indicating element should be hidden
ELEMENT_VISIBLE_OFFSET = 0               -- Offset value indicating element should be visible

-- Screen width thresholds for responsive UI
MIN_SMALL_SCREEN_WIDTH = 300             -- Minimum width for image content display
MIN_NORMAL_SCREEN_WIDTH = 500            -- Minimum width for normal content display
MIN_MEDIUM_SCREEN_WIDTH = 400            -- Minimum width for medium-sized elements
MIN_LARGE_SCREEN_WIDTH = 550             -- Minimum width for larger button groups
MIN_XLARGE_SCREEN_WIDTH = 1150           -- Minimum width for volume bar and extensive UI

-- Softrepeat timing
SOFTREPEAT_FRAMES_BEFORE_REPEAT = 15     -- Frames to wait before repeating action
SOFTREPEAT_FRAME_INTERVAL = 5            -- Interval between repeated actions (in frames)
```

---

## Helper Functions

### `sanitize_title(raw_title, should_escape_ass)`

**Purpose:** Clean and format window/element titles

**Parameters:**
- `raw_title` (string|nil): The raw title text to sanitize
- `should_escape_ass` (boolean): Whether to escape ASS formatting codes

**Returns:** (string) Sanitized title, or "mpv" if empty

**Behavior:**
- Replaces newlines with spaces
- Escapes ASS formatting when requested
- Returns "mpv" as fallback for empty titles
- Handles nil input gracefully

**Example:**
```lua
-- In window title element
ne.content = function()
  local title = mp.command_native({ "expand-text", user_opts.window_title })
  return sanitize_title(title, true)
end
```

---

### `get_visibility_offset(condition)`

**Purpose:** Convert visibility condition to element offset value

**Parameters:**
- `condition` (boolean): The visibility condition to evaluate

**Returns:** (number) Either `ELEMENT_VISIBLE_OFFSET` (0) or `ELEMENT_HIDDEN_OFFSET` (100)

**Behavior:**
- Returns 0 if condition is true (element visible)
- Returns 100 if condition is false (element hidden)
- Used for calculating UI layout offsets

**Example:**
```lua
-- Before: Confusing ternary operators
local audio_offset = (audio_track_count == 0 or not mp.get_property_native("aid")) and 100 or 0

-- After: Clear intent
local audio_offset = get_visibility_offset(audio_track_count > 0 and mp.get_property_native("aid"))
```

---

### `is_playlist_button_visible(nojump, noskip)`

**Purpose:** Calculate playlist button visibility based on screen width and offsets

**Parameters:**
- `nojump` (number): The jump button offset value
- `noskip` (number): The chapter skip button offset value

**Returns:** (boolean) True if playlist buttons should be visible

**Behavior:**
- Adapts minimum width based on whether displaying images or video
- Accounts for button group adjustments
- Performs responsive width calculation

**Example:**
```lua
ne = new_element("playlist_prev", "button")
ne.visible = is_playlist_button_visible(nojumpoffset, noskipoffset)
```

---

### `setup_slider_value_tracking(element)`

**Purpose:** Attach common slider event handlers for value tracking

**Parameters:**
- `element` (table): The slider element to configure

**Returns:** None (modifies element in place)

**Behavior:**
- Adds `mouse_move` handler for slider position tracking
- Adds `reset` handler for clearing last position
- Updates OSD message display during interaction
- Prevents redundant commands when position hasn't changed

**Example:**
```lua
ne = new_element("volumebar", "slider")
ne.slider = { min = { value = 0 }, max = { value = volume_max } }
setup_slider_value_tracking(ne)  -- Automatically setup handlers
ne.eventresponder["mbtn_left_down"] = function(element)
  local pos = get_slider_value(element)
  mp.commandv("osd-msg", "set", "volume", set_volume(pos))
end
```

---

### `create_button_with_softrepeat(name, content_val, should_softrepeat)`

**Purpose:** Create a button element with softrepeat configuration

**Parameters:**
- `name` (string): The element name identifier
- `content_val` (string|function): The button's display content
- `should_softrepeat` (boolean): Whether to enable softrepeat functionality

**Returns:** (table) The newly created button element

**Behavior:**
- Creates new element with `new_element(name, "button")`
- Sets content value
- Configures softrepeat property
- Returns element for further configuration

**Example:**
```lua
ne = create_button_with_softrepeat("jump_backward", jump_icon[1], user_opts.jump_softrepeat)
ne.eventresponder["mbtn_left_down"] = function()
  mp.commandv("seek", -jump_amount, jump_mode)
end
```

---

### `calculate_button_visibility(current_width, min_width, adjustment_offset)`

**Purpose:** Determine if button should be visible based on screen width

**Parameters:**
- `current_width` (number): Current screen width
- `min_width` (number): Minimum required width for button
- `adjustment_offset` (number|nil): Additional offset to subtract from minimum

**Returns:** (boolean) True if button should be visible

**Note:** Currently unused but available for future responsive UI improvements

---

### `create_button_element(name, content_value)`

**Purpose:** Create a simple button element

**Parameters:**
- `name` (string): The element name identifier
- `content_value` (string|function): The button's display content

**Returns:** (table) The newly created button element

**Note:** Available for future use; most button creation now uses `create_button_with_softrepeat()`

---

## Event Processing Helpers

### `process_event_down_press(action, source, what)`

**Purpose:** Handle mouse down and press events

**Part of:** The refactored `process_event()` function

**Behavior:**
- Resets hide timeout
- Finds elements under mouse cursor
- Sets active element and event source
- Fires appropriate event handlers

---

### `process_event_up()`

**Purpose:** Handle mouse button release events

**Part of:** The refactored `process_event()` function

**Behavior:**
- Executes up event on active element if mouse still hovering
- Triggers reset handlers
- Clears active element state

---

### `process_event_mouse_move(action, source)`

**Purpose:** Handle mouse movement and hover events

**Part of:** The refactored `process_event()` function

**Behavior:**
- Checks minimum mouse movement threshold
- Handles bottom hover zone detection
- Shows/hides OSC based on hover zones
- Updates active element handlers

---

## Usage Patterns

### Pattern 1: Simplifying Visibility Logic

**Before:**
```lua
local offset = (user_opts.feature_enabled and 0 or 100)
local offset2 = (some_check and 0 or 100)
```

**After:**
```lua
local offset = get_visibility_offset(user_opts.feature_enabled)
local offset2 = get_visibility_offset(some_check)
```

---

### Pattern 2: Title Formatting

**Before:**
```lua
ne.content = function()
  local title = mp.command_native({ "expand-text", user_opts.title }) or ""
  title = title:gsub("\n", " ")
  return title ~= "" and mp.command_native({ "escape-ass", title }) or "mpv"
end
```

**After:**
```lua
ne.content = function()
  local title = mp.command_native({ "expand-text", user_opts.title })
  return sanitize_title(title, true)
end
```

---

### Pattern 3: Responsive Button Visibility

**Before:**
```lua
ne.visible = (
  osc_param.playresx
  >= (state.is_image and 300 or 500) - nojumpoffset - noskipoffset * (nojumpoffset == 0 and 1 or 10)
)
```

**After:**
```lua
ne.visible = is_playlist_button_visible(nojumpoffset, noskipoffset)
```

---

## Best Practices

1. **Use Named Constants** instead of magic numbers
   - Makes code self-documenting
   - Easier to find and update values

2. **Use Helper Functions** to reduce duplication
   - Apply DRY (Don't Repeat Yourself) principle
   - Improves maintainability

3. **Prefer Helper Functions** over inline lambdas for complex logic
   - Improves readability
   - Easier to debug
   - More testable

4. **Use Consistent Naming**
   - `get_*` for functions that retrieve/calculate values
   - `setup_*` for functions that configure objects
   - `process_*` for functions that handle events
   - `create_*` for functions that instantiate objects

---

## Migration Guide

If you're updating existing code to use these helpers:

1. **Replace `and 0 or 100` patterns** with `get_visibility_offset(condition)`
2. **Replace title sanitization code** with `sanitize_title(title, escape_ass)`
3. **Replace repeated button creation** with helper functions
4. **Replace slider setup code** with `setup_slider_value_tracking(element)`

---

## Performance Notes

- All helper functions have minimal overhead
- Constants are evaluated at script load time (zero runtime cost)
- Helper functions use direct logic (no recursion or heavy computation)
- No performance regression compared to inline code

---

## Future Enhancement Opportunities

1. Create similar helpers for subtitle formatting
2. Extract element styling into helper functions
3. Create batch element creation helpers
4. Add type annotations using Lua comments
5. Create configuration preset helpers

---

## Questions or Issues?

Refer to the `REFACTORING_SUMMARY.md` file for detailed before/after comparisons and rationale for each change.