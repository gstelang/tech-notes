# How to take USA passport size 2x2 inch photo
1. Take a picture
2. Use GIMP to make a 300x300 square or something similar of the picture taken. Save it in PNG (jpg being lossy)
3. Remove background using https://www.remove.bg/upload and download the image.
4. Add white background using imagemagick
```
 magick input.png -background white -flatten output.png
```
6. Add a border so you can cut the image using scissors once you print it.
```
// This adds border of 1 px.
 magick output.png -border 1 border-output.png
```
7. Create a montage so you can send it to print at Walgreens/CVS. This can print 6 photos.
```
montage border-output.png border-output.png border-output.png border-output.png border-output.png border-output.png \
-tile 3x2 -geometry 600x600+0+0 -background white -gravity center 4x6_template.jpg
```
