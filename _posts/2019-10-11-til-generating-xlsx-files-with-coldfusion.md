---
published: true
title: "TIL: Generating .xlsx Files with CFML - An Easy Way to Reduce File Size"
layout: post
tags: [xlsx,til,cfml,coldfusion]
---
Just a quick note about generating XLSX (Excel) files with ColdFusion, which basically amounts to me regurgitating the documentation. It's a small change in code, but the reduction in spreadsheet file size can be considerable.
<!--more-->

## Some Background

I've been under the (mistaken) impression that when generating spreadsheet files with ColdFusion, my options were limited to XLS or CSV. I would generate the former using the built-in spreadsheet functions (`spreadsheetNew()`) and the latter by manually assembling strings into a comma-delimited format.

While I'm a big fan of the plaintext, interoperable nature of the CSV, some clients explicitly request Excel sheets - so they get their XLS files. These were always "good enough" for the basic reports they were requesting.

Nevertheless, it bothered me that I was using an outdated format for Excel files. My core frustration was practical - XLS files are so much larger than their XLSX counterparts - but this was never enough of a bother to warrant further investigation.

## The Code

Consequently, I was happily surprised to discover that `spreadsheetNew()` provides a second boolean argument, `xmlFormat`, which indicates if the spreadsheet should be created in the XLSX format - it's right there in the [docs](https://cfdocs.org/spreadsheetnew) - somehow I've just always missed it. Here's the complete method signature:

```cfc
spreadsheetNew( [sheetname] [, xmlFormat] );
```

The `xmlFormat` option defaults to `false`, generating an XLS file - and that's how I'd been using it for all these years. Sigh.

Here's an example generating a spreadsheet, first using the default XLS approach, and then setting `xmlFormat` to `true`, in order to generate an XLSX file:

```cfc
// legacy approach
filePath = expandPath( '/' ) & 'test.xls';
sheet = spreadsheetNew( 'XLS Test' );

for (i=1; i<=100; i++) {
  spreadsheetAddRow( sheet, i );
}

spreadsheetWrite( sheet, filePath, true );

// XLSX approach
xlsxFilePath = expandPath( '/' ) & 'test.xlsx';
xlsxSheet = spreadsheetNew( 'XLSX Test', true );

for (i=1; i<=100; i++) {
  spreadsheetAddRow( xlsxSheet, i );
}

spreadsheetWrite( xlsxSheet, xlsxFilePath, true );
```

In order to save it as an XLSX file, the only difference was setting the `xmlFormat` to true (and updating the file extension). 

To take a look at the benefit of this change, let's compare file sizes:

```cfc
sheetSize = getFileInfo( filePath ).size;
xlsxSheetSize = getFileInfo( xlsxFilePath ).size;
bytesSaved = sheetSize - xlsxSheetSize;
change = ( bytesSaved / sheetSize )*100;
writeOutput( "File size reduced by #round( change )#% ");
// File size reduced by 54%
```

Sure enough, **the resulting XLSX file is less than half the size of the identical XLS one**. That's a pretty big win for one very small change in code.

### A Note for Lucee CFML Users

The [Spreadsheet Functions](https://helpx.adobe.com/coldfusion/cfml-reference/coldfusion-functions/functions-by-category/spreadsheet-functions.html) provided by Adobe ColdFusion are not natively supported in Lucee. Fortunately, it's not too much work to add them, as I've previously blogged: [How Not To Use Spreadsheet Functions in Lucee](/2018/11/05/using-excel-spreadsheet-functions-with-lucee-5-on-docker.html). 

Read that post for a more detailed exploration of using spreadsheet functions in Lucee, but the TLDR; is that there are two basic approaches:

1. Install the [*cfspreadsheet*](https://forgebox.io/view/037A27FF-0B80-4CBA-B954BEBD790B460E) extension. This can be done programmatically or via the Lucee admin.
2. Take an altogether different approach and use Julian Halliwell's [cfsimplicity/lucee-spreadsheet](https://github.com/cfsimplicity/lucee-spreadsheet) standalone library for spreadsheet manipulation.

Happy spreadsheeting!
