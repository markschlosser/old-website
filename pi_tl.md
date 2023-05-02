# Raspberry Pi Time-Lapse Project

## Video

<video width="320" height="240" controls>
	<source src="https://user-images.githubusercontent.com/21339323/235735968-a6035423-0e09-4baf-aef2-b4e657560f9f.mp4" type="video/mp4">
	Your browser does not support the video tag.
</video>

## Setup

A snow storm was approaching my area and I wanted to try using one of my Raspberry Pis to capture a time-lapse video of the snow accumulation. Armed with a roll of plastic wrap, I set to work...

Supplies:
- Raspberry Pi Zero W
- Raspberry Pi Camera Module
- 16 GB SD card
- USB power bank with USB cable
- Plastic wrap
- Gallon zip-loc bag
- Wooden stake
- Rubber bands

I wrapped the Pi Zero with plastic wrap to prevent moisture from messing up the electronics. The USB power bank went in the zip-loc bag. Both got mounted to the wooden stake using generous amounts of rubber bands. At first, I tried to be fancy with actual cloth straps, but sometimes simpler is easier!

After taking a few experimental photos, I found a good position for the setup and pounded the stake into the cold ground. [Image](#image)

## The Storm

This particular Raspberry Pi already has a simple homemade web interface deployed on it. This lets me control the time-lapse camera from anywhere within the reach of my home wifi. I will share this code at some point. I set the Raspberry Pi to capture an image with automatic exposure every forty-five seconds.

The storm started around noon, but really picked up speed as the sun starting going down. Meaningful accumulation seemed to stop during the night. I left the patio light on all night (near the setup). The next morning, I retrieved the Raspberry Pi and unwrapped it from the plastic wrap. Everything survived! I moved the photos to my home computer.

## Making the Video

On Linux, I used jhead and ImageMagick to embed timestamps onto the images. ImageMagick also brightened some of the night shots. Then I used ffmpeg to stitch together a video from the shots. Audacity generated an ambient audio track.

Here are some of the bash scripts I used.

brighten_images:

```
# specify brighten percentage as argument. for example: 200%
# files will end up in a subdir named 'bright'

mkdir bright
for PHOTO in *.jpg
do
  BASE=`basename $PHOTO`
  convert -modulate $1 "$PHOTO" "bright/$BASE"
done
```

add_timestamp_to_images:
```
# operated on all .jpg images in the current directory.
# timestamped images will be written to same directory with ts_ prefix added to each image file.

for f in *.jpg
do
  echo $f
  timestamp=$(jhead $f | grep Date | cut -d' ' -f7)
  convert $f -fill white  -undercolor '#00000080' -pointsize 120 -gravity NorthEast -annotate 0 "$timestamp"  ts_$f
done
```

make_timelapse_video:
```
# this operated on all images in the current directory following a certain naming pattern.
# you should specify starting number in the filename as an argument to this script.
# this script assumes all image files will be named ts_cam00000000.jpg and so on.
# the starting number to specify in that case will be 00000000.
# if you want to start from a later image, specify something like 00000022.
# video will be sent to same directory, named 'output.mp4'

ffmpeg -r 15 -start_number $1 -i ts_cam%08d.jpg -s 3280x2464 -vcodec libx264 output.mp4
```

add_audio_track:
```
ffmpeg -i input_video.mp4 -i AudioTrack.wav -map 0:v -map 1:a -c:v copy output_video_with_audio.mp4
```

# Image

![20201216-120805-setup](https://user-images.githubusercontent.com/21339323/235734437-ca57306d-c03c-456a-978b-b4b06e3b2a52.jpg)
