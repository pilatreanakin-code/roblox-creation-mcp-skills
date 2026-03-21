# GUI Design Template — Required Sections

## 1. Style Guide (applies to ALL screens)
- Color palette: primary, secondary, background, text, accent, success, error, currency colors (exact RGB)
- Font: title font, body font, button font (exact Roblox font names)
- Border style: thickness, color, UICorner radius per element size
- Button style: colors, hover effect, click effect
- Close button style: shape, color, position
- Title style: size, color, stroke, position relative to frame
- Animation standards: open (style, duration, easing), close (style, duration, easing)

## 2. Screen List
Every screen in the game listed with: name, purpose, when it appears, how to open/close

## 3. HUD Layout (always visible during gameplay)
- What's shown: health, currency, level, XP, zone name, etc.
- Position of each element (top-left, top-right, bottom-center, etc.)
- Exact layout description with Scale sizes

## 4. Navigation Bar
- Position (left sidebar, bottom bar, etc.)
- Each button: icon, label, what it opens
- Style per button

## 5. Per-Screen Detail
For EVERY screen, document:
- **Purpose:** what this screen does
- **Frame:** size, position, background, border, corner radius
- **Title:** text, font, size, position
- **Close button:** position, style
- **Content area:** what's inside (grid, list, tabs, etc.)
- **Tabs/categories:** names, active/inactive styles
- **Interactive elements:** every button with what it does
- **Data displayed:** what info from the server is shown
- **Animations:** open/close, hover, click
- **Mobile differences:** what changes on small screens

## 6. Popup System
- What popups exist (confirmation, reward, level up, error)
- Priority order (which popup shows first if multiple trigger)
- Layout of each popup

## 7. Notification/Toast System
- Types (success, error, info, achievement)
- Position, animation, duration
- Queue behavior
