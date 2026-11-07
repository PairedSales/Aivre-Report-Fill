# Running these workflows in ChatGPT

The workflow files were written for a different assistant's browser tools. Read this before the first browser step. Wherever a workflow names one of the tools below, do what this file says instead. The workflow's intent (what to click, what to enter, what to check) does not change.

Everything here depends on running JavaScript in the page (see AGENTS.md section 1).

## Browser

| The workflow says | Do this |
|---|---|
| Claude in Chrome, the Chrome tools, "read the chrome-browser skill" | Use the appraiser's Chrome connected to Browser Use. Open Aivre in a new tab of your own; do not take over or close the appraiser's tabs. |
| javascript_tool, "inject this helper", "by script", "with JS" | CDP `Runtime.evaluate` on the Aivre tab, with `awaitPromise: true` and `returnByValue: true`. |
| find, find() ref, "click it by its ref" | Locate the element with a DOM query in `Runtime.evaluate` (label text, aria-label, role, button text). Scroll it into view, read its bounding box, then click its center with real mouse events: `Input.dispatchMouseEvent` mousePressed then mouseReleased. Use real clicks wherever a workflow insists on clicking by ref (Add name controls, nav buttons, contact cards). |
| file_upload, "the Chrome upload tool", "attach the file" | See Uploads below. After `A.markUpload(...)`, the target is `input[data-cc-target="1"]`. |
| `/mnt/user-data/uploads/...`, "stage the files", "staged under" | No staging. Put prepared copies in `Scratch/<address-slug>/uploads/` and upload them from there (see Uploads below). Workfile originals can be uploaded directly when no copy is needed. |
| "Windows hard-linked files cannot be staged" | Does not apply. If an upload of an original fails, copy it into `Scratch/<address-slug>/uploads/` and attach the copy. |
| screenshot, zoom | `Page.captureScreenshot`, with a `clip` region to zoom. Take a fresh one before any click by coordinate. |
| get_page_text | `Runtime.evaluate` of `document.body.innerText`. It does not show input values; read those with the helper. |
| "type the text with the keyboard", "select all" | Click into the field with real mouse events, press Ctrl+A with `Input.dispatchKeyEvent` (modifiers 2), then `Input.insertText` with the text, then click blank page area. |
| Escape | `A.esc()` or `G.esc()` first. If a popup stays open, send a real Escape with `Input.dispatchKeyEvent`. |

### Running the helpers

The core's `window.A` helper and the grid's `window.G` helper are injected by evaluating their full source once per page load. They are gone after any navigation or reload; inject again.

Call them inside an async wrapper and return the results so you can check them:

```js
(async () => [
  await A.pick('Assignment Reason', 'Purchase'),
  await A.set('Appraiser Fee', '650'),
  await A.yn('Property on Native American Lands', 'No')
])()
```

The workflows mention three output limits of the earlier tool: text with ";" or "&" was blocked, output was cut off after about 900 characters, and a call timed out at about 45 seconds. They may not apply to CDP, but keep returns short and keep each call to about 20 field sets anyway, so a failure is easy to locate.

Never use `Runtime.evaluate` to call Aivre's hidden APIs or to change data outside the helpers' normal input, change and click events.

## Uploads

Mark the target input first (`A.markUpload('<label>')`, or the selector the workflow gives, such as the TSV dialog's real `input[type=file]`). Then use the first method that works, and remember which one worked for the rest of the chat:

1. **The browser tool's own upload function**, if it has one that takes a local file path.
2. **CDP `DOM.setFileInputFiles`** with the absolute path. Get the node with `DOM.getDocument` and `DOM.querySelector`, or the `objectId` of an element returned by `Runtime.evaluate`. Many ChatGPT browser tools block this command. If it is refused once, do not try it again in that chat.
3. **JavaScript upload.** Needs only `Runtime.evaluate`. Read the file locally, base64 encode it, and send it to the page in pieces with this helper:

```js
window.U = {
 start(name, type){ window.__up = {name, type, parts: []}; return 'ready ' + name; },
 add(chunk){ window.__up.parts.push(chunk); return window.__up.parts.length; },
 attach(sel = 'input[data-cc-target="1"]'){
  const u = window.__up, input = document.querySelector(sel);
  if (!u) return 'nothing started';
  if (!input) return 'no input ' + sel;
  const bin = atob(u.parts.join('')), bytes = new Uint8Array(bin.length);
  for (let i = 0; i < bin.length; i++) bytes[i] = bin.charCodeAt(i);
  const dt = new DataTransfer();
  dt.items.add(new File([bytes], u.name, {type: u.type}));
  input.files = dt.files;
  input.dispatchEvent(new Event('input', {bubbles: true}));
  input.dispatchEvent(new Event('change', {bubbles: true}));
  delete window.__up;
  return 'attached ' + u.name + ' ' + bytes.length + ' bytes';
 }
};
```

   Call `U.start('Front.jpg', 'image/jpeg')`, then `U.add('<base64 piece>')` once per piece of about 500,000 characters, then `U.attach()` (or `U.attach('<selector>')`). Use the right type: `image/jpeg`, `image/png`, `application/pdf`, `text/tab-separated-values` for a TSV, `text/csv` for a CSV. Compress photos first (the workflows say how), so each one is a few pieces at most. Inject `U` again after any reload.

4. **Hand it to the appraiser.** If none of these work, keep filling everything else, and list each file still to upload in the summary with its path, the section and field, and the caption.

After every upload, check that it took: the thumbnail or file name shows in the field, or the TSV dialog moves on to analysis. Then set the caption, as the workflows say.

## Files and local tools

| The workflow says | Do this |
|---|---|
| scratchpad | `Scratch/<address-slug>/`. Create it when the assignment starts. |
| the workfile, the workfile folder | The folder the appraiser provided with the request (see AGENTS.md section 2). |
| "load the X skill with the Skill tool" | Read `workflows/X.md`. |
| `pdftotext -layout`, `pdftoppm`, image compression, contact sheets | Use the local tools available to you (Poppler, Python with Pillow, PyMuPDF or pypdf). If a tool is missing, use another that does the same job. Tell the appraiser only if nothing works. |
| Gmail search for AMC order emails | If a Gmail connector is connected in ChatGPT, search it read only by address and order number. Otherwise use what the workfile has and list the missing order details in the summary. Never send, reply, forward or delete email. |

## Aivre

- Start at https://www.aivre.com/dashboard. Open the existing report for the address by clicking its address text. Never create a report unless the appraiser asks for a new one.
- If the page shows a login or two-factor prompt, stop and ask the appraiser to sign in.
- Leave the report tab open at the end so the appraiser can review it.
