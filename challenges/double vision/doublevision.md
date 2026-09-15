## DOUBLE_VISION CTF
* downloaded and extract the challenge file, then apply visual image-combining techniques. The challenge title and hint ("Two images, one truth.") indicate that the solution involves combining or comparing two given images.
*Pixel Difference Extraction: Using Python with the Pillow and ImageOps libraries, calculate the absolute difference between pixel matrices and enhance contrast:

i tried to do overlay in cyberchef didnt work for me so i had to choose the harder path of using python with help of online tools to achive the combination of the two images

using python was a big headache tho but with the online resouces i got the source codes and i installed all the things needed for it

pip install Pillow
from PIL import Image

# Open an image
img = Image.open("example.png")

# Display basic information
print(img.size, img.format)

# Show the image
img.show()

ran into errors
'FileNotFoundError: [Errno 2] No such file or directory

so i checked the directory
dir *.png
cd %USERPROFILE%\Downloads
python -c "from PIL import Image, ImageChops; i1 = Image.open('img1.png').convert('RGB'); i2 = Image.open('img2.png').convert('RGB'); diff = ImageChops.difference(i1, i2); diff.save('flag.png'); diff.show()"
 # gave me a black png (-_-)

 so i started to amplify the brightness and contrast

 worked and i got a png

 Opening flag.png renders the hidden pixels highlighted in high contrast, displaying the complete flag text 

 ## flag= wired{d1ff3r3nc35_m4773r}


