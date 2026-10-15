---
id: microsoft-word-write-well-structured-reports-and
title: write well-structured reports and specifications
triggers: [Microsoft Word, headings, reports, specifications, structured, well, write]
tools: [microsoft-word]
status: approved
---

write well-structured reports and specifications: headings, lists, tables, consistent styles

## Directions learned in training

### Round 1
- Always open the target .docx file using Word's Open (File → Open or Application.Open) command before making any modifications, and verify that the document is loaded successfully by checking the presence of the expected core properties (e.g., title, heading count).
- When inserting headings, apply the built‑in style identifiers (Heading 1, Heading 2, etc.) via the Style property (Selection.Style = wdStyleHeading1 or equivalent) and then confirm the style assignment by reading back Selection.Style to ensure the correct hierarchy is used.
- After completing edits, invoke Save (File → Save or Document.Save) and, if a PDF is required, use ExportAsFixedFormat with ExportFormat:=wdExportFormatPDF, then programmatically confirm that both the .docx and .pdf files exist on disk and can be opened without errors.

### Round 2
- After inserting each heading or list item, read back its Style.NameLocal (or OutlineLevel) property to confirm it matches the intended built‑in style (e.g., "Heading 2" for a second‑level heading); if the property differs, raise an error before proceeding.
- Run a post‑edit validation that iterates through all paragraphs with a Heading style, checking that Heading 1 appears before any Heading 2, which appears before any Heading 3, and that no heading level is skipped; abort the save if the hierarchy is violated.
- Before exporting, set the document‑wide Normal style (font, size, line spacing) explicitly via the Styles collection (e.g., Styles("Normal").Font.Name = "Calibri", .Size = 11, .ParagraphFormat.LineSpacing = 1.15) to guarantee uniform formatting across the file.

### Round 1
- After finalizing the headings, create a Table of Contents via the menu: References → Table of Contents → Automatic Table 1; then confirm the TOC exists by checking that a field with code containing "TOC \h \z \t" is present in the document.
- Insert page numbers on every page using: Insert → Page Number → Bottom of Page → Plain Number 3; then verify each section's footer contains a PAGE field (visible when you toggle field codes with Alt+F9).
- Add a table with at least one header row via: Insert → Table → Insert Table (choose rows/columns); select the first row, set it as header row (Table Tools → Layout → Repeat Header Rows), then apply the "Table Grid" style using Table Tools → Design → Table Styles; finally, confirm the table's Styles collection reports "Table Grid" and that the header row flag is True.

### Round 1
- After opening a new document, immediately modify the built‑in Normal style via Home → Styles → right‑click Normal → Modify…, set Font to Calibri, Size to 11 pt, Line spacing to Multiple 1.15, click OK, then confirm the change by checking the Normal style preview shows these settings.
- Before saving, apply the Normal style to every body paragraph by selecting the paragraph and pressing Ctrl+Shift+N; verify the status bar displays “Normal” as the current style for each paragraph.
- Export to PDF using File → Export → Create PDF/XPS Document, click Options, choose “Publish what: Document”, ensure “Include non‑printing information” is unchecked, then click Publish; finally, open the created PDF via File → Open and verify that the Heading 1 appears with the same font size and weight as in the Word document.

