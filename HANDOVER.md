# Handover: Hanshith Ram's 2nd Birthday invite page

## 1. Objective

Build a single static web page that guests open from a WhatsApp link. The page shows the invitation, opens the venue in Google Maps, and adds the event to the guest's calendar on Android (Google Calendar) and iPhone (Apple Calendar). Host it on GitHub Pages.

The page must load fast on mid-range Android phones over mobile data, and work inside the WhatsApp in-app browser on Android and in Safari on iPhone.

## 2. Starting assets

Unzip `hanshith-birthday.zip` into the project folder. It contains:

| File | Purpose | Action |
|---|---|---|
| `invite.jpg` | Invite artwork, 1024 x 1536, 381 KB | Keep as the source image. Produce optimised versions (section 6). |
| `Hanshith_Birthday.ics` | Calendar file with reminders | Keep unchanged unless event details change (section 5). |
| `index.html` | Working first version with the three buttons | Use as the functional reference. Rebuild the design and motion. |

## 3. Event details (source of truth)

| Field | Value |
|---|---|
| Title | Hanshith Ram's 2nd Birthday |
| Date | Saturday, 24 October 2026 |
| Start | 11:00 AM IST |
| End (calendar only) | 2:00 PM IST. The invite says "11:00 AM onwards"; do not show an end time on the page. |
| Venue | Signature Dine, 3rd Line, Brindavan Gardens, Guntur |
| Map link | https://maps.app.goo.gl/awhavZrJzygwVywA7 |
| Sign-off | With regards, Pavan Kumar Paruchuri |
| Hosts named on the invite | Paruchuri & Meka family |

## 4. Required page content and behaviour

1. Invite image as the main element of the page, with descriptive `alt` text covering name, date, time and venue.
2. Event title, date and "11:00 AM onwards" as live text below the image.
3. **Get directions** button. Opens the map link in a new tab.
4. **Add to Google Calendar** button. Uses this exact URL:

```
https://calendar.google.com/calendar/render?action=TEMPLATE&text=Hanshith%20Ram%27s%202nd%20Birthday&dates=20261024T110000%2F20261024T140000&ctz=Asia%2FKolkata&location=Signature%20Dine%2C%203rd%20Line%2C%20Brindavan%20Gardens%2C%20Guntur&details=Hanshith%20Ram%27s%202nd%20birthday%20celebration%2C%2011%3A00%20AM%20onwards.%0AMap%3A%20https%3A%2F%2Fmaps.app.goo.gl%2FawhavZrJzygwVywA7%0A%0AWith%20regards%2C%0APavan%20Kumar%20Paruchuri
```

   In HTML attributes, encode `&` as `&amp;`.
5. **Add to Apple Calendar** button. A plain relative link to `Hanshith_Birthday.ics`. Do not use `webcal://`, a `data:` URI or a JavaScript download. iPhone Safari shows the Add to Calendar sheet only for a real `https` file served as `text/calendar`, which GitHub Pages does.
6. Platform ordering. On iPhone or iPad, show the Apple button above the Google button. On all other devices, Google first. Detect with the user agent, and treat `MacIntel` with `maxTouchPoints > 1` as iPad. Both buttons stay visible on every device.
7. A one-line note that the calendar entry includes reminders one day before and two hours before.
8. Countdown to 11:00 AM IST on 24 October 2026 in days, hours and minutes. Compute against the fixed instant `2026-10-24T05:30:00Z`, not the device time zone.
9. After the start time, replace the countdown with a short line stating the celebration has started. After 25 October 2026, hide the calendar buttons and show a thank-you line. Keep the map button.
10. Sign-off at the bottom.

## 5. Calendar file specification

The supplied `Hanshith_Birthday.ics` already meets this. Regenerate only if details change.

- CRLF line endings, `VERSION:2.0`, `METHOD:PUBLISH`.
- Times in UTC: `DTSTART:20261024T053000Z`, `DTEND:20261024T083000Z`.
- Stable `UID`. If the event changes, keep the UID and update `DTSTAMP` so re-imports update the same entry.
- Escape commas in `LOCATION` and `DESCRIPTION` with `\,`.
- Two `VALARM` blocks with `ACTION:DISPLAY`, triggers `-P1D` and `-PT2H`.
- Filename must match the link exactly. GitHub Pages is case-sensitive.

## 6. WhatsApp link preview (required fix)

The current `og:image` points to `invite.jpg` at 381 KB. WhatsApp often drops preview images above about 300 KB, so the preview may show without the image.

