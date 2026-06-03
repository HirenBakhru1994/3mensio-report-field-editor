# 3mensio Report Field Editor - Project Brief

## Project Goal

Build a single-file `.html` website/app that allows a user to upload an original 3mensio PDF report, enter new report identity details, and generate an updated PDF.

The app should update only these four fields:

- Physician
- Hospital
- City
- Country

The old values must be fully hidden automatically, and the new values must be written in the same report area.

## Core User Flow

1. User opens the HTML file in a browser.
2. User uploads an original 3mensio PDF report.
3. User enters:
   - New Physician Name
   - New Hospital
   - New City
   - New Country
4. User clicks `Generate Updated PDF`.
5. The app creates and downloads an updated PDF.

## Important Requirement

The user must not be able to move, resize, drag, or manually adjust the white cover background.

The white rectangles used to hide old values must be fixed inside the JavaScript code.

The visible UI should only include:

- PDF upload
- Four replacement text fields
- Generate button
- Status/error messages
- Optional read-only PDF preview

Do not add draggable overlays.
Do not add coordinate controls.
Do not add manual edit tools.

## Fields To Replace

The four fields appear on page 1 in the top-right `Report Details` section.

Examples from existing reports:

### Example 1

- Physician: `Dr. Semitko`
- Hospital: `NPCIK`
- City: `Moscow`
- Country: `Russia`

### Example 2

- Physician: `Dr. Epifanov S YU`
- Hospital: `UPD Losinoostrovskay`
- City: `Moscow`
- Country: `Russia`

Only the values should be covered and replaced.

The labels should remain visible:

- `Physician:`
- `Hospital:`
- `City:`
- `Country:`

## Deliverable

Create one file:

```text
3mensio-field-editor.html
```

The file must contain:

- HTML
- CSS
- JavaScript

The app should run locally in the browser without a backend server.

## Recommended Libraries

Use browser CDN libraries.

Recommended:

```html
<script src="https://unpkg.com/pdf-lib/dist/pdf-lib.min.js"></script>
```

Optional for read-only preview:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.0.379/pdf.min.mjs" type="module"></script>
```

Primary PDF modification should be done with `pdf-lib`.

## PDF Editing Logic

Use `pdf-lib` to:

1. Load the uploaded PDF.
2. Open page 1.
3. Draw fixed white rectangles over the old values.
4. Draw the new values over those rectangles.
5. Save the updated PDF.
6. Download the updated PDF in the browser.

## Coordinate Strategy

PDF coordinates start from the bottom-left corner of the page.

Use a fixed internal layout based on a reference A4 page size, then scale the coordinates to the uploaded PDF page size.

Recommended reference size:

```js
const FIELD_LAYOUT = {
  referenceWidth: 595.28,
  referenceHeight: 841.89,
  fields: {
    physician: { x: 455, y: 708, width: 105, height: 13 },
    hospital: { x: 455, y: 696, width: 115, height: 13 },
    city: { x: 455, y: 684, width: 105, height: 13 },
    country: { x: 455, y: 672, width: 105, height: 13 }
  }
};
```

These coordinates are an initial calibration and may need minor adjustment after testing with the sample PDFs.

Keep this coordinate object internal to the code. Do not expose it in the UI.

## Scaling Logic

Use this kind of scaling:

```js
const { width: pageWidth, height: pageHeight } = firstPage.getSize();

const scaleX = pageWidth / FIELD_LAYOUT.referenceWidth;
const scaleY = pageHeight / FIELD_LAYOUT.referenceHeight;

