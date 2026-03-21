# Screenshot Checklist — Run After EVERY Build

Take a screenshot. Check every item. Fix failures. Screenshot again. Repeat until all pass.

## Visual
- [ ] Title big enough? Min TextSize 56. Biggest text on screen.
- [ ] Title white + black stroke?
- [ ] Nav buttons square, not circle, not tiny? Min 0.06 Scale width.
- [ ] Label below nav button 2-4px gap max? Nearly touching.
- [ ] Close button rounded square (UICorner 8px)? Not circle.
- [ ] Smaller buttons less rounded than big frames?
- [ ] Cards have visible spacing? Not touching each other?
- [ ] All text fully visible? No "..." truncation?
- [ ] Decorative element used? (inner colored frame OR rotated bg frame, not both)

## Alignment
- [ ] ScrollingFrame bottom flush with parent frame bottom? Zero gap.
- [ ] Inner frame (if used) bottom flush with outer frame? Zero gap.
- [ ] Content centered properly? Nothing floating off to one side?
- [ ] Tabs evenly spaced?

## Functionality
- [ ] Nav button click opens the frame?
- [ ] Close X button closes?
- [ ] Clicking dim background closes?
- [ ] Escape key closes?
- [ ] Tabs swap actual content (not just color change)?
- [ ] Hover effect on buttons (color change)?
- [ ] Click press animation on buttons (shrink + bounce)?
- [ ] Open animation plays (scale pop)?
- [ ] Close animation plays (scale down)?

## Reopen Test
- [ ] Close the GUI and reopen. Does everything look identical to first open?
- [ ] No transparency bugs? (title, decor frame, close button all fully visible)

## Responsive
- [ ] All sizes use Scale, not Offset?
- [ ] Would this look correct on a phone half this width?
- [ ] Square elements have UIAspectRatioConstraint?

## Fix and Repeat
If ANY item above fails: fix it, take another screenshot, run the checklist again. Only show the user when EVERY item passes.
