---
id: microsoft-excel-build-spreadsheets-data-entry-fo
title: build spreadsheets
triggers: [Microsoft Excel, build, data, entry, formulas, functions, spreadsheets]
tools: [microsoft-excel]
status: approved
---

build spreadsheets: data entry, formulas and functions, formatting, tables, charts, pivot tables, and fixing formula errors

## Directions learned in training

### Round 1
- When applying number formats in Excel via openpyxl, explicitly set cell.number_format to '"$"#,##0.00' for currency columns rather than relying on default or general formats.
- Before saving any workbook, iterate over all data cells in numeric columns to verify that number_format, font, and alignment properties match the exact specifications requested.

### Round 2
- When generating spreadsheets programmatically, ensure the script saves and closes the file, and never rely on GUI screenshot verification of code editors to prove spreadsheet generation.
- Always verify that the output file path exists and that the workbook is saved to disk before completing any spreadsheet generation task.

### Round 3
- After adding the required worksheets, open the workbook and confirm each sheet name appears on the sheet‑tab bar (e.g., Catalog, Orders, Staff, Dashboard). If a sheet is missing, create it via Home → Insert → Insert Sheet, then right‑click the new tab, choose Rename, and type the exact name.
- When turning a data range into a structured table, select the full range, choose Insert → Table, ensure the “My table has headers” box is checked, then on the Table Design ribbon enable “Banded Rows” (and optionally set a specific Table Style). Verify banded shading is visible on the sheet.
- For every required formula (VLOOKUP, IF, SUMIFS, etc.), after entry press Ctrl+` (Formulas → Show Formulas) to display the formula text in the cells, confirm the syntax and cell references are correct, then press Ctrl+` again to return to normal view and verify that each cell now shows a calculated value with no #REF! or #N/A errors.

### Round 4
- After entering any formulas, go to Formulas → Calculation Options and ensure that 'Automatic' is selected; then press Ctrl+` to show formulas, verify no #REF! or #N/A appear, and press Ctrl+` again to return to values.
- Select each column that must display currency, press Ctrl+1 to open the Format Cells dialog, choose the Number tab → Currency, set Symbol to "$" and Decimal places to 2, then click OK and verify the dollar sign appears in the cells.
- When converting a range to a structured table, press Ctrl+T, confirm the 'My table has headers' box, click OK, then on the Table Design ribbon set a Table Name in the 'Table Name' box and verify the name appears in the Name Box; this confirms the table was created and can be referenced in formulas.

### Round 5
- After entering the data, verify the table contains exactly the required rows: select the top‑left cell (e.g., A1), press Ctrl+Shift+End and ensure the highlighted range reads $A$1:$D$20 (or $A$1:$C$20 if no Trend column), confirming 20 rows are present.
- Immediately after setting each conditional format, open the Rules Manager (Home → Conditional Formatting → Manage Rules…), set 'Show formatting rules for' to 'Current Selection', and confirm that the rule’s range covers the whole column, the formula or condition matches the specification, and the displayed format (fill color, bold font, etc.) is exactly as required.
- When creating a combo chart, select the chart, go to Chart Design → Change Chart Type, and in the dialog verify that the Revenue series is assigned to "Clustered Column" on the Primary Axis and the Units series to "Line with Markers" on the Secondary Axis; also ensure that Chart Elements include a Chart Title, Axis Titles, and that data labels are enabled only for the Revenue columns.

### Round 6
- After entering the data, press F5, click 'Special', choose 'Last cell', then verify the address equals the expected bottom‑right cell (e.g., $C$21 for a 20‑row table); if it does not, add the missing rows via Home → Insert → Insert Sheet Rows until the address matches.
- When adding each conditional‑formatting rule, open Home → Conditional Formatting → Manage Rules…, set the ‘Applies to’ field explicitly to the full column range (e.g., =$B$2:$B$21), arrange the rules in the order Score > 90, Score < 60, Status = "Pending", and ensure ‘Stop If True’ is unchecked so all rules can be evaluated.
- After inserting the combo chart, use Chart Design → Add Chart Element to add a Chart Title (type "Revenue vs Units"), Primary Horizontal Axis Title (type "Month"), Primary Vertical Axis Title (type "Revenue"), Secondary Vertical Axis Title (type "Units"), then select the Revenue series, right‑click → Add Data Labels, and choose ‘Show Value’ to display labels only on the Revenue columns.

### Round 7
- After converting the data range to a structured table (Ctrl+T), immediately set a specific Table Name in the Table Design ribbon and verify it by opening Formulas → Name Manager and confirming that the table name appears with a Refers To range that exactly matches the data range.
- To add slicers for a table, use Insert → Slicer, select the table name, tick the required fields (e.g., Region, Product), then right‑click each slicer, choose Slicer Settings, and ensure the "Connect slicer to" list contains only the intended table, confirming the slicer is correctly linked.
- When building the combo chart, select the chart, go to Chart Design → Change Chart Type, assign the "Units" series to "Clustered Column" on the Primary Axis and the "Revenue" series to "Line with Markers" on the Secondary Axis; then open the Format Data Series pane for each series and verify under Series Options that the Axis setting reads "Primary Axis" for Units and "Secondary Axis" for Revenue.