function scaleRect(rect) {
  return {
    x: rect.x * scaleX,
    y: rect.y * scaleY,
    width: rect.width * scaleX,
    height: rect.height * scaleY
  };
}
```

## Covering Old Values

For each field, draw a white rectangle:

```js
firstPage.drawRectangle({
  x,
  y,
  width,
  height,
  color: PDFLib.rgb(1, 1, 1)
});
```

The rectangles should be slightly wider than the old text, so old details are never visible.

Important:

- Cover the value area only.
- Do not cover the labels.
- Do not cover nearby report content.
- Use a solid white fill.

## Drawing New Text

After drawing the white rectangles, write the new text:

```js
firstPage.drawText(value, {
  x: x + 2,
  y: y + 3,
  size: fontSize,
  font,
  color: PDFLib.rgb(0, 0, 0)
});
```

Use:

- Black text
- Helvetica or another standard PDF font
- Font size around 8 to 10 points
- Similar position and size to the original report

## Long Text Handling

If a new value is too long for the field area, the app should reduce the font size slightly.

Example helper:

```js
function calculateFontSize(text, font, maxWidth, preferredSize = 8.5, minimumSize = 6.5) {
  let size = preferredSize;

  while (size > minimumSize && font.widthOfTextAtSize(text, size) > maxWidth) {
    size -= 0.25;
  }

  return size;
}
```

Do not allow new text to overlap other fields.

## File Naming

If the uploaded file is:

```text
report.pdf
```

The output file should be:

```text
report_updated.pdf
```

## Validation

Before generating:

- Require a PDF file.
- Require all four fields to be filled.
- Reject non-PDF files.
- Check that the PDF has at least one page.

Show a clear message if validation fails.

## Status Messages

Show simple status messages such as:

- `PDF loaded`
- `Generating updated PDF...`
- `Updated PDF downloaded`
- `Please upload a PDF file`
- `Please fill all four fields`

Disable the generate button while processing.

## Privacy Requirement

The app should process the PDF locally in the browser.

Do not upload the PDF to any server.

Add this note in the UI:

```text
PDF processing happens locally in your browser.
```

## UI Requirements

The design should be clean, professional, and work-focused.

The first screen should be the actual tool, not a landing page.

Suggested layout:

- Header: `3mensio Report Field Editor`
- Short description: `Upload a report, enter replacement details, and generate an updated PDF.`
- PDF upload area
- Four input fields
- Generate button
- Status/error area
- Optional read-only preview of page 1

Avoid:

- Marketing-style landing page
- Decorative hero sections
- Draggable overlays
- Manual coordinate controls
- Advanced settings visible to the user

## Mobile And Desktop

The app should work on:

- Desktop browsers
- Laptop browsers
- Mobile browsers

Use responsive CSS.

Inputs and buttons should remain readable and usable on smaller screens.

## Suggested Claude Code Prompt

Use this prompt with Claude Code:

```text
Build a complete single-file HTML app named 3mensio-field-editor.html.

The app should let a user upload a 3mensio PDF report, enter new Physician, Hospital, City, and Country values, and generate an updated PDF.

Use pdf-lib through a CDN to modify the PDF in the browser.

The app must edit page 1 only.

The old values are located in the top-right Report Details section. Keep the labels visible and replace only the values.

Draw fixed white rectangles over the old value areas, then draw the new values on top.

The user must not be able to drag, move, resize, or manually adjust the white rectangles. The rectangles must be fixed internally in JavaScript.

Use an internal FIELD_LAYOUT object with reference A4 page dimensions and scaled coordinates:

const FIELD_LAYOUT = {
  referenceWidth: 595.28,
  referenceHeight: 841.89,
  fields: {
    physician: { x: 455, y: 708, width: 105, height: 13 },
    hospital: { x: 455, y: 696, width: 115, height: 13 },
    city: { x: 455, y: 684, width: 105, height: 13 },
    country: { x: 455, y: 672, width: 105, height: 13 }
  }
};

Scale these coordinates based on the actual uploaded PDF page size.

Use solid white rectangles to hide old text completely.

Draw new text in black, using Helvetica, around 8 to 10 pt. If a value is too long, reduce the font size slightly so it fits.

Validate that the user uploads a PDF and fills all four fields.

Show status messages.

Disable the generate button while processing.

Download the final file as originalfilename_updated.pdf.

The app should process everything locally in the browser and should not upload files to any server.

Create a clean professional medical-report utility UI with embedded CSS. The first screen should be the actual tool, not a landing page.

Optional: include a read-only preview of page 1 after upload. Do not include draggable overlays or coordinate adjustment tools.
```

## Testing Checklist

Test with these sample report types:

- Report with short values such as `NPCIK`
- Report with longer hospital value such as `UPD Losinoostrovskay`
- Report with longer physician name
- PDF with missing file upload
- Non-PDF file upload
- Empty input fields

Confirm:

- Old values are fully hidden.
- Labels remain visible.
- New values appear in the correct location.
- No text overlaps.
- Output PDF downloads correctly.
- App works without a backend server.

