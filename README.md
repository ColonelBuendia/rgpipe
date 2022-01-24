`rgpipe` is a single bash/sh script and an alias to use with ripgrep to search through a myriad of file types that are otherwise not grep friendly.  Use it with ripgrep's -pre command which allows ripgrep to selectively process files before searching.


# TL;DR

The most basic usage is to point `rgpipe` at some file, and it will attempt to print the contents of said file to stdout. 

`rgpipe MyFancyExcelFile.xlsx`

The more involved usage is as a filter in front of [ripgrep](https://github.com/BurntSushi/ripgrep) to systematically attempt to grep through the contents of assorted non-text files much as you would text files. The basic incantation looks like: 

`rg --pre-glob '*.{xlsx,pptx,docx,pdf}' --pre rgpipe "$YourSearchTermHere"`

## Overview

I wrote up an extended gist about how to use it
[here](https://gist.github.com/ColonelBuendia/314826e37ec35c616d70506c38dc65aa)

That gist is only useful because of the kind note by BurntSushi in [this hacker news comment](https://news.ycombinator.com/item?id=19675934) explaining how `rg --pre-glob` works.

**This helps grep through:**

- New MS Office files (DOCX, PPTX, XLSX, variants thereof)  
    + Uses `unzip` and `sed`
- Old MS Office files (DOC, PPT, XLS, variants thereof) & new excel binary format
    + Uses `strings`
- LibreOffice files (ODS, ODT, ODP)  
    + Uses `unzip` and `sed`
- PDF  
    + Uses `pdftottext` from poppler
- Web/structured formats (HTML, XHTML ...)  
    + Uses `w3m` lynx and friends also works.  Not 100% necessary.  
- Web formats disguised as books (chm, epub)  
    + `unzip` and `w3m` for EPUB
    + `7zip` and `w3m` for chm


#### Specifically

Ubuntu wants: `sudo apt install poppler-utils p7zip w3m unzip`

termux wants: `pkg install poppler p7zip w3m`


## Usage notes

### Vanilla ripgrep usage

Assuming rgpipe is in path, use /path/to/rgpipe if it's not

```bash
rg --pre rgpipe YourSearchTermHere
```

### Better ripgrep usage

Above uses rgpipe even when it's not needed, which is slow, ripgrep can selectively use it with --pre-glob

```bash
rg --pre-glob '*.{xlsx,pptx,docx,pdf}' --pre rgpipe YourSearchTermHere
```

A more thorough pre glob:

```bash
rg --pre-glob '*.{pdf,xl[tas][bxm],xl[wsrta],do[ct],do[ct][xm],p[po]t[xm],p[op]t,html,htm,xhtm,xhtml,epub,chm,od[stp]}' --pre rgpipe YourSearchTermHere
```

An alias because that is a lot of typing

```bash
alias rgg="rg -i -z --max-columns-preview --max-columns 500 --hidden --no-ignore --pre-glob \
'*.{pdf,xl[tas][bxm],xl[wsrta],do[ct],do[ct][xm],p[po]t[xm],p[op]t,html,htm,xhtm,xhtml,epub,chm,od[stp]}' --pre rgpipe"
```

### Poor man's full text search

Step 1: use rgpipe to make text sidecar files

```bash
find-rgpipe-type() {
     find `pwd` -type f -iname "*.$1" -exec sh -c 'for f; do rgpipe "$f" > "${f%.*}.txt"; done' _ {} +
}

# or get fancy with xargs for multithreaded goodness

find-rgpipe-type-xargs() {
    find "$(pwd)" -type f -iname "*.$1" -print0 | xargs -0 -P0 -n 1 -I {} sh -c 'rgpipe "{}" > "{}.txt"'
}

```

Make text sidecars for all files with PDF extension under current directory using the function defined above.  

```bash
find-rgpipe-type pdf
```

Step 2: Use ripgrep to search those files 

```bash
rg YourSearchTermHere
```

## Super useful

1 - [this hacker news comment](https://news.ycombinator.com/item?id=19675934)

2 - [The pre processing script](http://manpages.ubuntu.com/manpages/cosmic/man1/rg.1.html) that is the template into which I added some more file types

3 - midnight commander has [great scripts](https://github.com/MidnightCommander/mc/tree/0075f36b693109a34f8729d47fdf931b0481a68e/misc/ext.d) on this subject

4 - [lesspipe of course](https://github.com/wofr06/lesspipe)

5 - [rga](https://github.com/phiresky/ripgrep-all) is a rust based tool doing a similar thing

## The name

`rgpipe` because the idea is similar to lesspipe.  
