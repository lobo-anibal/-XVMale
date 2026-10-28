# Plan: WhatsApp RSVP Number Change + Photo Upload

## Context

Single static HTML file `index.html` for Malena's XV de Oro quinceañera invitation. The RSVP form (`#rsvpForm`, lines 378–410) currently:
- Collects guest name, attendance, guest count, dietary restrictions, optional message
- On submit, opens `wa.me/{CONFIG.whatsapp}?text=...` with pre-filled text (line 693)
- `wa.me` URL scheme supports **text only** — images cannot be sent through it

## Decisions

### 1. WhatsApp Number
- **Current**: `CONFIG.whatsapp = '5493834688070'` (index.html:461)
- **Requested**: `3834582464` — Catamarca landline/mobile format
- **Action**: Set `CONFIG.whatsapp = '5493834582464'`
- Format rationale: `54` (Argentina country code) + `9` (mobile prefix) + `383` (Catamarca area code) + `4582464` (number)

### 2. Photo Upload Strategy
**Recommended: Imgur anonymous upload API**

Since `wa.me` cannot carry images, the approach is:
1. User selects an image file in the RSVP form
2. JavaScript uploads it to Imgur's anonymous upload endpoint (`POST https://api.imgur.com/3/image`)
3. Imgur returns a direct image URL
4. That URL is appended to the WhatsApp message text (as a clickable link)

**Why Imgur:**
- Free, no backend required (client-side upload)
- Simple one-header API: `Authorization: Client-ID {id}`
- Returns `link` field with direct image URL
- Widely accessible from mobile browsers

**Setup required:** User must create a free Imgur account and obtain a Client-ID from `https://api.imgur.com/oauth2/addclient`. This ID only goes in the browser and is safe to expose (it's not a secret).

**Alternative considered:** Form backend services (Formspree, etc.) — rejected because the user explicitly wants WhatsApp integration, not email/form submissions.

### 3. Photo Request Recommendation (Opinion)

**My recommendation: Sí, pero mantelo opcional.**

Requesting a photo of guests with the quinceañera is a lovely idea — it gives Malena content for her album and social media, and creates engagement. However, add friction only if required. Making it optional achieves the benefit (those who have/want to share will) without blocking RSVPs from those who can't.

**UX approach:**
- Label the field clearly: "Foto con Malena (opcional) 📸"
- Add helper text: "Compartí una foto que tengas con la quinceañera y la incluimos en tu mensaje de WhatsApp"
- Show preview thumbnail of selected image
- If upload fails or is skipped, the RSVP still sends via WhatsApp with just the text

## Implementation Tasks

### Task 1: Update WhatsApp number
- File: `index.html`
- Line 461: Change `'5493834688070'` → `'5493834582464'`

### Task 2: Add Imgur Client-ID to CONFIG
- File: `index.html`, line 455 (CONFIG object)
- Add: `imgurClientId: 'TU_CLIENT_ID_AQUI',` — with a comment instructing the user to replace it
- The user must obtain a Client-ID from imgur.com

### Task 3: Add photo upload field to RSVP form
- File: `index.html`, after the "Mensaje para la quinceañera" textarea (line 404), before the submit button (line 406)
- Add:
  ```html
  <div>
    <label class="text-xs uppercase tracking-widest text-[color:var(--muted)]" for="fPhoto">Foto con Malena (opcional)</label>
    <label class="mt-1.5 flex items-center justify-center gap-2 px-4 py-3 border-2 border-dashed border-[color:var(--gold)]/40 rounded-xl text-sm text-[color:var(--muted)] cursor-pointer hover:border-[color:var(--gold)]/70">
      <input id="fPhoto" type="file" accept="image/*" capture="environment" class="hidden">
      <span>📎 Seleccionar foto</span>
    </label>
    <div id="photoPreview" class="mt-2 hidden">
      <img class="w-20 h-20 object-cover rounded-lg" alt="Vista previa">
      <span class="text-xs text-[color:var(--muted)] block mt-1" id="photoName"></span>
    </div>
  </div>
  ```
- `capture="environment"` — opens camera directly on mobile for convenience
- `accept="image/*"` — restricts to images

### Task 4: Add photo upload JavaScript
- File: `index.html`, inside the `<script>` block (after RSVP section, ~line 694)
- Add functions:
  - `uploadToImgur(file)` — POSTs to Imgur API, returns image URL or null
  - Event listener on `#fPhoto` change — validates size (<10MB), shows preview, triggers upload
  - On successful upload, store the URL in a variable to include in WhatsApp message

### Task 5: Update RSVP submit handler to include photo URL
- File: `index.html`, line 672–694 (form submit handler)
- Before opening WhatsApp, if a photo was uploaded and its URL exists:
  - Append to message text: `\n📸 Foto: ${photoUrl}`
- If photo upload is in progress, show loading indicator and wait
- If upload fails, show toast and proceed without photo

### Task 6: Update submit button label/text
- Consider changing "Confirmar por WhatsApp" to "Confirmar y enviar foto por WhatsApp" if photo is present
- Or keep as-is since photo is optional

## Risks & Limitations

1. **Imgur rate limits**: 1250 uploads/day per client ID. Acceptable for a single event.
2. **No image persistence guarantee**: Imgur may remove inactive images. Not recommended for permanent archives.
3. **File size**: Imgur free API limit is 10MB per upload. Add client-side validation.
4. **Mobile UX**: The `capture="environment"` attribute opens the camera on mobile. On desktop it opens the file picker. Good progressive enhancement.
5. **No backend**: This is purely client-side. The WhatsApp message includes the image *link*, the image isn't "attached" to the WhatsApp message itself.

## Validation

1. Open `index.html` in browser
2. Verify WhatsApp button opens `wa.me/5493834582464` (number changed)
3. RSVP form without photo — sends text message as before
4. RSVP with photo — uploads to Imgur, includes URL in WhatsApp text message
5. Upload preview shows correctly
6. Upload failure falls back gracefully (sends text without photo)
7. File size validation works (files >10MB rejected)
8. Works on mobile and desktop browsers

## Out of Scope

- Backend image storage
- Direct WhatsApp image attachment (technically impossible via `wa.me`)
- Multiple photo uploads
- Photo editing/cropping
