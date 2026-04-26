# Provence itinerary: practical phone upgrade (iPhone + Edge + offline use)

Reviewed for your stated use case on **April 26, 2026**:
- file opened locally on iPhone
- browser: Edge on iOS
- sometimes no Wi-Fi/cellular
- Apple Maps region already downloaded

## Straight answer

Your current itinerary content is good, but to be truly useful in-the-moment you need **offline-first tap actions**.

Most important: replace “read then decide” sections with “tap once and go” controls.

## What to add first (highest impact)

1. **One-tap map buttons per stop**
   - `Open in Apple Maps`
   - `Navigate in Apple Maps`
   - optional fallback: `Copy address`

2. **Sticky quick bar**
   - always visible: `Now`, `Next`, `Park`, `Navigate`

3. **Day quick card at top of each day**
   - arrival target time, parking plan, stop duration, backup plan

4. **Offline essentials block**
   - lodging address, emergency contacts, rental-car return details, booking refs

## Important compatibility notes for your setup

- **Edge on iOS uses Apple WebKit** (same browser engine family as Safari), so Safari/iOS compatibility patterns are the right target.
- Because your file is local and may be offline, avoid any dependency on external CDNs, fonts, frameworks, or APIs.
- Apple Maps links can still open from a local HTML file and leverage downloaded map regions.

## Drop-in HTML/CSS/JS (offline-safe)

Paste this near the end of your `<body>` and adapt the values.

```html
<div id="quickBar" class="quick-bar" aria-label="Trip quick controls">
  <span id="quickNow">Day 2 • Gordes</span>
  <button onclick="openAppleMaps('Place de l\'Église, Gordes, France')">Navigate</button>
  <button onclick="openAppleMaps('Parking du Château, Gordes, France')">Park</button>
  <button onclick="copyText('Place de l\'Église, Gordes, France')">Copy Address</button>
</div>

<section class="day-card">
  <h3>Day 2 Quick Card</h3>
  <ul>
    <li><strong>Best arrival:</strong> 08:30</li>
    <li><strong>Parking:</strong> Parking du Château (8 min walk)</li>
    <li><strong>Time on site:</strong> 90 min</li>
    <li><strong>Plan B:</strong> If crowded, go to Sénanque first</li>
  </ul>
</section>

<script>
  function openAppleMaps(query) {
    const q = encodeURIComponent(query);
    // Works from local HTML and opens Apple Maps app.
    window.location.href = `https://maps.apple.com/?q=${q}`;
  }

  async function copyText(value) {
    try {
      if (navigator.clipboard && window.isSecureContext) {
        await navigator.clipboard.writeText(value);
      } else {
        const ta = document.createElement('textarea');
        ta.value = value;
        document.body.appendChild(ta);
        ta.select();
        document.execCommand('copy');
        ta.remove();
      }
      alert('Address copied');
    } catch {
      alert('Copy failed — long press and copy manually.');
    }
  }
</script>
```

```css
:root {
  --bg: #ffffff;
  --text: #111111;
  --accent: #0a84ff;
}
body {
  margin: 0;
  font-size: 16px;
  line-height: 1.45;
  background: var(--bg);
  color: var(--text);
}
.quick-bar {
  position: sticky;
  top: 0;
  z-index: 999;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  align-items: center;
  padding: 10px;
  background: #fff;
  border-bottom: 1px solid #ddd;
}
.quick-bar button {
  min-height: 44px;
  min-width: 44px;
  border: 0;
  border-radius: 10px;
  background: var(--accent);
  color: #fff;
  padding: 0 12px;
  font-size: 16px;
}
.day-card {
  margin: 12px;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 12px;
  background: #fafafa;
}
@media (max-width: 430px) {
  .quick-bar { padding: 8px; gap: 6px; }
  .quick-bar button { flex: 1 1 auto; }
}
```

Also confirm your `<head>` includes:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="format-detection" content="telephone=no">
```

## Recommended data model per stop (short)

For each stop entry, keep these fields in a compact block:
- label
- address string
- parking string
- arrival window
- stay minutes
- plan-B stop

## Final practical checklist before your trip

- Open file locally in Edge while airplane mode is ON.
- Tap every `Navigate` and `Park` button once.
- Verify each route opens Apple Maps with the right pin.
- Verify text remains readable in bright light (high contrast).
- Keep all buttons at least 44px tall.

If this passes, your itinerary will feel much smoother on the ground.