### Round 8
- After any formula entry or edit, press Alt+M, then V to open the Evaluate Formula dialog (Formulas → Evaluate Formula). Step through each part of the formula to ensure it returns a valid result and fix any #DIV/0!, #REF! or #N/A before saving.
- When a formula needs to reference a range on another sheet (e.g., VLOOKUP, INDEX/MATCH, SUMIFS), first create a named range for that lookup table via Formulas → Name Manager → New, assign it a descriptive name, and then use that name in the formula. This prevents broken references if the source sheet is renamed or deleted.
- After inserting any chart, select the chart and choose Chart Design → Select Data. In the Select Data Source dialog, verify that each Legend Entry (Series) and the Horizontal (Category) Axis Labels point to the correct absolute ranges (e.g., =Sheet1!$B$2:$B$13). Click OK and confirm the chart displays the intended data.

### Round 9
- After inserting each slicer, right‑click the slicer, choose **Slicer Settings…**, confirm that the **Connect slicer to** list contains only the intended table name, then click **OK**; this guarantees the slicer is linked to the correct Table.
- When adding KPI formulas, press **Ctrl+`** (Show Formulas) to display the formula text, verify there are no #REF! or #N/A errors, then press **Ctrl+`** again and use **Formulas → Evaluate Formula** (Alt+M, V) to step through each part of the formula and confirm the result is correct before saving.
- Set calculation mode to Manual via **Formulas → Calculation Options → Manual**; then open the VBA editor (Alt+F11), open the Immediate window (Ctrl+G), type `?Application.Calculation` and ensure the returned value is **xlManual** (‑4135) to confirm the setting was applied.

### Round 10
- When generating workbooks programmatically via openpyxl where native Excel UI features like slicers or drawing objects are requested, verify whether the target file format (.xlsx) supports programmatic persistence of those drawing elements, and if unsupported, implement equivalent data filtering or summary structures instead.
- When checking formula outputs and error values using Python libraries, iterate through all formula cells after generation to evaluate string contents for error tokens such as '#DIV/0!', '#REF!', or '#N/A' to guarantee error-free calculations before saving.

### Round 11
- When applying header styling in an Excel script, explicitly configure the font name to 'Calibri', font size to 11, font bold to True, fill color to light green (hex code #90EE90 or equivalent), and iterate through cells A1:D5 to apply a thin black border to every side using openpyxl.styles.Border and Side(style='thin', color='000000').
- Before saving any workbook that contains summary data or pivot tables, include code to programmatically update and refresh all pivot caches and pivot tables in the workbook (e.g., setting pivot.cache.refreshOnLoad = True) to ensure they summarize data without errors.

### Round 12
- After entering all data and creating the workbook, immediately use the menu File → Save As, navigate to the exact folder (e.g., output), type the required filename (e.g., dynamic_lookup.xlsx), select "Excel Workbook (*.xlsx)" as the file type, click Save, and then verify the file exists in Explorer before proceeding.
- Before finishing the task, open the saved workbook via File → Open and systematically verify each required element: confirm each sheet tab name appears (e.g., Sheet1, Sheet2), select a cell in each data range and press Ctrl+L to ensure a structured Table was created with "My table has headers" checked, insert a PivotTable via Insert → PivotTable and verify its location and source range, add slicers with Insert → Slicer and check the correct field is selected, and ensure all formulas are present in the formula bar and evaluate without errors.
- When replacing a volatile formula, select the target cell, press F2, replace the OFFSET expression with the INDEX/MATCH version, press Enter, then press Ctrl+` (Formulas → Show Formulas) to display all formulas on the sheet and use Ctrl+F to search for the text "OFFSET"; confirm the search returns zero results before saving the workbook.

### Round 13
- Before saving the workbook, ensure the target folder exists: use File → Save As, navigate to the desired directory, and if the folder is missing click the "New folder" button to create it, then save the file to the exact path (e.g., output/pivot_drill.xlsx).
- Always create the PivotTable from a structured Table: select the data range, press Ctrl+T, confirm "My table has headers", give the table a name, then Insert → PivotTable, choose the table name as the source, place it on a new worksheet, and in PivotTable Analyze → Options → Data enable both "Refresh data when opening the file" and "Refresh data when filter changes".
- After inserting a slicer for a field, immediately verify its connection: right‑click the slicer, choose "Report Connections", and in the dialog check the box for the intended PivotTable; then select a slicer item and confirm the PivotTable updates with no #REF! or #VALUE! errors.

## Common errors, and what fixes them
- AttributeError: 'ConditionalFormatting' object has no attribute 'type'
  Fix: import time | time.sleep(2)
- ModuleNotFoundError: No module named 'pandas'
  Fix: import os, sys | try: | import win32com.client as win32 | except Exception as e: | print('win32 import error', e)
- ImportError: cannot import name 'PivotTable' from 'openpyxl.pivot.table' (C:\Users\eric1\Documents\aGENticHub\core\.venv\Lib\site-packages\openpyxl\pivot\table.py)
  Fix: import openpyxl.chart | print(dir(openpyxl.chart))
- NameError: name 'win32com' is not defined
  Fix: excel = win32.Dispatch('Excel.Application')
- TypeError: 'PivotFields' object is not callable
  Fix: ws_pivot = wb.Worksheets.Add(After=wb.Worksheets(wb.Worksheets.C
- AttributeError: 'method' object has no attribute 'ShowValue' and no __dict__ for setting new attributes
  Fix: from openpyxl.chart.label import DataLabelList | cats = Reference(ws, min_col=1, min_row=2, max_row=
- TypeError: must be a pywintypes time object (got datetime.date)
  Fix: start_date = datetime.datetime.today() - datetime.timedelta(days=365) | for i
- NameError: name 'datetime' is not defined. Did you forget to import 'datetime'?
  Fix: import time
