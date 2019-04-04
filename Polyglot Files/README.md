# PDF-ZIP-NES polyglot file generator Example
Run this: 
```
./gen_poly.py --out magic.pdf --in "flowers_book.pdf" --message txtfile.txt --zip flower.jpg --header sample.nes
```

Let's break this up : 
* `--out` is the path/name of the resulting file – which will be a perfectly valid PDF file
* `--in` is your original PDF file
	* in my example : *flowers_book.pdf*
* `--message` is the plaintext message that should appear when the resulting file will be opened in a hex editor, or directly `cat`-ed in a terminal
* `--zip` is the (list of) file(s) that are to be zipped and appended in the original PDF
	* in my example : *flower.jpg*
	* additional feature : when unzipped, the original PDF will also appear, *although it is not duplicated in the resulting file*
* `--header` is the file that is to be included at the beginning of the file, before the PDF itself
	* in my example : a NES game, for emulators. So that you can also the PDF in an emulator !

Still, with all this stuff appended to and included at the beginning of the PDF, it stays valid and viewable in any standard PDF viewer (such as Adobe Reader, Preview on macOS, etc...)
All of this is possible thanks to the fact that the PDF header does not have to be at the beginning of the file, for it to be a valid PDF.




* reference: https://github.com/perfaram/pdf-zip-nes-polyglot/blob/master/README.md