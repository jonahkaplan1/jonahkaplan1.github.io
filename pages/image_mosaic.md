
- [Take me back to Overview](https://jonahkaplan1.github.io/)

Image Mosaic Generator
------

As a personal project I set out to do / build something related to music using R. In the end, I built a method to re-generate an image from many albums covers. The project uses Spotify's API (via an R package) to pull a large set of album covers. Finally, the script will analyze an image as an input and re-generate it from the database of images. The following project goes over:

* Methods and project spec
* How to analyze an image's color pixel by pixel
* Comparing colors
* How to gather a large set of albums using Spotify's API
* Generating an image from other images
* Future improvements + Examples!

In true DIY fashion, finished product first:

![logo](https://i.imgur.com/V1X3lNe.jpg)

generated from: [https://i.scdn.co/image/fc48b64069e0011cd18bdc4f2ad336e25f3dfd8b](https://i.scdn.co/image/fc48b64069e0011cd18bdc4f2ad336e25f3dfd8b)



# Methods and process

I started by building up a large base of albums, so that we have plenty to choose from when re-generating an image. For this, I got a large list of artists, and used Spotify’s API to pull similar artists. From this list of artists, I pulled all albums they have created. Once there are a sufficient amount of albums (16,000) we can analyze the “composite” or overall color of the image. This is stored as a dataframe which holds each album's ID, a path to the file, and the album's composite color. To improve run time, I downloaded all albums locally, however, using Spotify's API file paths can be replaced with URL's 

Next, we can focus on the image we want to generate. To start, the source image is analyzed in clusters, in the example case 5x5 pixel grids. Each clusters's color is analyzed and assigned a hexidemal color value. When analyzing the image, we created a dataframe that includes the position, the RGB values of the grid, and it's hexidecimal color value. 

We create variables for the source image to be used while regenerating. This includes the original image size, grid height (which is squared to get whole cluster), and album size for regenerating.

Lastly, we iterate over the dataframe from the source image (called the match_frame) to match each pixel grid with an album. The file path to the album is placed in our match frame. Once all grids from the source image are matched, we begin the final image processing. We generate our image in columns, and then append columns together. The first album is selected, resized, and then stored as a variable. The next album is select, resized, then appended to the bottom of the first image. Now the appended images are stored to overwrite the original variable. This is repeated until a column is created and stored as a variable. We now create a second column, which is appended to our first column once complete, and the column variable is overwritten. This is repeated until the entire image is generated


# Analyzing an image

A key to success of this project was being able to compare the color of one image or a few pixels of one image to another image. In this use case, all of our images are album covers. To do this, we used a few packages: 

`library(png)
library(grid)
library(gridExtra)
library(magick)`

* Magick is a pretty awesome package, I totally recommend exploring this further!

To get the overall, or “composite”, RGB of an image, we run the following script:
```r
download.file('https://upload.wikimedia.org/wikipedia/en/1/17/The_Killers_-_Hot_Fuss.png', 'sample.png')
img <- readPNG('sample.png')
df = data.frame(
  red = matrix(img[,,1], ncol=1),
  green = matrix(img[,,2], ncol=1),
  blue = matrix(img[,,3], ncol=1)
)

# now in a dataframe - we need to get average RGB values 

df$red <- (df$red*255)^2
df$green <- (df$green*255)^2
df$blue = (df$blue*255)^2

hot_fuss <- df %>% mutate(
    red = sum(red)/nrow(df),
    green = sum(green)/nrow(df),
    blue = sum(blue)/nrow(df)
    ) %>% mutate(
    red = sqrt(red),
    green = sqrt(green),
    blue = sqrt(blue) 
    )
# condense to single row
hot_fuss <- hot_fuss[!duplicated(hot_fuss$red),]

You can also take the Hexidecimal color value with something like:
rgb(averaged$red, averaged$green, averaged$blue, maxColorValue=255)
```

In the above, what we are doing is separating each pixel of an image into it’s Red, Green, and Yellow parts. We are then finding the average of these three parts to get the composite color. Next we need to figure out how to compare two colors. 

# Comparing colors

Comparing colors is actually a bit tricky, I learned how to do it [here](https://medium.com/@kevinsimper/how-to-average-rgb-colors-together-6cd3ef1ff1e5)

Lets compare the above example with Kanye West’s My Beautiful Dark Twisted Fantasy. Link here: 
http://cdn.wegotthiscovered.com/wp-content/uploads/cover.png

I ran the new image to a data frame called mbdtf. We can get a quantifiable value for the similarity of their color by doing:
```ralbum_distance = sqrt((hot_fuss$red[1]-mbdtf$red[1])^2+(hot_fuss$green[1]-mbdtf$green[1])^2+(hot_fuss$blue[1]-mbdtf$blue[1])^2)```

**In this example our album distance is 185.6228**. Not very good. Lets just check accuracy and compare two more similar albums. [Radiohead OK Computer](https://i.pinimg.com/originals/f5/b0/6e/f5b06eccafa9bc51a51f93dd49d72c25.png) and [LCD Soundsystem Sound of Silver](https://lastfm-img2.akamaized.net/i/u/ar0/62e79d7331b34ea9ced494570a2fe797)

The similarity of these two is 50 - much better!

# Using Spotify’s API and sourcing albums

For this project, it’s important to have a very large set of album’s to choose from, since we want to put an album that most closely resembles a block of color in this position. In order to use Spotify’s API, I primarily used the [Rspotify package](https://github.com/tiagomendesdantas/Rspotify) which is an awesome package! Get started by following the installation and set up steps through the link to the package. There is another R + Spotify package you should check out - [Spotifyr](http://www.rcharlie.com/spotifyr/)

I started with a large set of Artist names, and then used the function `getRelated` to get related artists of the input artist. Then once I have A LOT of artists, I used `getAlbums`to find ALL albums from the artist. Once you do this for everyone, you’ll have a large set of albums. Now simply find the composite color of each album as described above so we can later match it to another color. 

At this point, I realized I’m going to have a lot of random album covers in my image, so I wanted to go an extra step to find albums that I know I’ll like. I used the function `getPlaylist` to take all playlists from my personal Spotify account, then `getPlaylistSongs` to take all songs from my playlists. Then, I took all artists from my songs to get Artists I know I’ve heard before. Repeat the same process as above, but be sure to indicate that this is a “personal” artist for later on. 

# Generating an image from other images

Now that we have a large set of albums and their composite color, we can generate our first image. Lets use [Daft Punk](https://qph.fs.quoracdn.net/main-qimg-b33ef050d7eac7cdd4b170a3f548d461-c) as our example image and download it as “daft.png” and in this case set the size of the image to 735x735 pixels. What we need to do is split this image into grids that are divisible by the total height and width. So 735 / 49 = 15. This means we can break the image down into 7x7 pixel grids.

Then for each grid, we will take the composite color, which we will later match to the closest album. Once we have an album match for each grid we can put all the albums together in the right size to re-create the image. 

```r
img <- readPNG("~/Downloads/daft.png")
df = data.frame(
  red = matrix(img[,,1], ncol=1),
  green = matrix(img[,,2], ncol=1),
  blue = matrix(img[,,3], ncol=1)
  )



# lets just assume readPNG writes left to right, then down one it hits width 
# so we want 1,2,3,4,5, 601,602,603,604,605, etc

pixel_cluster <-  49# the total cluster area (7 x 7, 8 x 8, etc)
root_pixel_cluster <- sqrt(pixel_cluster) # 5 



# whatever size of the image you choose, must be divisable by root_pixel_cluster
height <- 735
width <- 735


x_shift <- 0
while (x_shift < width/root_pixel_cluster) {

cycle <- 0
while (cycle < height/root_pixel_cluster) {

# got first row 
i <- 1
position <- root_pixel_cluster + (cycle*root_pixel_cluster) + (x_shift*width*root_pixel_cluster)
while (i < root_pixel_cluster) {
    position_1 <- root_pixel_cluster + (cycle*root_pixel_cluster) + (x_shift*width*root_pixel_cluster) - i
    position_2 <- c(position,position_1)
    position <- position_2
    i <- i +1

}
# second row would be first row PLUS height. end at 4 since row 1 is done by first while loop and we're adding height here

h <- 1
final <- position
while (h < root_pixel_cluster) {
    row <- position + (height*h)
    row_1 <- c(final,row)
    final <- row_1
    h <- h + 1
}

# now we can select just rows from df by this subset 
top_left <- df[final,]

# lets get aggregate color of this df 
top_left$red <- (top_left$red*255)^2
top_left$green <- (top_left$green*255)^2
top_left$blue = (top_left$blue*255)^2

top_left <- top_left %>% mutate(
    red = sum(red)/nrow(top_left),
    green = sum(green)/nrow(top_left),
    blue = sum(blue)/nrow(top_left)
    ) %>% mutate(
    red = sqrt(red),
    green = sqrt(green),
    blue = sqrt(blue) 
    )
# remove duplicates
top_left <- top_left[!duplicated(top_left$red),]

# we make the hex so we can match off of this later
top_left$hex <- rgb(top_left$red, top_left$green, top_left$blue, maxColorValue=255) 

if (exists("match_frame") == TRUE) {
    temp <- rbind(match_frame, top_left)
    match_frame <- temp
} else {
    match_frame <- top_left
}

cycle <- cycle + 1
}

cat(x_shift, "success", " ", (width/root_pixel_cluster)-x_shift, "left", "\n")
x_shift <- x_shift + 1
}
```

Here what we are doing is splitting our source image into 7x7 grids, taking the composite of each grid, and storing this as a data frame called match_frame


```r

i <- 1

while (i <= nrow(match_frame)) {

#on first run create the album_match column 
if (i == 1) {
    match_frame$album_match <- NA
    match_frame$was_personal <- NA
}

# make sure it's null so we can overwrite it - we're going to fill in the blank a couple times along the way
if (is.na(match_frame$album_match[i])) {

# perhaps we can take closest match between the two
r1 <- match_frame$red[i]
g1 <- match_frame$green[i]
b1 <- match_frame$blue[i]  

storage <- storage %>% mutate(d = sqrt((red-r1)^2+(green-g1)^2+(blue-b1)^2))
storage$weighted_match <- storage$d - storage$personal_album
match_frame$album_match[i] <- storage$album_uri[storage$weighted_match == min(storage$weighted_match)]
match_frame$was_personal[i] <- storage$personal_album[storage$weighted_match == min(storage$weighted_match)]
#match_frame$album_match[i] <- storage$album_img[storage$d == min(storage$d)] # use this if we don't have all the images downloaded already
storage <- storage %>% select(-c(d, weighted_match))

}

# now we match values and fill in the blank - only want to do this after we have a good amount of values
# do it at 20% done, 33% done, 1/2 done
if (i %in% c(nrow(match_frame)/5, nrow(match_frame)/3, nrow(match_frame)/2)) {
    match_frame$album_match <- match_frame$album_match[match(match_frame$hex, match_frame$hex)]
}

cat("success", " ",i, "\n")

i <- i + 1
}
```

What we are doing here is matching each grid to our album set (storage). We also have an additional column in our storage df called personal_album which helps us weigh our personal albums heavier. Lastly, for efficiencyy we fill in other albums that have the same hex value, since we don’t mind duplicating albums.


```r
# image dimensions for generating:
box <- paste0(pixel_cluster,"x",pixel_cluster,"!")
box_together <- (height / root_pixel_cluster) * pixel_cluster
box_paste <- paste0(pixel_cluster,"x",box_together,"!")


shift <- 0
while (shift < width/root_pixel_cluster) {

i <- 1
position <- i + (shift*(width/root_pixel_cluster))
#column <- image_scale(image_read(match_frame$album_match[position]), "64x64!")
column <- image_scale(image_read(paste0("~/lyrics/album_images/",match_frame$album_match[position],".png")), box)
while (i < width/root_pixel_cluster) {
image <- image_scale(image_read(paste0("~/lyrics/album_images/",match_frame$album_match[position+i],".png")), box)
# image <- image_scale(image_read(match_frame$album_match[position+i]), "64x64!")
temp <- column
column <- c(temp, image)
column <- image_scale(column, box)
i <- i + 1
}



if (shift == 0) {
    main <- image_scale(image_convert(image_append(image_scale(column, "64x64"), stack = TRUE), "jpg"), box_paste)
} else {
    new_column <- image_scale(image_convert(image_append(image_scale(column, "64x64"), stack = TRUE), "jpg"), box_paste)
    temporary <- image_append(c(main,new_column), stack = FALSE)
    main <- temporary
}
shift <- shift + 1
cat("In progress", shift, "\n")

}
```

This was the last piece, and we now have an image called “main”. What we are doing here is taking the album that was matched to each grid and putting it in it’s position for the new image. We generate the first column, working top down, then the second column, append them together, and continue right until completion. Now we’re done!


# Future Improvements

The script is not efficient and takes a while to run. Additionally, images with curves or gradients are not handled very well. If people read this and decide to improve it, please reach out!

# Examples

Some examples I generated along the way - some came out better than others!

Dark Side of the Moon             |  Pink Floyd
:-------------------------:|:-------------------------:
<img src="https://pisces.bbystatic.com/image2/BestBuy_US/images/products/5708/5708825_sa.jpg" width="425"/>  |  <img src="https://i.imgur.com/uAkJ58N.jpg" width="425"/>


I set the cluster size very small for this image

Graduation             |  Kanye West
:-------------------------:|:-------------------------:
<img src="http://cdn.shopify.com/s/files/1/0993/9646/products/B007CD.jpeg?v=1466319003" width="425"/>  |  <img src="https://i.imgur.com/npbkF4K.jpg" width="425"/>


Starboy             |  The Weekend
:-------------------------:|:-------------------------:
<img src="https://images.genius.com/565381daa3a7d73a551d9738732b26d5.1000x1000x1.jpg" width="425"/>  |  <img src="https://i.imgur.com/eDAAtVt.jpg" width="425"/>

Small details are tough to capture

Chunk of Change             |  Passion Pit
:-------------------------:|:-------------------------:
<img src="https://images-na.ssl-images-amazon.com/images/I/71uYh7t106L._SL1500_.jpg" width="425"/>  |  <img src="https://i.imgur.com/tmXx4mB.jpg" width="425"/>


As are gradients

My Type             |  Saint Motel
:-------------------------:|:-------------------------:
<img src="https://i.pinimg.com/originals/68/9f/27/689f27f705c73650b5ed63979f885a8c.jpg" width="425"/>  |  <img src="https://i.imgur.com/qEPR6Vc.jpg" width="425"/>


Worlds             |  Porter Robinson
:-------------------------:|:-------------------------:
<img src="https://upload.wikimedia.org/wikipedia/en/thumb/e/eb/Porter_Robinson_-_Worlds.jpg/220px-Porter_Robinson_-_Worlds.jpg" width="425"/>  |  <img src="https://i.imgur.com/ZAq5PCa.jpg" width="425"/>


Teens of Denial             |  Car Seat Headrest
:-------------------------:|:-------------------------:
<img src="https://img.discogs.com/3ELt9EWGr5lRYuPF9_7gF_KVax0=/fit-in/500x500/filters:strip_icc():format(jpeg):mode_rgb():quality(90)/discogs-images/R-8679649-1475730077-2803.jpeg.jpg" width="425"/>  |  <img src="https://i.imgur.com/isxEOY5.jpg" width="425"/>


An early version of my final output - before I removed the writing

Random Access Memories             |  Daft Punk
:-------------------------:|:-------------------------:
<img src="https://qph.fs.quoracdn.net/main-qimg-b33ef050d7eac7cdd4b170a3f548d461-c" width="425"/>  |  <img src="https://i.imgur.com/CkSjTEw.jpg" width="425"/>

I thought this output was very interesting

Transitions             |  SBTRKT
:-------------------------:|:-------------------------:
<img src="http://cdn2-www.musicfeeds.com.au/assets/uploads/744981f945b41d82ef9dd496a890b383-300x300.jpg" width="425"/>  |  <img src="https://i.imgur.com/yx7c7jE.jpg" width="425"/>


It doesn’t always work as expected

Flower Boy             |  Tyler The Creator
:-------------------------:|:-------------------------:
<img src="https://complex-res.cloudinary.com/images/c_limit,f_auto,fl_lossy,q_auto,w_1030/bebllwzjpsujz9ffwp6s/tyler-the-creator-scum-fuck-flower-boy-cover" width="425"/>  |  <img src="https://i.imgur.com/oqA0Ll2.jpg" width="425"/>


And this is what happens when you mess up the image and cluster sizing

Starboy             |  The Weekend
:-------------------------:|:-------------------------:
<img src="https://images.genius.com/565381daa3a7d73a551d9738732b26d5.1000x1000x1.jpg" width="425"/>  |  <img src="https://i.imgur.com/ozJK552.jpg" width="425"/>
