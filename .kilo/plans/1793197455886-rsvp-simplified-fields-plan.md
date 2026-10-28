# Plan: Simplify RSVP Form — Remove Photo Upload and Guest Count

## Context

The previous plan added a photo upload field (Imgur) and kept a guest-count field. The user now wants the opposite: remove the photo upload field entirely and replace it with a plain-text invitation to send a photo via WhatsApp. Also remove the "Cantidad de personas" field.

## Decisions

1. **Remove photo upload field** (`#fPhoto`, preview, upload JS, `imgurClientId`, `uploadToImgur`).
2. **Remove guest-count field** (`#fGuests`, `guestsWrap`, stepper buttons).
   - This affects the RSVP submit handler which builds `👥 Asistiremos: ${count} persona(s).` — this line must be removed.
3. **Add invitation text** in place of photo field: a short `<p>` inviting guests to send a photo with Malena via WhatsApp (e.g., "¿Querés compartir una foto con la quinceañera? Mandá la foto por WhatsApp cuando confirmes tu asistencia.").
4. **WhatsApp number change** (`5493834582464`) remains in scope (unchanged from prior plan).
5. **Graceful degradation**: since photo upload is dropped, no Imgur dependency.

## Risks & Limitations

- Removing guest count changes RSVP message format — confirm text line removal.
- The stepper JS (`data-step`) references `#fGuests` — remove that script too.

## Implementation Tasks

### Task 1: Update WhatsApp number
- File: `index.html`, line 461
- Change `'5493834688070'` → `'5493834582464'`

### Task 2: Remove guest-count markup
- File: `index.html`, lines 390–396 (`#guestsWrap` block including label, stepper buttons, `#fGuests` input)

### Task 3: Remove guest-count JS
- File: `index.html`, `#guestsWrap` observer and `data-step` listener (lines 670–672 and stepper block). Remove or no-op. The `form.attend.value` change listener that hides `guestsWrap` should also drop that hide/show.

### Task 4: Remove photo upload field + replace with invitation text
- File: `index.html`, the `#fPhoto` field block (added in prior plan, now removed) replaced by:
```html
<p class="text-xs text-[color:var(--muted)] mt-1.5">¿Querés compartir una foto con la quinceañera? Mandá la foto por WhatsApp cuando confirmes tu asistencia.</p>
```

### Task 5: Remove Imgur config and JS
- Remove `imgurClientId` from `CONFIG`.
- Remove `uploadToImgur`, `photoInput` change listener, `photoPreview`/`photoName` elements, and `uploadedPhotoUrl`/`photoUploading` variables.

### Task 6: Update submit handler
- File: `index.html`, form submit handler
- Remove the `👥 Asistiremos:` count line construction.
- Remove photo URL append logic.

## Validation

1. Open `index.html` — form shows: name, attendance, diet, message, invitation text, submit.
2. No `#fGuests`, no `#fPhoto` in DOM.
3. WhatsApp button opens `wa.me/5493834582464`.
4. No JS references to removed IDs (no console errors).
5. RSVP text omits guest count line.
