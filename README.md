# CropCare AI

A specialized crop-health monitoring system, not a general chatbot: turns a
leaf photo into a disease name, a severity percentage, a field-level risk
call, and a recommendation — and remembers every scan so it can show trends
over time.

## Quick start

```bash
pip install -r requirements.txt
python database.py        # creates cropcare.db and seeds the Field A demo data
streamlit run app.py       # opens the dashboard in your browser
```

The app runs immediately in **demo mode** even without a trained model —
`model_inference.py` falls back to realistic simulated predictions so you
can build and demo the rest of the app (tracking, trends, risk) before the
classifier is trained. Train the real model whenever you're ready:

```bash
# get the PlantVillage dataset from Kaggle first — see model_train.py header
python model_train.py
# copy the resulting model/tomato_disease_model.h5 — app.py picks it up automatically
```

## Architecture

```
                 CropCare AI
                     |
        +------------+------------+
        |                         |
   Vision Model              Farm Database
   (model_inference.py)      (database.py)
        |                         |
 Disease + Severity        History + Trends
   (severity.py)                  |
        +------------+------------+
                     |
              Decision Engine
              (risk_engine.py)
                     |
             Farmer Dashboard
                  (app.py)
```

## What's built now (Phase 1 — this repo)

| Feature from the pitch | Status | File |
|---|---|---|
| Detect (crop + disease + confidence) | ✅ Working (demo mode until trained) | `model_inference.py` |
| Measure (severity as % leaf area) | ✅ Working, tested against a synthetic image | `severity.py` |
| Track (per-plant/field scan history, weekly trend) | ✅ Working | `database.py` |
| Field-level risk decision | ✅ Working | `risk_engine.py` |
| Farmer-specific longitudinal history | ✅ Working (it's what `database.py` stores) | `database.py` |
| Dashboard tying it together | ✅ Working | `app.py` |

## Roadmap (Phase 2 — pitch it, build it after the MVP works)

These are real, buildable features — just bigger than a hackathon weekend,
and worth being upfront about rather than claiming they're done:

- **Regional disease map.** Needs multi-user accounts and a shared backend
  (not just one local SQLite file) so many farmers' scans can be aggregated
  by district. Once `scans` live in a shared database, the query itself is
  simple: group by district + week, compare against the prior week.
- **True offline on-device inference.** `model_train.py` includes a
  `convert_to_tflite()` step — MobileNetV2 was chosen specifically because
  it converts cleanly to TensorFlow Lite for a phone app. What's not built
  yet is the actual mobile app (Flutter/Kotlin) and the local-then-sync
  logic; today's `app.py` is a server-side Streamlit dashboard, not an
  on-device app.
- **AI assistant layer.** The structured output (`disease`, `severity_pct`,
  risk `level`, `reasons`) is already exactly the input an LLM call would
  need to generate natural-language, farmer-friendly explanations — that's
  a small addition (one function that formats a prompt from this data) once
  the core pipeline above is solid.

## How severity actually works

`severity.py` uses HSV colour segmentation, not a trained model: it masks
out "healthy green" and "diseased brown/dark" pixel ranges and divides one
by the other. It's a real, standard plant-pathology image-processing
technique — and it's honest to say the HSV thresholds are starting points,
not calibrated against your specific dataset yet. If a judge asks about
accuracy: tune the ranges in `severity.py` against a handful of your own
sample photos before the demo, and mention that a photo taken against a
plain background (not soil or a patterned surface) is what the current
version assumes — background segmentation is a documented improvement, not
a hidden flaw.

## Notes

- Default crop scope is tomato (matches the pitch's leaf-spot example).
  Extending to other crops means retraining `model_train.py` on more
  PlantVillage folders and extending `CLASS_NAMES` in `model_inference.py`.
- `database.seed_demo_data()` reproduces the exact "Field A" numbers from
  the pitch (20 plants, 12 healthy / 6 Early Blight / 2 Septoria, weekly
  average severity 4% → 9% → 14%) so the dashboard has something real to
  show the moment you open it.
