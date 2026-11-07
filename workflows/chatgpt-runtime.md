# Running these workflows in ChatGPT

The workflow files were written for a different assistant's browser tools. Read this before the first browser step. Wherever a workflow names one of the tools below, do what this file says instead. The workflow's intent (what to click, what to enter, what to check) does not change.

Full CDP access must be on (see AGENTS.md section 1). Everything here depends on it.

## Browser

| The workflow says | Do this |
|---|---|
| Claude in Chrome, the Chrome tools, "read the chrome-browser skill" | Use the appraiser's Chrome connected to Browser Use. Open Aivre in a new tab of your own; do not take over or close the appraiser's tabs. |
| javascript_tool, "inject this helper", "by script", "with JS" | CDP `Runtime.evaluate` on the Aivre tab, with `awaitPromise: true` and `returnByValue: true`. |
| find, find() ref, "click it by its ref" | Locate the element with a DOM query in `Runtime.evaluate` (label text, aria-label, role, button text). Scroll it into view, read its bounding box, then click its center with real mouse events: `Input.dispatchMouseEvent` mousePressed then mouseReleased. Use real clicks wherever a workflow insists on clicking by ref (Add name controls, nav buttons, contact cards). |
| file_upload, "the Chrome upload tool" | CDP `DOM.setFileInputFiles` with absolute local file paths. Get the node with `DOM.getDocument` then `DOM.querySelector`, or pass the `objectId` of an element returned by `Runtime.evaluate`. After `A.markUpload(...)`, the target is `input[data-cc-target="1"]`. |
| `/mnt/user-data/uploads/...`, "stage the files", "staged under" | No staging. Put prepared copies in `Scratch/<address-slug>/uploads/` and pass their absolute paths. Workfile originals can be attached directly by absolute path when no copy is needed. |
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

## Files and local tools

| The workflow says | Do this |
|---|---|
| scratchpad | `Scratch/<address-slug>/`. Create it when the assignment starts. |
| the workfile, the workfile folder | `Workfiles/<address-slug>/` (see AGENTS.md section 2). |
| "load the X skill with the Skill tool" | Read `workflows/X.md`. |
| `pdftotext -layout`, `pdftoppm`, image compression, contact sheets | Use the local tools available to you (Poppler, Python with Pillow, PyMuPDF or pypdf). If a tool is missing, use another that does the same job. Tell the appraiser only if nothing works. |
| Gmail search for AMC order emails | If a Gmail connector is connected in ChatGPT, search it read only by address and order number. Otherwise use what the workfile has and list the missing order details in the summary. Never send, reply, forward or delete email. |

## Aivre

- Start at https://www.aivre.com/dashboard. Open the existing report for the address by clicking its address text. Never create a report unless the appraiser asks for a new one.
- If the page shows a login or two-factor prompt, stop and ask the appraiser to sign in.
- Leave the report tab open at the end so the appraiser can review it.