### Round 2
- After adding headings, a table of contents, and page numbers, press Ctrl+A then F9 to update all fields; verify that the TOC entries and PAGE fields reflect the current document hierarchy before exporting to PDF.
- Save the document using File → Save As, type the filename ending with .docx, click Save; then export to PDF via File → Export → Create PDF/XPS Document, type the filename ending with .pdf, ensure 'Publish what' is set to Document, and click Publish.
- Set page margins to the standard 1‑inch on all sides via Layout → Margins → Normal (or Layout → Margins → Custom Margins, set Top, Bottom, Left, Right to 1"), to prevent excessive white borders in the final document.

### Round 3
- After all heading insertions, run a VBA validation loop (e.g., For i = 1 To ActiveDocument.Paragraphs.Count …) that reads each paragraph’s OutlineLevel and aborts the save if any Heading 2 (OutlineLevel = 2) occurs before the first Heading 1 (OutlineLevel = 1) or if a level is skipped; this guarantees correct hierarchical order before export.
- Before saving, execute a checklist using Word’s Find object: (a) search for a field whose Code contains "TOC" to confirm a Table of Contents exists, (b) search the Footer story for a PAGE field, (c) verify ActiveDocument.Tables.Count > 0 and that Tables(1).Rows(1).HeadingFormat = True with the table style "Table Grid", and (d) find at least one paragraph with Style.NameLocal = "List Bullet"; if any check fails, raise an error and halt the process.
- When creating a bulleted list, apply the built‑in List Bullet style via the ribbon Home → Paragraph → Bullets → List Bullet (or Selection.Style = wdStyleListBullet in VBA), then immediately read back each paragraph’s Style.NameLocal to confirm it equals "List Bullet" before proceeding to further edits.

### Round 4
- When inserting a missing Heading 1 to correct hierarchy, first move the cursor to the very start of the document (Ctrl+Home), press Enter, apply the built‑in Heading 1 style (Home → Styles → Heading 1), then verify that the first paragraph with style "Heading 1" is at position 1 using the Navigation pane (View → Navigation Pane → Headings).
- After creating a bulleted list, select the entire list, apply the List Bullet style via Home → Paragraph → Bullets → List Bullet, then open the Find dialog (Ctrl+F), search for the bullet character (•), and confirm that the number of matches equals the required number of list items before proceeding.
- Before saving, open the Outline view (View → Outline) and ensure the heading hierarchy is correct—Heading 1 must appear before any Heading 2, and no level is skipped—by dragging headings into the proper order if needed; then return to Print Layout view (View → Print Layout) for final export.

### Round 5
- After inserting the Table of Contents, press Alt+F9 to toggle field codes and verify the TOC field contains the "\h" switch (e.g., the code begins with "TOC \h"), then press Alt+F9 again to return to normal view.
- In each section’s footer, open the footer (double‑click), press Ctrl+F and search for the word PAGE; confirm at least one PAGE field is found per footer, and if any are missing insert a page number via Insert → Page Number → Bottom of Page → Plain Number 3.
- Before exporting to PDF, open Print Preview via File → Print, scroll through all pages to ensure a page number appears on every page; if any page lacks a number, return to the document and insert page numbers using Insert → Page Number → Bottom of Page → Plain Number 3 with the "Same as Previous" option enabled for all footers.

### Round 6
- After inserting a Table of Contents, hold Ctrl and click each TOC entry; confirm that the cursor jumps to a paragraph whose Style is the appropriate built‑in heading (e.g., Heading 1 for top‑level entries). If any click does not navigate to a heading, add or correct the heading style and then update the TOC (References → Table of Contents → Update Table).
- When a numbered list is interrupted by a table, place the cursor in the first list item after the table, then use Home → Paragraph → Numbering → Continue Numbering; verify that the displayed number follows the previous sequence (e.g., if the prior list ended with 2, the next item should show 3).
- Export to PDF via File → Export → Create PDF/XPS Document, click Options, set Publish what to "Document", uncheck "Include non‑printing information", then click Publish. Immediately open the created PDF (File → Open) and press Ctrl+F to search for the expected page numbers ("1" on first page, "2" on second, etc.) to confirm that page numbers appear in the footer and that the page count matches the Word document.

### Round 7
- After completing all edits, use File → Save As, specify the exact path (e.g., output\user_guide.docx), click Save, then immediately use File → Open to reopen that file and verify it loads without errors before proceeding to any export steps.
- When adding page numbers, select Insert → Page Number → Format Page Numbers…, ensure "Start at" is set to 1 and that the numbering option is set to continue across sections; then toggle field codes with Alt+F9 and confirm a PAGE field appears in the footer on every page.
- Validate heading hierarchy via the Navigation Pane: open View → Navigation Pane, select the Headings view, and confirm the first entry is a Heading 1, all Heading 2 entries are nested under a Heading 1, and no Heading 3 appears without a preceding Heading 2; correct any violations before saving.

### Round 8
- When creating a tip box, insert the table via Insert → Table → Insert Table, then with the table selected open Table Tools → Design → Shading, choose a light gray (or any specified color), and confirm the shading by checking that Table.Shading.BackgroundPatternColor equals the selected color value.
- Before exporting to PDF, set the page layout to portrait A4 by selecting Layout → Orientation → Portrait and Layout → Size → A4, then verify the settings by confirming PageSetup.Orientation = wdOrientPortrait and PageSetup.PageWidth/PageHeight correspond to A4 dimensions (210 mm × 297 mm).

### Round 9
- Before saving, verify the document is not empty by selecting all (Ctrl+A) and ensuring the Selection.Range.Text contains at least one non‑whitespace character; if none are found, abort with an error.
- After inserting a table, apply the built‑in "Table Grid" style via Table Tools → Design → Table Styles → "Table Grid", then confirm Table.HasBorder is true and that Table.Rows.Count > 1 and each data row (Rows(2) onward) contains non‑empty text in at least one cell (Cell.Range.Text.Trim() ≠ "").
- Immediately after exporting to PDF (File → Export → Create PDF/XPS Document → Publish), open the generated PDF (File → Open), verify that the page count matches the Word document’s page count (View → Status Bar shows same number) and that the first page contains the Heading 1 title by searching for its exact text (Ctrl+F).

### Round 10
- For every table created in the document, including callout and tip boxes, apply the 'Table Grid' style via Table Tools → Design → Table Styles → Table Grid (or iterate `For Each tbl In ActiveDocument.Tables: tbl.Style = "Table Grid"`) and set the first row as a header row via Table Tools → Layout → Repeat Header Rows (or `tbl.Rows(1).HeadingFormat = True`).
- When exporting to PDF, select File → Export → Create PDF/XPS Document, paste the full absolute output path ending in `.pdf` into the 'File name' box to match the `.docx` directory, confirm 'Publish what' is set to 'Document' in Options, and click 'Publish'.

### Round 12
- After all headings are inserted, open the Find dialog (Ctrl+F), click 'More >>', choose 'Format → Style → Heading 1' and verify the count is ≥ 1; repeat with 'Heading 2' (≥ 4) and 'Heading 3' (≥ 1 per Heading 2) and abort the save if any count is insufficient.
- When a table is added (including tip‑box tables), select the table, go to Table Tools → Design → Borders → All Borders, then Table Tools → Design → Shading and apply a light‑gray fill; open Table Properties → Table and confirm the Shading color equals the chosen fill to guarantee uniform borders and required shading.
- Before exporting to PDF, right‑click the Table of Contents, choose 'Update Field' → 'Update entire table', then press Alt+F9 to display field codes and confirm each TOC entry contains the term "HYPERLINK" (verifying clickable links); press Alt+F9 again, double‑click the footer and ensure a PAGE field is present, then proceed with Export → Create PDF/XPS Document.

### Round 13
- Begin each task by creating a new blank document (File → New → Blank document), then immediately save it to the required location using File → Save As, set "Save as type" to Word Document (*.docx), enter the exact target filename (e.g., output/spec.docx), and click Save before performing any other operations; this ensures all following edits act on the correct file.
- Insert automatic page numbers using Insert → Page Number → Bottom of Page → Plain Number 2 (centered). Immediately open the footer (double‑click the footer area), select the inserted number, and confirm via the status bar that the field type is PAGE and that the paragraph alignment is centered; repeat this check for each section to guarantee a centered page number appears on every page.

### Round 14
- After creating all headings, open the Navigation pane via View → Navigation Pane, switch to the Headings view, and verify that the list shows a single Heading 1 "Contents Demo" followed by Heading 2 "Section A" and "Section B", each with a nested Heading 3 "Subsection A1" and "Subsection B1" in that exact order; abort if the order or nesting is incorrect.
- Place the Table of Contents immediately after the title by moving the cursor to the end of the "Contents Demo" paragraph (Home → End), then insert it through References → Table of Contents → Automatic Table 1; confirm the TOC field is the next paragraph and that its field code (toggle with Alt+F9) contains "\h" to ensure hyperlinks are enabled.
- After exporting to PDF, open the PDF file, display the Bookmarks pane via View → Show/Hide → Navigation Panes → Bookmarks, and verify that a bookmark exists for each Heading 1‑3 entry and that clicking each bookmark navigates to the correct page; raise an error if any bookmark is missing or navigation does not work.

### Round 15
- Insert the Table of Contents right after the first Heading 1: place the cursor on the paragraph following the Heading 1 (press Enter to create a new paragraph if needed), then go to References → Table of Contents → Automatic Table 1; press Alt+F9 to view the field code and verify it contains "TOC \h", press Alt+F9 again, update all fields (Ctrl+A, F9), and finally lock the TOC with Ctrl+F11 to prevent accidental changes before final export.
- Add page numbers formatted as "Page X" and centered: use Insert → Page Number → Bottom of Page → Plain Number 3; double‑click the footer, move the cursor before the PAGE field, type "Page " (ensure a non‑breaking space if desired), then center the entire footer paragraph via Home → Center alignment; finally update fields (Ctrl+A, F9) and verify that each page shows "Page 1", "Page 2", etc.

## Common errors, and what fixes them
- TypeError: _Document.ExportAsFixedFormat() got an unexpected keyword argument 'Password'
  Fix: ExportFormat=17  # wdExportFormatPDF
- TypeError: TablesOfContents.Add() got an unexpected keyword argument 'UseStyles'
  Fix: UseOutlineLevels=True
- AttributeError: 'NoneType' object has no attribute 'NameLocal'
  Fix: word.Selection.Style = doc.Styles("Heading 1") | print("Inserted 'Project Specification'
- AttributeError: '<win32com.gen_py.Microsoft Word 16.0 Object Library.Range instance at 0x2240996418064>' object has no attribute 'OutlineLevel'. Did you mean: 'SetListLevel'?
  Fix: outline_level = p.OutlineLevel | print(f"  Pa
- AttributeError: '<win32com.gen_py.Microsoft Word 16.0 Object Library.Range instance at 0x2226473594432>' object has no attribute 'InsertField'. Did you mean: 'InsertFile'?
  Fix: footer_range.Text = ""  # Clear any existing text | field = footer_range.Fields.Add( | Range=footer_range, | Type=16,  # wdFieldPage = 16 | Text="PAGE", | PreserveFormatting=False
- AttributeError: wdFooterPrimary
  Fix: footer = section.Footers(1)  # wdFooterPrimary = 1 | pdf_path
- AttributeError: '<win32com.gen_py.Microsoft Word 16.0 Object Library.TableOfContents instance at 0x2598528526736>' object has no attribute 'Field'
  Fix: print("TOC entries:") | print(f"  TOC Entry: '{text}' (Style: {style_name})") | toc_para_count = 0 | for para in toc.Range.Paragraphs:
- AttributeError: '<win32com.gen_py.Microsoft Word 16.0 Object Library.Style instance at 0x2324882338352>' object has no attribute 'Name'. Did you mean: 'Frame'?
  Fix: doc = word.Documents.Open(r"C:\Users\eric1\Documents\aGENticHub\projects\personal\_training\20261009-210201-fccd\round2-heldout-1-ed5f\output\custom_style.docx") | for i, p in enumerate(doc.Paragraphs): | if text: | style_name = p.Style.NameLocal | print(f"Paragraph {i}: '{text[:50]}...' - Style: {style_name}")
