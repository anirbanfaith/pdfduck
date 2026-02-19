# pdfduck

> bulk pdfs to structured csv

---

## what it does

drop PDFs (shipping bills, invoices, BoL, any tabular documents) and get clean, structured CSV output. 

**backend** extracts tables using pdfplumber. **frontend** displays results and downloads CSV. zero AI API costs. no gemini, no openai — pure python pdf parsing.

---

## architecture

```
Frontend (Cloudflare Pages)
  → uploads PDF via POST
  → displays preview table
  → downloads CSV

Backend (Railway - Python FastAPI)
  → receives PDF
  → extracts tables with pdfplumber
  → returns structured JSON
  → falls back to text pattern matching if no tables
```

---

## how it works

**extraction logic:**

1. **pdfplumber** opens the PDF and scans for tables
2. first row of each table = column headers
3. remaining rows = data rows
4. all rows from all tables are merged into a single JSON array
5. if no tables found, it falls back to regex pattern matching for common fields:
   - invoice number, date, amounts, consignee, shipper, email, phone, etc.
6. if still minimal data, it extracts line-by-line (useful for resumes, general docs)

**what works best:**

- PDFs with clear tabular structure (bordered or grid-based tables)
- invoices, shipping bills, BoL with line-item tables
- multi-page documents with consistent table formats

**what doesn't work:**

- heavily scanned/skewed PDFs (poor OCR quality)
- PDFs with complex multi-column layouts that aren't tables
- pure image PDFs without embedded text

---

## project structure

```
.
├── backend/
│   ├── main.py              
│   ├── requirements.txt     
│   ├── Procfile (optional)
│   ├── railway.json         
│   └── README.md            
│
├── index.html               
├── favicon.ico              
├── build.sh (injects env vars)
└── README.md
```

---

## features

✓ bulk upload (multiple PDFs at once)  
✓ per-file status tracking (pending → reading → done/error)  
✓ live progress bar  
✓ preview table before download  
✓ CSV generation (client-side, zero server storage)  
✓ copy to clipboard  
✓ mobile responsive  
✓ no AI API costs — pure PDF parsing  
✓ secure backend URL via environment variables  

---

## api reference

### POST /extract

upload a PDF, get structured JSON back.

**request:**

```bash
curl -X POST http://your-backend/extract \
  -F "file=@document.pdf"
```

**response (success - tables found):**

```json
{
  "success": true,
  "method": "table_extraction",
  "rows": 15,
  "data": [
    {
      "item": "Widget A",
      "quantity": "100",
      "price": "50.00",
      "_source_page": 1,
      "_source_table": 1,
      "_source_row": 1
    }
  ]
}
```

**response (success - text extraction):**

```json
{
  "success": true,
  "method": "text_extraction",
  "rows": 1,
  "data": [
    {
      "name": "John Doe",
      "email": "john@example.com",
      "phone": "+1-555-0100",
      "line_1": "Software Engineer",
      "_full_text": "...",
      "_text_length": 1523
    }
  ]
}
```

---

## contributing

open source. do what you want. PRs welcome.

---

## license

mit