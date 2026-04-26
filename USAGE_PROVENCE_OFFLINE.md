# How to use `provence_itinerary_offline.html` on iPhone (Edge, offline)

## 1) Locate the file

In this repo, the itinerary file is:

- `provence_itinerary_offline.html`

## 2) Put it on your iPhone

Use one of these:

- AirDrop the file to your iPhone and save to **Files**.
- Email it to yourself and use **Share -> Save to Files**.
- iCloud Drive: copy file to iCloud folder, then open from Files app on iPhone.

## 3) Open it in Edge

1. Open **Files** app.
2. Browse to the saved `provence_itinerary_offline.html`.
3. Long-press file -> **Share** -> **Edge**.
4. Edge opens the local HTML page.

## 4) Offline check (recommended before trip)

1. Turn on **Airplane Mode**.
2. Open the file again from Files -> Edge.
3. Tap:
   - `Navigate`
   - `Park`
   - `Open Market`
4. Confirm Apple Maps launches and uses downloaded region.

## 5) Day-to-day usage

- Use top bar buttons for immediate actions.
- Use **Must-hit markets tracker** checkboxes; state is saved on device with local storage key:
  - `provence_market_checks_v1`
- Use weekday market filter to focus only today/tomorrow.

## 6) Customize your content

Edit `TRIP` object near top of script in the HTML:

- dates (`startDate`, `endDate`)
- markets list
- per-day stops
- lodging/emergency details

Then resend the updated file to your iPhone.

## Troubleshooting

- If copy button fails on local file context, use long-press on address text and copy manually.
- If map route looks wrong, adjust address string to a fuller address (street + town + postal code).
- If checklist seems reset, check if browser/site data was cleared.
