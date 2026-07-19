# Kindle Home Assistant Remote

Turn a 6th-gen Kindle Paperwhite into a low-power, touch-interactive
Home Assistant control panel. No proxy server, no Docker, no persistent
connection — just one static HTML file served directly from Home Assistant.

## Prerequisites

- Kindle Paperwhite 2 (6th gen)
- Home Assistant instance on the same local network
- `kindle-dashboard.html` (the dashboard file)

---

## Part 1: Get the dashboard running

### 1. Generate a Home Assistant long-lived access token

1. In Home Assistant, click your profile (bottom left).
2. Go to **Security** tab.
3. Under **Long-Lived Access Tokens**, click **Create Token**.
4. Name it (e.g. `kindle-dashboard`) and copy the token — you won't be able to see it again.

### 2. Create the `www` folder (if it doesn't exist yet)

Home Assistant does **not** create this folder automatically.

1. Open the **File editor** (or **Studio Code Server**) add-on.
2. At the root level — same level as `configuration.yaml` — create a new folder named exactly `www`.

### 3. Add the dashboard file

1. Inside `www`, create a new file named `kindle-dashboard.html`.
2. Paste in the full contents of the dashboard file.
3. Near the top of the `<script>` section, set:
   - `HA_TOKEN` — paste the token from step 1.
   - `HA_URL` — leave as `""` (empty string). This only needs to change if you host the file somewhere other than HA's own `www` folder.
4. In the `GROUPS` object, edit the entity list to match your own lights/switches/covers (see **Customizing** below).
5. Save the file.

### 4. Test it on a computer or phone first

Visit `http://YOUR-HA-IP:8123/local/kindle-dashboard.html` in a normal browser
on the same network (e.g. `http://192.168.0.118:8123/local/kindle-dashboard.html`).

- **404 error** → the `www` folder or filename is wrong, or Home Assistant needs a restart to pick up a brand-new `www` folder.
- **Buttons show up but state stays "?"** → check the token was pasted correctly and entity IDs match yours exactly.
- **Loads and toggles fine** → you're ready for the Kindle.

### 5. Open it on the Kindle

1. From the Kindle home screen, tap **Menu → Experimental Browser**.
2. Type the same URL from step 4 (make sure it starts with `http://`).
3. Tap a button to toggle a light/switch/blinds — the state updates a moment later.

This is enough to have a working interactive dashboard. The Kindle sleeps
normally when idle and wakes on tap, so battery drain is minimal — nothing
holds a connection open in the background.

---

## Customizing the dashboard

Entities are grouped by section in the `GROUPS` object near the top of the
`<script>` block:

```js
var GROUPS = {
  "group-living-room": [
    { id: "light.living_room_main_light", name: "Main Light" },
    { id: "switch.0x348d13fffec8ac9f", name: "Lamp" },
    { id: "cover.living_room_blinds", name: "Blinds" }
  ],
  "group-kitchen": [
    { id: "light.main_kitchen_lights", name: "Main Kitchen Lights" }
  ]
};
```

- `id` must be the exact Home Assistant entity ID (find it in **Settings → Devices & Services → Entities**).
- `name` is whatever label you want shown on the button.
- Supported domains out of the box: `light`, `switch`, `cover` (blinds) — each gets its own icon and correct on/off (or open/closed) labeling automatically.
- Adding a new section: add a new key to `GROUPS`, and a matching `<div id="group-yourkey"></div>` plus a `<div class="section-title">Your Title</div>` in the HTML body above the script.
- Adding a new domain (e.g. `fan`, `lock`): add an entry to both `DOMAIN_META` and `ICONS` — ask if you'd like one added.

## Troubleshooting quick reference

| Symptom | Likely cause |
|---|---|
| 404 on any device | File/folder path wrong, or `www` folder needs a HA restart |
| 404 on Kindle only, works elsewhere | Kindle-specific network/typing issue, not the file |
| Blank white page | Server/file issue if blank everywhere; old-browser JS issue if blank only on Kindle |
| Buttons load but state is "?" | Bad/missing token, or entity ID typo |
| Kindle browser shows chrome/URL bar | Use WebLaunch (Part 2) i
