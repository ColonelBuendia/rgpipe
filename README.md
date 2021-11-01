```rgpipe``` is a single bash/sh script and an alias to use with ripgrep to search through a myriad of file types that are otherwise not grep friendy.  ```rgpipe``` because the idea is similar to lesspipe.  
Use it with ripgrep's -pre command which allows ripgrep to selectively process files before searching


I wrote up an extended gist about how to use it
[here](https://gist.github.com/ColonelBuendia/314826e37ec35c616d70506c38dc65aa)

That gist is only useful because of the kind note by BurntSushi in [this hacker news comment](https://news.ycombinator.com/item?id=19675934) explaining how ```rg --pre-glob``` works.

## This helps grep through:
- New MS Office files (docx,pptx,xlsx, variants thereof)  
    + Uses ```unzip``` and ```sed```
- Old MS Office files (doc,ppt,xls,variants thereof) & new excel binary format
    + Uses ```strings```
- LibreOffice files (ods,odt,odp)  
    + Uses ```unzip``` and ```sed```
- PDF  
    + Uses ```pdftottext``` from poppler
- Web/structured formats (html, xhtml ...)  
    + Uses ```w3m``` lynx and friends also works.  Not 100% neccesary.  
- Web formats disguised as books (chm, epub)  
    + ```unzip``` and ```w3m``` for epub
    + ```7zip``` and ```w3m``` for chm


### Specifically

Ubuntu wants: `sudo apt install poppler-utils p7zip w3m unzip`

termux wants: `pkg install poppler p7zip w3m`


# Usage notes

## Vanilla ripgrep usage
Assuming rgpipe is in path, use /path/to/rgpipe if it's not
```
rg --pre rgpipe YourSearchTermHere
```
## Better ripgrep usage
Above uses rgpipe even when it's not needed, which is slow, ripgrep can selectively use it with --pre-glob
```
rg --pre-glob '*.{xlsx,pptx,docx,pdf}' --pre rgpipe YourSearchTermHere
```
A more thorough pre glob:
```
rg --pre-glob '*.{pdf,xl[tas][bxm],xl[wsrta],do[ct],do[ct][xm],p[po]t[xm],p[op]t,html,htm,xhtm,xhtml,epub,chm,od[stp]}' --pre rgpipe YourSearchTermHere
```
An alias because that is alot of typing
```
alias rgg="rg -i -z --max-columns-preview --max-columns 500 --hidden --no-ignore --pre-glob \
'*.{pdf,xl[tas][bxm],xl[wsrta],do[ct],do[ct][xm],p[po]t[xm],p[op]t,html,htm,xhtm,xhtml,epub,chm,od[stp]}' --pre rgpipe"
```

## Poors mans's full text search

Step 1: use rgpipe to make text sidecar files
```
findrgpipetype() {
	 find `pwd` -type f -iname "*.$1" -exec sh -c 'for f; do rgpipe "$f" > "${f%.*}.txt"; done' _ {} +
}
```
Step 2: Use ripgrep to search those files 
```
rg YourSearchTermHere
```

# Super useful

1 - [this hacker news comment](https://news.ycombinator.com/item?id=19675934)

2 - [The pre processing script](http://manpages.ubuntu.com/manpages/cosmic/man1/rg.1.html) that is the template into which I added some more file types

3 - midnight commander has [great scripts](https://github.com/MidnightCommander/mc/tree/0075f36b693109a34f8729d47fdf931b0481a68e/misc/ext.d) on this subject

4 - [lesspipe of course](https://github.com/wofr06/lesspipe)

5 - [rga](https://github.com/phiresky/ripgrep-all) is a rust based tool doing a similair thing

