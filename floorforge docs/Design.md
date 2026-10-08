# Design.md: Design System

*Stop your app from looking like every other AI-built app.*

## Colour and Theme

Dark theme by default (calm, technical, lets the 3D model stand out). A light theme is a nice-to-have.

| Role | Hex | Use |
|---|---|---|
| Background | `#0F172A` | App background |
| Surface | `#1E293B` | Cards, sidebar, panels |
| Border | `#334155` | Dividers, table lines |
| Text primary / muted | `#F1F5F9` / `#94A3B8` | Headings and body / secondary text |
| Accent (teal) | `#14B8A6` | Primary buttons, active toggles, links |
| Door | `#F59E0B` | Doors in overlay and 3D |
| Window | `#38BDF8` | Windows in overlay and 3D |
| Wall / floor (3D) | `#E2E8F0` / `#64748B` | Wall material / floor material |
| Confidence high / med / low | `#22C55E` / `#F59E0B` / `#EF4444` | Overlay and warning badges |

## Fonts and Typography

| Element | Font | Size / weight |
|---|---|---|
| App title | Inter (Google Fonts) | 24 px / 700 |
| Section headings | Inter | 16 px / 600 |
| Body, labels | Inter | 14 px / 400 |
| Numbers, dimensions, JSON | JetBrains Mono | 13 px / 500 |

## Layout and Components

- **Screen 1:** centred upload card (dashed border, teal on hover), sample plans as three thumbnails below, one primary button.
- **Screen 2:** two equal panels. Left = plan image with overlay toggle chips (Mask, Vectors, Confidence). Right = 3D viewer. Right sidebar (320 px) = scale badge, rooms table, warnings list, Download GLB.
- **3D viewer:** dark gradient background (`#0F172A` to `#1E293B`), soft directional light plus ambient light, subtle ground grid, orbit with damping, auto-fit camera on load.
- **Badges:** scale method pill ("OCR", "Estimated", "Manual"), confidence dots, warning chips with icon and short text.
- **Spacing:** 8 px grid, 12 px card radius, no heavy shadows. **Motion:** only for progress steps and camera fit (200–300 ms).

## Reference

Calm, dense-but-clean dark UI in the style of Linear; 3D viewer clarity in the style of Planner 5D / Floorplanner. Each person should screenshot one reference they like and paste it into the AI tool along with these tokens.

> **Tip:** A vague design prompt gets the statistical average of every app ever made. Always include the hex codes above.
