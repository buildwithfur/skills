---
name: download-instagram-carousel
description: Download every image and video slide from a public Instagram carousel in its real slide order. Extract full CDN media URLs from embedded page data instead of saving a cropped Open Graph preview, then verify the downloaded assets.
---

# Download an Instagram carousel

Use this skill for an Instagram `/p/` post with multiple slides. Save the actual carousel assets in slide order, not the post's preview image.

## 1. Inspect the post

Open the post in a browser and dismiss any login dialog that blocks the page. Confirm it is a carousel by navigating to a high slide index:

```text
https://www.instagram.com/p/<shortcode>/?img_index=20
```

Instagram clamps the URL to the highest valid 1-based slide index.

## 2. Extract the true `carousel_media` array

Do not broadly scrape all page image URLs: the parent post's cover can duplicate slide 1 and profile images are unrelated. In the page console, find the embedded script containing `carousel_media`, recursively locate the non-empty array, and retain its child order.

```javascript
const script = [...document.scripts]
  .map(s => s.textContent || '')
  .find(t => t.includes('carousel_media'));
if (!script) throw new Error('No carousel_media script found');

const root = JSON.parse(script);
let media = [];
function walk(x) {
  if (!x || typeof x !== 'object') return;
  if (Array.isArray(x.carousel_media) && x.carousel_media.length) media = x.carousel_media;
  for (const value of Object.values(x)) walk(value);
}
walk(root);

const slides = media.map((m, i) => ({
  index: i + 1,
  id: m.pk,
  type: m.__typename,
  imageUrl: m.image_versions2?.candidates?.[0]?.url,
  videoUrl: m.video_versions?.[0]?.url,
}));
console.log(slides);
```

Prefer `videoUrl` for video slides and `imageUrl` for image slides.

## 3. Transfer signed URLs safely

Long signed Instagram CDN URLs can appear truncated as `...` in browser-console output. Never download a visibly truncated URL. Base64-encode it in the browser, then decode locally:

```javascript
const u = slides[0].imageUrl;
let bytes = '';
for (const b of new TextEncoder().encode(u)) bytes += String.fromCharCode(b);
btoa(bytes);
```

```bash
python3 -c "import base64; print(base64.b64decode('<base64-value>').decode())"
```

Re-extract only failed URLs if some individual downloads receive a 403; signed URLs expire.

## 4. Download in slide order

Use the user's requested destination. Otherwise use a task-local directory such
as `/tmp/instagram-carousel/<post-name>`.

```bash
mkdir -p '<output-dir>'

# Use .jpg for image URLs and .mp4 for video URLs.
curl --fail --location --silent --show-error \
  -o '<output-dir>/slide_01.jpg' \
  '<full-cdn-url>'
```

Name assets `slide_01`, `slide_02`, and so on. Retain the correct extension based on the selected source URL.

## Common pitfalls

- `og:image` is usually a cropped preview, not a full slide.
- `?img_index=` is 1-based.
- Page data may include a parent cover plus carousel children; only use the children in `carousel_media`.
- A carousel may contain video slides. Do not save only their preview frame if the user asked for the media.
- Public extraction can fail on private, deleted, or restricted posts. State that result plainly and do not claim a successful download.

## Verify

```bash
file '<output-dir>'/slide_*
identify '<output-dir>'/slide_*.jpg 2>/dev/null || true
ffprobe -v error -show_entries format=duration -of default=nw=1 \
  '<output-dir>'/slide_*.mp4 2>/dev/null || true
```

Verify the count against the carousel's displayed slide count. Report the output folder, ordered asset list, and any slides that could not be retrieved.
