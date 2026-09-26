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
lualatex DEMO-facture-factur-x.tex
```

The file was originally compiled with pdfLaTeX and the result passed the FNFE-MPE validator (Factur-X, EN 16931 profile). It has since been adapted to LuaLaTeX, in line with the other demo files (`fontspec` instead of `fontenc[T1]`, `currency=EUR` instead of `currency=€` — see note below), and re-validated successfully with the FNFE-MPE validator. `pdflatex DEMO-facture-factur-x.tex` should still work as well, as nothing else engine-specific was changed.

The result is a PDF/A-3b file with the embedded `factur-x.xml`.

> **Note on `currency=EUR` vs `currency=€`**: `zugferd.dtx` defines a
> `currency / €` meta-key as an alias for `currency=EUR`. Under pdfTeX
> (8-bit engine), the literal `€` in the source is always turned into an
> active character expanding to a macro, so both sides of the comparison
> (package and document) go through the same transformation and match.
> Under LuaTeX (native Unicode), `€` is a single raw token instead, which
> does not match the key path defined in the package, and silently falls
> back to an "unknown currency" warning followed by a cascading
> `Missing \begin{document}` error. Using `currency=EUR` directly avoids
> the issue and is strictly equivalent.

### Compile in a container

To test in an isolated environment matching CI, e.g. with
[`reitzig/texlive-base-luatex`](https://hub.docker.com/r/reitzig/texlive-base-luatex),
create a `Texlivefile` at the repository root (this image installs missing
CTAN packages on the fly, using this file as a plain list — no `#` comments):

```
babel
babel-french
hyphen-french
koma-script
booktabs
ragged2e
siunitx
xltabular
ltablex
geometry
xcolor
fontspec
lm
```

Then, from the repository root:

```bash
mkdir -p out

docker run --rm \
  --volume "$PWD":/work/src:ro \
  --volume "$PWD/out":/work/out \
  reitzig/texlive-base-luatex:2026.6 \
  work sh -c '
    latex -interaction=nonstopmode -halt-on-error zugferd.ins &&
    cd DEMO-factur-X &&
    TEXINPUTS=..: lualatex -interaction=nonstopmode -halt-on-error \
      -output-directory=/work/out DEMO-facture-factur-x.tex
  '
```

- `latex zugferd.ins` regenerates `zugferd.sty` from the local `.dtx`
  (picked up ahead of any CTAN-installed copy, since kpathsea searches the
  current directory first) — useful when testing local changes to the
  package before opening a PR.
  `zugferd-invoice.sty` needs no such step; it is used as committed.
- `TEXINPUTS=..:` lets `lualatex`, run from `DEMO-factur-X/`, find
  `zugferd.sty`/`zugferd-invoice.sty` one level up.
- The PDF and logs are written to `./out/` on the host.
- `reitzig/texlive-full` also works and needs no `Texlivefile` (everything
  from CTAN is already included), at the cost of a much larger image; being
  archived/unmaintained, it may pin an older TeX Live than `base-luatex`.

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