- Create `og-image.jpg`, 1200 x 630, under 200 KB. Crop or recompose from the invite so the name and "2nd Birthday" are readable at thumbnail size.
- Set `og:title`, `og:description`, `og:image` (absolute URL), `og:image:width`, `og:image:height`, `og:type`, and the matching `twitter:card` tags.
- Absolute base URL: `https://capavankumar.github.io/hanshith-birthday/`
- For display on the page, serve `invite.webp` (under 150 KB) with `invite.jpg` as fallback via `<picture>`. Set `width` and `height` to prevent layout shift.

## 7. Visual direction

Follow the invite artwork. It is a hand-drawn children's racing theme: chequered flags, bunting in red, yellow and blue, a red race car numbered 2, a traffic light, warm paper background.

- Palette drawn from the invite: navy `#1B2F6B`, red `#D62828`, yellow `#F2B705`, blue `#2F6FD1`, paper `#FFF8E8`.
- One rounded, friendly display typeface from Google Fonts (for example Baloo 2 or Fredoka) with a system fallback stack. Do not load more than two weights.
- Chequered and bunting motifs as CSS or inline SVG, not extra images.
- Mobile first, single column, max width about 460 px, centred on larger screens.
- Light and dark mode via CSS variables and `prefers-color-scheme`. The invite image stays as is in both.
- Respect safe-area insets (`viewport-fit=cover` and `env(safe-area-inset-*)`).

## 8. Motion and transitions

Plan one orchestrated load sequence and responsive feedback on taps. Total load sequence under 1.8 seconds. Content must be readable and tappable before the animation finishes.

1. **Start lights.** A small traffic light (inline SVG) lights red, then yellow, then green over about 0.9 s.
2. **Car pass.** On green, a small red car SVG with number 2 drives across the top of the page and leaves a chequered flag strip behind it.
3. **Card reveal.** The invite image rises and settles with a slight overshoot.
4. **Buttons.** The three buttons enter in a short stagger, 60 to 80 ms apart.
5. **Tap feedback.** Buttons compress slightly on press. After a calendar button is tapped, show a brief confirmation (small chequered-flag burst or confetti in the palette colours) and a line such as "Calendar opened. Save the event to get reminders."
6. **Countdown.** Digits change with a short vertical roll, not a full-page effect.

Constraints:
- Animate only `transform` and `opacity`. No layout-triggering properties.
- Under `prefers-reduced-motion: reduce`, skip the start lights and car pass, show everything immediately, and keep only an instant state change for tap feedback.
- No animation library is required. If one is used, load a pinned version from a CDN and keep it under 30 KB.
- The intro plays once per visit. Do not replay on scroll.
- A tap anywhere during the intro skips to the final state.

## 9. Technical constraints

- Plain HTML, CSS and JavaScript. No build step, so GitHub Pages serves the repository as is.
- Total page weight under 600 KB including images and fonts.
- No analytics, trackers, cookies or external scripts other than Google Fonts.
- Minimum tap target 48 px. Visible focus outlines. Colour contrast meets WCAG AA for text.
- Do not use `localStorage` for anything required. The page must work with storage blocked.
- Test that links open correctly from the Android WhatsApp in-app browser, which may block `target="_blank"` popups in some versions. The links must still work when opened in the same tab.

## 10. Repository and deployment

- Repository: `capavankumar/hanshith-birthday`, public.
- Files at the repository root: `index.html`, `Hanshith_Birthday.ics`, `invite.jpg`, `invite.webp`, `og-image.jpg`, plus any SVG files if not inlined.
- Add an empty `.nojekyll` file.
- Enable GitHub Pages from branch `main`, folder `/ (root)`.
- Live URL: `https://capavankumar.github.io/hanshith-birthday/`
- If the GitHub CLI is authenticated, Claude Code may create the repository and enable Pages with `gh`. Otherwise, prepare the commit and give Pavan the manual steps.

## 11. Acceptance checklist

1. `curl -I https://capavankumar.github.io/hanshith-birthday/Hanshith_Birthday.ics` returns `Content-Type: text/calendar`.
2. iPhone, opened from WhatsApp: Apple button shows above Google; tapping it shows the Add to Calendar sheet with two alerts.
3. Android, opened from WhatsApp: Google button opens Google Calendar with title, date, 11:00 AM to 2:00 PM IST, location and description filled in.
4. Map button opens the correct venue on both platforms.
5. WhatsApp link preview shows the image, title and description. Re-check after any change to `og-image.jpg`, since WhatsApp caches previews.
6. Countdown shows the correct value on a device set to a time zone other than IST.
7. Reduced-motion setting on the phone removes the intro animation.
8. Lighthouse mobile performance score of 90 or above.

## 12. WhatsApp caption to send with the link

```
We would be delighted to have you join us for Hanshith Ram's 2nd birthday.

Location and add to calendar:
https://capavankumar.github.io/hanshith-birthday/

With regards,
Pavan Kumar Paruchuri
```
