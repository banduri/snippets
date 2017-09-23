### how to download images 

to get some images off scribd i first have to download and extract the jsonp-files. Also 
the links to the jsonp files are generated through javascript. So i visited the webpage 
via tor-browser and clicked 'file->save page as'

To extract the jsonfiles i did.
```bash

cut -d '"' -f2 '/tmp/1995 Garth Ennis - Preacher Volume 1.pdf.htm' | grep -i jsonp
```
translated to human words this means cut every linke in the html-File at the delimiter " into fields but show me only field two. Show me (grep) all lines containing the word 'jsonp' (incasesensitive).
This gives us lot's of links to the assetstore of scribd

```
https://html2-f.scribdassets.com/8r1f1u8irk4qsu8q/pages/4-53bf432e6e.jsonp
https://html2-f.scribdassets.com/8r1f1u8irk4qsu8q/pages/5-d29908ddab.jsonp
...
https://html2-f.scribdassets.com/8r1f1u8irk4qsu8q/pages/710-f7d4a8a4b0.jsonp
https://html2-f.scribdassets.com/8r1f1u8irk4qsu8q/pages/711-cb60f348d4.jsonp
https://html1-f.scribdassets.com/8r1f1u8irk4qsu8q/pages/712-79d26ed86b.jsonp
```

now we need to download all those jsonp-files. Extract them and get the links to the actual jpg-image.

```bash

export htmlfile='/tmp/1995 Garth Ennis - Preacher Volume 1.pdf.htm'

for i in $(cut -d '"' -f2 ${htmlfile} | grep -i jsonp);do
  curl ${i} --output - | gunzip -f -d -c >> images.json
  echo "" >> images.json
done
```

With a for-loop i use the curl-binary to download that jsonp. I tell curl to print the content to standard output, e.g. the terminal. I take that output and send it to gunzip to decompress it. I append the output of the decompressed data to the file images.json. So i get one big file containing all jsonp data. The echo at the end of the for-loop just adds a newline after every jsonp-content.

Now every line in images.json looks like this
```
window.page10_callback(["<div class=\"newpage\" id=\"page10\" style=\"width: 902px; height:1347px\">\n<div class=image_layer style=\"z-index: 1\">\n<div class=ie_fix>\n<img class=\"absimg\" style=\"left:-1px;top:-1px;width:904px;height:1349px;clip:rect(1px 903px 1348px 1px)\" orig=\"http://html.scribd.com/8r1f1u8irk4qsu8q/images/10-18a062b021.jpg\"/>\n</div>\n</div>\n</div>\n\n"]);
```

I do the same trick again. Just cutting off everything but the links to the jpg-files.

```bash
cut -d "=" -f 10  images.json  | cut -d '"' -f 2 | tr -d "\\"
```
First use '=' as the delimiter and give me field 10. Then use '"' as a delimiter and give me field 2. Finaly remote all single backslashs (i need to quote the backslash with a backslash).

this gives me a nice list of Images to download so lets get them all.

```bash
for i in $(cut -d "=" -f 10  images.json  | cut -d '"' -f 2 | tr -d "\\");do
  curl -O ${i}
done
```

Because the imagefilenames are not in alphabeticaly order...

```
page4-e556a32bbe.jpg
page40-b69174e64c.jpg
page400-5f571f8db5.jpg
```

I rename the images to have them in a nice nicer format.

```bash
# all pages shall start with 'page-'
renmae '' page *.jpg
# rename all pageX-garbage.jpg to page0X-garbage.jpg
rename page page0 page?-*.jpg
# rename all pageXX-garbage.jpg to page0XX-garbage.jpg
rename page page0 page??-*.jpg
```

Now every image-filename is strucked as "pageXXX-garbage.jpg". So the alphabeticaly order of images is the same as the order in which they are supposed to be read.

```
page004-e556a32bbe.jpg
...
page040-b69174e64c.jpg
...
page400-5f571f8db5.jpg
...
```

### to pdf

We are done. you can use various tools to convert those images to *.cbr or *.cbz. I do prefer 
pdfs since my tablet can handle them pretty well. So i used [stackoverflow](https://stackoverflow.com/questions/4283245/using-ghostscript-to-convert-jpeg-to-pdf), `ghostscript` and `convert` to convert it to pdf

```bash
export param=''
export targetpdf='1234.pdf'

for file in $(ls -1 *.jpg);do
  export dimension=$(identify -format "%[fx:(w)] %[fx:(h)]" "${file}")
  export param="${param} <</PageSize [${dimension}]>> setpagedevice (${file}) viewJPEG showpage"
;done

gs \
	-sDEVICE=pdfwrite \
	-dPDFSETTINGS=/prepress \
	-o ${targetpdf} \
	viewjpeg.ps \
	-c "${param}"

```

In short it generates a long command for ghostscript by getting the dimension of every jpg-file in the current directory and adding a pdf-page to the resulting targetpdf containing that single jpg in the  order 'ls -1 *.jpg' provides the filenames. That's why i renamed them initialy.

### drawbacks
This way the first 3 pages are missing. But i don't mind. I can download them with my browser befor i start the conversion to pdf. I only tried one comic to download. I don't know if this working for other documents as well. No i will not setup a service to download comic from scribd for you. 



