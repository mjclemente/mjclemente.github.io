---
published: true
title: Using PDFBox with ColdFusion
layout: post
tags: [pdfbox,java,cfml,coldfusion,pdfbox.cfc,pdfs]
---
This post started as an investigation of how best to extract text from a PDF; it then turned into an exploration of [PDFBox](https://pdfbox.apache.org/), lead me down the rabbit hole of PDF sanitization, and ultimately resulted in developing [pdfbox.cfc](https://github.com/mjclemente/pdfbox.cfc).
<!--more-->

### Extracting Text from PDFs with ColdFusion
A project at work required the text to be extracted from thousands of PDFs, some of which were quite large. Within Adobe ColdFusion, you can conveniently do this via `cfpdf action='extract'`. There are two potential issues with this approach.

* The first drawback (which may or may not matter to you) is that `cfpdf action='extract'` isn't supported in Lucee versions prior to 5.2.8.[^1] Personally, I prefer approaches that work in both CFML engines, so that my coding solutions and codebases are as flexible as possible.
* Second, I found that the performance of `cfpdf` varied considerably from one version of Adobe ColdFusion to the next. Further details are provided in the [table below](#cfpdf-performance-by-engine) - it generally suffered with larger documents.

### Apache PDFBox and Text Extraction
As is often the case when ColdFusion's built-in functionality doesn't quite meet my needs, I set out to find a Java library that would get the job done. [Apache PDFBox](https://pdfbox.apache.org/) stood out among the other possible options such as [iText](https://itextpdf.com/) and [PDF Clown](https://pdfclown.org/) - the former ruled out due to cost and licensing, the latter due to lack of active development.

A little background information; PDFBox is a free, open source Java PDF library; it's over 15 years old, has been an Apache project for 10 years, and, as of July 2018, is still being actively developed. It also has, I found, an incredibly helpful and responsive [mailing list](https://lists.apache.org/list.html?users@pdfbox.apache.org). If you have a question about PDFBox, they have the answer.

PDFBox can do a lot with PDFs, but my initial interest was in how quickly it could extract the text from a document. I tested this using a 393 page, 1.3 MB PDF of the book [_1984_](https://www.planetebook.com/free-ebooks/1984.pdf). __Using PDFBox, regardless of CFML engine or version, text was extracted in an average of 0.98 seconds__; variations were miniscule. As you can see below, using `cfpdf action='extract'` was generally slower, and far less consistent.

| CFML Engine <a name="cfpdf-performance-by-engine"></a> | Time to Extract |
|:---------------------------------|:--------|
| ColdFusion 10                    | 1.7 s   |
| ColdFusion 11                    | 1.7 s   |
| ColdFusion 2016                  | 0.8 s   |
| ColdFusion 2018                  | 4.0 s   |
| Lucee 4.5                        | NA      |
| Lucee 5                          | NA      |

Of particular concern was the performance drop-off in the latest version of Adobe ColdFusion; it seems that a bug is the cause.[^2]

Pleased with PDFBox's performance and functionality, I turned to it again when I was looking for a solution to sanitize PDFs.

### PDF Sanitization
A related project at work required me to _sanitize_ PDFs. What do I mean by that? PDFs can contain a lot of hidden elements, including metadata, annotations, forms, embedded search indexes, attachments, JavaScript, and more. Sanitization is the process of removing these from a document, so that it is no longer interactive and sensitive information isn't shared.

In Adobe ColdFusion 2016 and later this is trivial; you can simply run `cfpdf action='sanitize'`. However, this functionality is not present in ColdFusion 11, or in Lucee.[^3] Again, I turned to PDFBox, to see if it could provide a cross-engine solution. It did, but in this case it required a little more work on my part, and learning a lot more about PDFs than I anticipated.

### The Complexity of PDFs (A Rabbit Hole)
While PDFBox doesn't have a simple method for document sanitization, it does provide programatic access to the elements that make up a PDF. How hard could it be to list and remove these elements? This seemed to me like a straightforward way to sanitize a document.

That was before I learned that __[the spec for the PDF document format](https://www.adobe.com/content/dam/acom/en/devnet/acrobat/pdfs/pdf_reference_1-7.pdf) is over 1,300 pages__. I won't bore you with all the details, but here's a smattering of the complications I encountered:

* PDFs have two distinct types of metadata: Info Dictionary and XMP metadata (embedded as XML). These don't necessarily match, aren't necessarily present, and are handled/displayed differently by different PDF readers.
* JavaScript can be embedded in at least four different document elements.
* There are two different type of forms that be embedded in a PDF: Acroforms and XFA forms.
* Even Adobe Acrobat Pro doesn't show all the information contained in a PDF. Tools like [PDFDebugger](https://pdfbox.apache.org/2.0/commandline.html#pdfdebugger) are needed.

With the help of the PDFBox mailing list and a lot of trial-and-error, I was able to use PDFBox to provide what, in my use case, was an acceptable degree of sanitization. In my testing I was able to remove metadata, JavaScript, links and other annotations, attached documents, and embedded search indexes, as well as flatten forms. Every test PDF I used was sanitized properly. However...

...given the complexity of the PDF document type and my limited experience with its structure, I'm certain there's more work to be done and edge cases that I've not accounted for. If you encounter a shortcoming in using [pdfbox.cfc's `sanitize()`](https://github.com/mjclemente/pdfbox.cfc#sanitize) method or know of other places that sensitive data might be hiding in a PDF, I'd appreciate you letting me know.

### Putting It All Together with `pdfbox.cfc`
Rather than having to deal with PDFBox's jars and Java classes all the time, I packaged its methods together into [pdfbox.cfc](https://github.com/mjclemente/pdfbox.cfc). It's initialized with the path to a PDF; once the component is created, it provides a simple, object-oriented interface for working with the PDF.

You can call any of the implemented PDFBox operations on the document, like extracting text or sanitizing it. The component has a few bells and whistles, like being able to pass in a `cfdocument` variable in order to add a page to a PDF, and the ability to selectively sanitize elements. For example, you can flatten a form but leave metadata untouched, or vice-versa. There's more information about what it can do and how to use it in the project's README.

PDFBox provides a wide range of features and tools for working with PDFs; at this point I've barely scratched the surface. However, because I haven't found a way to add hours to the day, I don't currently have plans for adding functionality to pdfbox.cfc beyond my immediate needs. However, if there's some PDFBox functionality that you'd like added, please let me know. I'd be happy to incorporate it, or to merge pull requests that add it. Good luck PDFing!

___
[^1]:From what I could tell, there was an [issue raised for this in Railo](https://issues.jboss.org/browse/RAILO-1559) which never made it across the fork to Lucee. I created a [ticket for pdf text extraction in Lucee](https://luceeserver.atlassian.net/browse/LDEV-1941), which was quickly addressed, so this functionality will hopefully be available in the next release.
[^2]:Vote or comment on [this issue in Adobe's Bug Tracker](https://tracker.adobe.com/#/view/CF-4203233) to help get it addressed and track its progress.
[^3]:If you want it added to Lucee, you can [vote for the JIRA issue](https://luceeserver.atlassian.net/browse/LDEV-1825).