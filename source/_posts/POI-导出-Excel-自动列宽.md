---
title: POI 导出 Excel 自动列宽
date: 2017-02-09 12:33:15
categories:
- java

tags:
- poi
---
```
Sheet sheet = workbook.getSheetAt(0);
sheet.autoSizeColumn(0); //adjust width of the first column
sheet.autoSizeColumn(1); //adjust width of the second column
```
For SXSSFWorkbooks only, because the random access window is likely to exclude most of the rows in the worksheet, which are needed for computing the best-fit width of a column, the columns must be tracked for auto-sizing prior to flushing any rows.
<!-- more -->
```
SXSSFWorkbook workbook = new SXSSFWorkbook();
SXSSFSheet sheet = workbook.createSheet();
sheet.trackColumnForAutoSizing(0);
sheet.trackColumnForAutoSizing(1);
// If you have a Collection of column indices, see SXSSFSheet#trackColumnForAutoSizing(Collection<Integer>)
// or roll your own for-loop.
// Alternatively, use SXSSFSheet#trackAllColumnsForAutoSizing() if the columns that will be auto-sized aren't
// known in advance or you are upgrading existing code and are trying to minimize changes. Keep in mind
// that tracking all columns will require more memory and CPU cycles, as the best-fit width is calculated
// on all tracked columns on every row that is flushed.

// create some cells
for (int r=0; r < 10; r++) {
    Row row = sheet.createRow(r);
    for (int c; c < 10; c++) {
        Cell cell = row.createCell(c);
        cell.setCellValue("Cell " + c.getAddress().formatAsString());
    }
}

// Auto-size the columns.
sheet.autoSizeColumn(0);
sheet.autoSizeColumn(1);
```
Note, that Sheet#autoSizeColumn() does not evaluate formula cells, the width of formula cells is calculated based on the cached formula result. If your workbook has many formulas then it is a good idea to evaluate them before auto-sizing.


来源： http://poi.apache.org/spreadsheet/quick-guide.html