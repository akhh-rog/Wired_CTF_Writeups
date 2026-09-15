## CTF Writeup: Tyler_Durden
The challenge title (Tyler_Durden) and prompt hint ("You are not your file extension") suggest that the downloaded file chal.pdf is not actually a PDF

 Renaming the file extension from .pdf to .png fixes the file association issue

 i converted the file in a basic chrome website and got the png 
 wired{if_i_h4d_4_png_
 so this was the prefix

 Inspecting the file's raw header bytes (magic bytes) revealed that the signature matched a PNG file (89 50 4E 47 / %PNG), proving the file was an image disguised with a .pdf extension.

 ren chal.pdf chal.png

 Flag Reconstruction: Opening chal.png displayed the second portion of the flag (i_w0uld_nm43_i7_pdf}). Combining this with the initial prefix (wired{if_i_h4d_4_png_) revealed the complete flag.

 this is what i remember didnt had any screenshot 

 Flag
wired{if_i_h4d_4_png_i_w0uld_nm43_i7_pdf}