---
published: true
title: "Reading Specific Lines from a File with CFML (and a Question)"
layout: post
tags: [cfml,coldfusion,snippets]

---

This post touches on two subjects - the first involves reading a range of lines from a file in ColdFusion - and the second is a question: if you have a useful CFML function, where can you share it?

<!--more-->

## Reading File Lines

While working on a Commandbox custom command to [generate markdown documentation from CFCs](https://forgebox.io/view/commandbox-cfc-to-markdown-docs), I needed to read a range of lines from the file being processed. Specifically, there were situations in which I wanted to analyze the code of a specific function. Parsing the CFC with [`getComponentMetadata()`](https://cfdocs.org/getcomponentmetadata) returns a wealth of information, including the line numbers of each function.

While CFML provides a lot of handy built-in functions, extracting specific lines from a file is not one of them. A little searching lead me to CFLib.org, which had a (20 year old) [UDF by Raymond Camden](https://cflib.org/udf/FileRead) to read a file using Java. I'm no Java expert, but the UDF didn't appear to close the `fileReaderClass`, and it also seemed that CFML has developed enough over the past two decades that a revised version would be worthwhile, so I put together my own UDF for the job. Here's [a gist of it](https://gist.github.com/mjclemente/0c88b567801db63e6126e3a986ed5d0c), or you can see the code below:

```cfc
/**
*
* Based on a UDF by Raymond Camden (https://cflib.org/udf/FileRead)
*
* @hint Read a range of lines from a file
* @filepath path to the file being read
* @start number of the first line to read
* @end number of the last line to read
*/
public string function fileReadLines( required string filepath, required numeric start, required numeric end ) {

  if( !fileExists(filepath) ){
    return "";
  }

  var fileLines = '';
  var fileObject = fileOpen(filepath);
  var done = false;
  var lineNumber = 0;

  try {
    while( !done ) {
      lineNumber++;
      var line = fileReadLine(fileObject);
      if( lineNumber >= start ){
        fileLines &= line;
        if( lineNumber < end ){
          fileLines &= Chr(13) & Chr(10);
        }
      }
      if( lineNumber == end ){
        done = true;
      }
    }
  } catch( any e ) {
      rethrow;
  } finally {
      fileClose( fileObject );
  }

  return fileLines;
}
```

## Sharing Code Snippets

Later in the week, another problem resulted in a series of web searches that ultimately lead me to Brian F. Love's blog post on [hashing any ColdFusion Type](https://brianflove.com/2015-02-24/hash-any-coldfusion-type/). This included code for a very handy `hashify()` function.

All of which got me thinking, where's the best place to share CFML code snippets and functions today?

- [CFLib.org](https://cflib.org/) still has a wealth of valuable functions, but it's been a good three years since the last update.
- [Michael Borne put together cfsnippets.com](https://michaelborn.me/entry/announcing-cfsnippets-com) a little over a year ago, but the site is now down (though the [repo is still on Github](https://github.com/LearnCF/cfSnippets)).
- There's also ForgeBox, though my initial thought is that it seems a bit overkill for sharing small code snippets and functions. Then again, [`left-pad` was on npm](https://www.npmjs.com/package/left-pad), so maybe the package manager approach isn't out of the question.

I don't have an answer to this, but I figured that at the least, this might be fodder for the next episode of [Modernize or Die - CFML News](https://cfmlnews.modernizeordie.io/). It certainly seems a worthwhile topic for discussing.
