# Fake Guard AI setup (Python agents optional)

The browser files live in `frontend/public/`; the Node.js API, local record checks and Gemini fallback live in `backend/`. The optional Python vision and QR/barcode agents live in `backend/agent/`.

## 1. Configure `.env`

Do not replace your `.env`. Only append:

```
GEMINI_API_KEY=paste_your_free_key_here
GEMINI_MODEL=gemini-2.5-flash
GEMINI_TIMEOUT_MS=5500
GEMINI_IMAGE_TIMEOUT_MS=18000
ADMIN_KEY=choose-a-long-random-password
PY_AGENT_URL=
AGENT_TOKEN=
```

Get the free Gemini key at https://aistudio.google.com/apikey (sign in with a Google account, "Create API key").
Make `ADMIN_KEY` long. Generate one with:
`node -e "console.log(require('crypto').randomBytes(24).toString('hex'))"`

## 2. Run

```
npm run seed            # only if you have not already
npm run seed:manual     # optional: adds 4 clearly labelled example notes
npm run dev
```

- Live inspect: http://localhost:3000/inspect
- Manual data (admin): http://localhost:3000/manual (paste your `ADMIN_KEY`)

If `/inspect` shows your old app instead, your service worker is serving a cached page. In Chrome press F12, Application, Service Workers, Unregister, then refresh. To add a menu link, point any button in your app to `/inspect`.

Check it works: `node --test test/inspect.test.js` (11 tests, no database needed).

## 3. What you get

| Feature | How it works |
|---|---|
| Local-first answer | YOLO vision, QR/barcode reading and sourced record matching run first. Gemini is called only when those local checks cannot provide an exact record-backed answer. The Product camera provides a 100-category user-selected hint list, not a trained category model. |
| Automatic | Turn on live auto-check, or let the camera read a barcode (Chrome on Android and some desktops) and it checks by itself. |
| Labels on the picture | Coloured boxes with a name and detail, e.g. `EAN-13 8901234567890`, `Batch A24-0718`, `Seal intact`, `Robot arm`. Dashed boxes = low confidence. |
| Human, robot, classroom | Classroom mode counts people and robots and lists desks, chairs, boards, screens. It never names or identifies anyone. |
| Gemini model | Tries `GEMINI_MODEL` first, then API-supported Flash models and stable fallbacks if the selected model is unavailable or rate-limited. Vision requests use a longer timeout than text requests. |
| Manual data | `/manual`: notes, products, alerts. Notes are given to Gemini and shown under the result. "Ask" answers from your notes and Gemini; with no Gemini key it answers from notes only. |
| Local agents | `backend/agent/`: YOLO vision and a QR/barcode reader. The third local check matches codes and batches to sourced product/alert records. |
| Gemini fallback | If local evidence does not answer the check, Gemini analyses the image. The browser says when this fallback was used. |
| Color preview | An optional in-browser center-area median sRGB estimate is informational only. It is not calibrated and is not a CIELAB/ΔE00 product-color match. |

### Result statuses (same rules as your design brief)

- **LIKELY COUNTERFEIT**: code and batch match an alert you recorded, with a named source.
- **SUSPICIOUS**: the photo shows a concrete problem (one "risk" or two "warn" observations).
- **LIKELY AUTHENTIC**: code and batch match a product record and nothing looks wrong. Still not manufacturer proof.
- **INCONCLUSIVE**: not enough evidence. A missing record never means fake.
- **NOT APPLICABLE**: the picture is a room or person, not a product.
- **VERIFIED** is never produced. It would need a live manufacturer connection, which does not exist yet.

## 4. Python agents and training (optional)

```
cd backend\agent
python -m venv .venv
.venv\Scripts\activate            # Windows PowerShell
pip install -r requirements.txt   # big download (PyTorch)
uvicorn agent_server:app --host 127.0.0.1 --port 8001
```

Then in `.env` set `PY_AGENT_URL=http://127.0.0.1:8001` and restart `npm run dev`.
Without training, the pretrained COCO model detects generic classes such as bottles, phones, backpacks and handbags; COCO has no shoe or brand class and cannot assess product authenticity. The code reader reads QR/barcodes. A specialized packaging or brand detector requires your own reviewed data and trained weights. Gemini provides visual product analysis when local agents cannot answer.

Train your own detector:

1. Put product photos in `backend/agent/dataset/raw/` (sub-folder per session or place). Use many angles, lighting conditions and backgrounds for each labeled product class.
2. `set GEMINI_API_KEY=...` then `python autolabel.py --limit 20` (PowerShell: `$env:GEMINI_API_KEY="..."`). Gemini proposes boxes.
3. **Check the proposals** in `dataset/proposed/preview/`. Fix or delete bad ones with a label tool (labelImg, CVAT, Roboflow). Copy approved files to `dataset/reviewed/images` and `dataset/reviewed/labels`.
4. `python train.py split` then `python train.py train`. Weights are saved to `backend/agent/models/fakeguard_best.pt` and picked up on the next agent restart.

Expect to need a few hundred labelled boxes per class before results are useful. `train.py` prints how many boxes each class has and warns when too few. Training on CPU is slow; Google Colab's free GPU is much faster.

## 5. Privacy and safety notes

- Photos go to the configured local agents first. When local checks cannot provide a sourced record answer, photos are sent to Google Gemini for fallback reading. On the free tier Google may use submitted content to improve its products, so do not send photos you would not want shared. This app does not save the photos.
- Classroom mode: get permission before pointing a camera at students, and check your school's rules. Counts only, no identification.
- `/manual` is protected only by `ADMIN_KEY`. Use a long value, use https in production, and do not share it.
- The Python agent should stay on `127.0.0.1`. If you deploy it, put it on a private network and set `AGENT_TOKEN` in both places.
- Gemini's confidence words (high, medium, low) are its own rating, not a probability that a product is genuine.
- On Render/Railway, the Python agent needs its own service. The Node app works fine without it.
