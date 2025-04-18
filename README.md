# Simple-pdf-extractor 🐍📝
    ├── data/                # 🆕 Pollinator PDF auto‑downloaded here
    ├── output/              # 📝 extraction.csv appears here
    ├── extract.py           # 🐍 Python script
    ├── requirements.txt     # 📦 Dependencies
    └── README.md            # 📖 This doc

# !/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
🚀 simple-pdf-extractor

Downloads a Colombian pollinator PDF, then extracts:
- Scientific names (Genus species)
- Locations (LOC/GPE)
- Dates (DATE)
and writes them to output/extraction.csv

# 1️⃣ Download a Colombian PDF if not present
    import os, glob, csv, langid, requests
    import fitz                    # PyMuPDF
    import spacy
    from spacy.matcher import Matcher
    from datetime import datetime

    PDF_URL  = "https://archivo.minambiente.gov.co/images/BosquesBiodiversidadyServiciosEcosistemicos/pdf/fauna_flora/PLAN_DE_ACCIÓN_INICIATIVA_COLOMBIANA_DE_POLINIZADORES.pdf"
    PDF_PATH = "data/polinizadores_co.pdf"
    os.makedirs("data", exist_ok=True)
    if not os.path.exists(PDF_PATH):
    print("🌐 Downloading Colombian pollinator PDF…")
    r = requests.get(PDF_URL)
    r.raise_for_status()
    with open(PDF_PATH, "wb") as f:
        f.write(r.content)

# 2️⃣ Load Spanish NLP model & prepare matcher
    print("🧠 Loading spaCy model…")
    nlp = spacy.load("es_core_news_sm")

    matcher = Matcher(nlp.vocab)
    matcher.add("SPECIES", [[
    {"TEXT": {"REGEX": "^[A-Z][a-z]+$"}},
    {"TEXT": {"REGEX": "^[a-z]+$"}}
    ]])

# 3️⃣ Extract from all PDFs in data/
    OUT_DIR   = "output"
    CSV_PATH  = os.path.join(OUT_DIR, "extraction.csv")
    os.makedirs(OUT_DIR, exist_ok=True)

    with open(CSV_PATH, "w", newline="", encoding="utf-8") as csvfile:
    writer = csv.writer(csvfile)
    writer.writerow(["file","timestamp","species","places","dates"])

    for pdf_file in glob.glob("data/*.pdf"):
        print(f"📄 Processing {pdf_file} …")
        doc = fitz.open(pdf_file)
        text = "".join(page.get_text() for page in doc)
        doc.close()

        # Language detection
        lang = langid.classify(text[:1000])[0]
        # (We only have Spanish model, but you can add English if needed)
        
        # NLP & matching
        doc_sp = nlp(text)
        species = {doc_sp[s:e].text for _, s, e in matcher(doc_sp)}
        places  = {ent.text for ent in doc_sp.ents if ent.label_ in ("LOC","GPE")}
        dates   = {ent.text for ent in doc_sp.ents if ent.label_ == "DATE"}

        # Write row
        writer.writerow([
            os.path.basename(pdf_file),
            datetime.utcnow().isoformat(),
            ";".join(sorted(species)),
            ";".join(sorted(places)),
            ";".join(sorted(dates))
        ])
    print(f"✅ Done! Results saved to {CSV_PATH}")
