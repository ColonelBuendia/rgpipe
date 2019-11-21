rgpipe is a single bash/sh script and an alias to use with ripgrep to search through a myriad of file types that are otherwise not grep friendy.  rgpipe because the idea is similar to lesspipe.  Use it with ripgrep's -pre command which allows ripgrep to selectively do something to a file before searching it. It works with other tools as well, namely less.

I wrote up an extended gist about how to use it
[here](https://gist.github.com/ColonelBuendia/314826e37ec35c616d70506c38dc65aa)

That gist is only useful because of the kind note by BurntSushi in [this hacker news comment](https://news.ycombinator.com/item?id=19675934) explaining how ```rg --pre-glob``` works.

## This helps grep through:
- Old MS Office files (doc,ppt,xls,variants thereof like templates and add ins)
- New MS Office files (docx,pptx,xlsx, variants thereof, new excel binary format)
- LibreOffice files (ods,odt,odp)
- PDF
- Web/structured formats (html, xhtml ...)
- Web formats disguised as books (chm, epub)

There are some notes below on mobi and keynote files

# Requires
You likely already have but doublecheck: unzip, sed, strings

ubuntu wants: sudo apt-get poppler-utils p7zip w3m

termux wants: pkg install poppler p7zip w3m

why?

poppler is where you get pdftotext, p7zip is for chm files - unzip wont open them and w3m has superior support for tables and impromptu data extraction


# Usage notes

## Vanilla usage
```
rgpipe yourfilehere.xlsx
```

## Ripgrep usage
The pre-glob keeps ripgrep from being bogged down by applying rgpipe, the pre command helper script, to things that don't match the specified rules.  

```
alias rgg="rg -i -z --max-columns-preview --max-columns 500 --hidden --no-ignore --pre-glob \
'*.{pdf,xl[tas][bxm],xl[wsrta],do[ct],do[ct][xm],p[po]t[xm],p[op]t,html,htm,xhtm,xhtml,epub,chm,od[stp]}' --pre rgpipe"
```

## Rippgrep make a folder of crap searchable usage
aka poors mans's full text index

Make a script called rgindex.sh
```
#!/bin/sh
rg -i -z --no-line-number --heading --hidden --no-ignore --pre rgpipe "" "$1" > "$1".txt
```
then run it on your folder however you want leaving you with text files adjacent to your other files which are easy to grep
```
find /home/mike/mdoc/Investing -type f -iname '*'".pdf"'' -exec rgindex.sh {} \;
```

I actually use rg for that part too usually, but the above is more legible here.

## Less usage
Less for example allows you to set some varaibles to use it with less like so
```
export LESSOPEN="|/home/sanchopanda/scripts/rgpipe %s"
LESSOPEN="|lesspipe.sh %s"
```
## MC usage (midnight commander)
...

## fzf usage
find a file by glancing through it and then cd to that drectory
```
f-cd() {
	cd $( dirname "$(locate -i "$1" | fzf --preview 'rg -iz --pre rgpipe . {} | head -n 100')")
}
```
same thing but open with less or sublime
```
f-less() { less `locate -i "$1" | fzf --preview 'rg -iz --pre rgpipe . {} | head -n 100'`; }
f-subl() { subl `locate -i "$1" | fzf --preview 'rg -iz --pre rgpipe . {} | head -n 100'`; }
```

# Notes on file types

## *.pdf)
```
exec pdftotext -layout "$1" -
```

## .xl[ast][xmt])
aka new excel
```
exec unzip -qc "$1" *.xml |  sed -e 's/<\/[vft]>/\n/g; s/<[^>]\{1,\}>//g; s/[^[:print:]\n]\{1,\}//g'
```

## *.xlsb)
aka new new excel
```
unzip -qc "$1" *.bin |  strings -e l
```

## *.xl[wsrta])
aka old excel
```
exec strings  "$1"
```

## *.do[ct])
aka old word
```
exec strings -d -15 "$1"
```

## *.do[tc][xm])
aka new word
```
exec unzip -qc "$1" word/document.xml | sed -e 's/<\/w:p>/\n/g; s/<[^>]\{1,\}>//g; s/[^[:print:]\n]\{1,\}//g'
```

## *.p[po]t)
aka old powerpoint
```
exec strings -d "$1"
```

##   *.p[po]t[xm])
```
exec unzip -qc "$1" ppt/slides/*.xml | sed -e 's/<\/a:t>/\n/g; s/<[^>]\{1,\}>//g; s/[^[:print:]\n]\{1,\}//g'
```

### *.xhtm)
```
exec cat "$1" | w3m -T text/html -dump -cols 120
```

## *.xhtml)
```
exec cat "$1" | w3m -T text/html -dump -cols 120
```

## *.htm)
```
exec cat "$1" | w3m -T text/html -dump -cols 120
```

## *.html)
```
exec cat "$1" | w3m -T text/html -dump -cols 120
```

## *.epub)
```
exec unzip -qc "$1" "*.*htm*" |  w3m -T text/html -dump -cols 120
```

## *.chm)
```
exec 7z e -r -so "$1" *.htm *.xml *.htm *.html *.xhtm *.xhtml | w3m -T text/html -dump -cols 120
```

## *.od[stp])
These are the main libreoffice office filetypes
```
exec unzip -qc "$1" *.xml |  sed -e 's/<\/text:p>/\n/g; s/<[^>]\{1,\}>//g; s/[^[:print:]\n]\{1,\}//g'
```

## *.key)
This is Apple's keynote format.  This is not implemented in the script because this currently works poorly (hitting the relevant iwa files with strings, use someting like Snzip to do it right). Note the format has varied over time without changing extension, and that key files are often otherwise greppable text files in other contexts.  
```
 unzip -qc "$1" */Slide* */*/Data | strings -n 13
```


## mobi
A simple way is to use [this script](https://github.com/kevinhendricks/KindleUnpack/archive/v032.zip) but it writes to disk so I don't really. Any notes on how to do this using the existing tools here would be highly appreciated.  


# Superior tools to transform to to text but extra dependencies so eh
catdoc, catppt, xls2csv, xlsx2csv, ods2txt, ebook-convert, ssconvert

# Sorry not sorry
Redundant cat usage makes things easier to read 

Regex was mixed with html in earlier versions of this script, it may be again in the future  

# Super useful

1 - [this hacker news comment](https://news.ycombinator.com/item?id=19675934)

2 - [The pre processing script](http://manpages.ubuntu.com/manpages/cosmic/man1/rg.1.html) that is the template into which I added some more file types

3 - midnight commander has [great scripts](https://github.com/MidnightCommander/mc/tree/0075f36b693109a34f8729d47fdf931b0481a68e/misc/ext.d) on this subject

4 - [lesspipe of course](https://github.com/wofr06/lesspipe)

5 - [rga](https://github.com/phiresky/ripgrep-all) is a rust based tool that also references my original gist (yay!)

6 - Some [notes](https://github.com/obriensp/iWorkFileFormat/blob/master/Docs/index.md#snappy-compression) on [keynote files](https://github.com/kubo/snzip/issues/14)
