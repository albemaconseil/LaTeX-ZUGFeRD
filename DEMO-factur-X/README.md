# DEMO-factur-X

French invoice in the **Factur-X** format (profile EN 16931), created with the
`zugferd` package and the demo wrapper `zugferd-invoice`.
All data (companies, persons, IBAN, ...) is fictional.

## Files

| File | Purpose |
|------|---------|
| `DEMO-facture-factur-x.tex` | Main file: invoice number, date, items, layout |
| `DEMO-facture-vendeur.tex` | Seller data, legal notices, bank details, page footer |
| `DEMO-facture-acheteur.tex` | Buyer data |

## Compile

Use a current LaTeX distribution (the file starts with `\DocumentMetadata`) and run it twice:

```
pdflatex DEMO-facture-factur-x.tex
```

The result of pdfLaTeX passed the FNFE-MPE validator (Factur-X, EN 16931 profile). LuaLaTeX, which the other demo files use, has not been tested with this file.

The result is a PDF/A-3b file with the embedded `factur-x.xml`.

## What is specific to France

The demo uses `format=facturx` (alias for `format=en16931`). Compared to the
`xrechnung` formats this

* writes the SIREN as legal identifier with the scheme `0002` (`seller/legal-id`, `buyer/legal-id`),
* does not write the Peppol business process (set `business-process` if your platform requires one, e.g. `S1`),
* does not write an empty `buyer/reference`,
* provides `note-AAB`, `note-PMD` and `note-PMT` for the legal notices required on French invoices
  (discount conditions, late payment penalties, fixed recovery fee).

The demo package `zugferd-invoice` is written for German invoices. The French
adaptations are done in the main file: column titles (`\InvoiceTabular...`),
VAT name, `\InitVAT{20}`, `default-vat=20`, `\sisetup{locale=FR}` and the sum names.

## PDF/A

The MIME type of `.xml` files is set in the main file (`\g_pdffile_mimetypes_prop`); PDF/A-3 requires a MIME type for the embedded `factur-x.xml`.

## Validation

Check the generated PDF with a Factur-X validator before using it for real invoices,
e.g. the FNFE-MPE validator: <https://services.fnfe-mpe.org>.
