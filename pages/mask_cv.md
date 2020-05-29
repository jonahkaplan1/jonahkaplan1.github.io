# Face Mask Detection

_Associated Github Repo [HERE](https://github.com/jonahkaplan1/face_mask_detection)_

## Project Summary
This project developed as a way I can apply computer vision to the COVID-19 pandemic. My intention is that by being able to passively quantify mask usage, outbreak predictions, resourcing of PPE, and community education can be improved. While this project is a hyper-local survey, and is no way scientific nor statistically significant, my hope is that this is a tiny, tiny step in the right direction. 

The project at its basis is the process of gathering data, training a deep learning CV model, and deploying said model in order to categorize mask usage in my neighborhood (Panhandle, San Francisco). This project focuses on gathering images of pedestrians (people walking down a sidewalk) to be used in training the binary classification model and later for developing daily mask usage data. I used my [Raspberry Pi](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/), the [camera module V2](https://www.raspberrypi.org/products/camera-module-v2/), and a [DSLR](https://www.nikonusa.com/en/nikon-products/product-archive/dslr-cameras/d3300.html) to gather images. 

I highly leverage [pyimagesearch](http://pyimagesearch.com/) models and code throughout the project. A large amount of the code for person detection, face detection, and Raspberry Pi controls are thanks to them. 

<div style="text-align:center"><img src="https://i.imgur.com/3upxYOI.jpg" /></div>


## Privacy
In any computer vision project privacy is important, let alone one which captures images of people. For that reason the following steps were taken to protect peoples' privacy:
* All images/videos in the [public github repo](https://github.com/jonahkaplan1/face_mask_detection) are either directly from google or of me
* All images/video used for training data are classified, processed, then deleted. Only images of solely faces are used for training. Once training is complete, images were deleted
* All images/video used to generate mask usage data are processed to aggregate daily numbers (Ex: 5/21/20, 60% with mask, 40% without) then deleted directly after. No images are held for more than 24 hours on any machines or SD cards used in the project

<div style="text-align:center">
	<figure>
		<img src="https://i.imgur.com/GjRV92r.jpg" width="450" />
	    <figcaption>Privacy is Maintained Throughout the Project</figcaption>
	</figure>
	</div>


## Data Collection
_Folder:_ [data_processing](https://github.com/jonahkaplan1/face_mask_detection/tree/master/data_processing)

For any CV project data collection can be the most challenging aspect. This one is no different, as I struggled through a couple different attempts before landing on high quality data. 

Constraints:
* Limited "labelers", ie: I was the person collecting and classifying data
* Raspberry Pi processing power
* Low initial image quality
* Limited storage space
* Unreliable internet connection

My initial thought was to use the Raspberry Pi camera module directly and simply record a long stream of video. I assumed, over a few hours, I would capture enough content needed for training. But due to my constraints, I realized I needed to minimize my data scrubbing work. I opted to detect when a person was in frame and only then save the image. I would then classify it later on. The issue with this approach though is that it requires far more manual classification, as every image needs to be manually inspected. Because it was more common to have nobody in frame I could gather data in short videos and classify the entire video. I could then break the video into images (individual frames) later on. I could also more easily remove bad data (such as if multiple people are in frame both with or without masks)

#### Lesson #1: Data cleaning is pretty unavoidable

#### Lesson #2: It can be more efficient to process videos rather than images if your data frequently comes in unique and infrequent streams (someone walking down the street)

I also came to quickly realize that, due to the limited processing power of the Raspberry Pi running a person detection model I was only getting about 3 frames per second. This simply didn't give me a good change of capturing high quality content. In addition, the image quality of the camera module is low, resulting in unideal training data. I decided to have the Rpi trigger my DSLR camera to record video upon detecting a person. This solves my FPS and image quality problem.

#### Lesson #3: It may be optimal to utilize specialized hardware, even if it introduces complexity into your system. 

Due to my unreliable internet connection I had to save videos locally on my DSLR. It would have been more ideal to automatically upload content when the system is inactive (at night) but I did not have that option. Additionally, due to the limited storage space I had to manually process the days' assets each night.

#### Lesson #4: System processes and data handling is complementary to the data collection technology.




<div style="text-align:center">
	<figure>
		<img src="https://i.imgur.com/iD6fP16.jpg" width="450" />
	    <figcaption>A Very Basic Data Collection Setup</figcaption>
	</figure>
	</div>


## Data Cleaning
_Folder:_ [data_processing](https://github.com/jonahkaplan1/face_mask_detection/tree/master/data_processing)

In the above process we gather lots of content to be used for training our binary classification model, but we still need to clean the data (see: lesson #1). The following was done in order to get our data to a high quality and usable place:
* Videos were classified (with or without mask). If both states appeared, they were removed
* Videos were broken down into images, and just the face was extracted from each image
* The top N images were taken from each video based on face detection confidence. This is done to not over-bias one video on the wider dataset
* Image size was standardized
* Images from google were gathered, classified, and processed in order to avoid overfitting

<div style="text-align:center">
	<figure>
		<img src="https://i.imgur.com/qErqu3d.jpg" width="450" />
	    <figcaption>Data Cleaning and Standardization</figcaption>
	</figure>
	</div>



## Training
_Folder:_ [training](https://github.com/jonahkaplan1/face_mask_detection/tree/master/train)

Training the binary classification model was one of the more straightforward aspects of the project. I tried a few different models I had at my arsenal from [pyimagesearch](pyimagesearch.com) guides. At the begining, I was using more "general" photos - training data which had not been processed to only include faces. By focusing on just the faces in a photo I had much more success. This makes sense, as I removed a large amount of noise from my data. 

The tradeoff, of course, is that in order to get reliable predictions from the model the input image must be processed to extract just the faces. However, this method is optimal for our use case because our intention is to _quantify_ mask usage. This requires counting of mask / non mask, and therefore it is both an object detection and classification problem. A more general model may require less processing but would not be able to count occurences or handle mixed images very well (when both masks and no-masks are present).

I found the most success in a model which utilizes a cyclical learning rate. It took about 2 minutes to train 96 epochs. 

<div style="text-align:center"><img src="https://i.imgur.com/9Nqhstw.jpg" /></div>




## Testing
_Folder:_ [testing](https://github.com/jonahkaplan1/face_mask_detection/tree/master/test)

The testing for this project was straightforward - test out how well the mask classifier works. I gathered new google images and DSLR images to test my classifier. The testing script works by segmenting each person in the input image, detecting their face, and classifying if their face has a mask or not. The result is written on the image along with a bounding box of the persons' face. 

<div style="text-align:center"><img src="https://i.imgur.com/fxflMDQ.jpg" /></div>



## Deploy + Results
_Folder:_ [deploy](https://github.com/jonahkaplan1/face_mask_detection/tree/master/deploy)

The deployment section works very similarly to the testing script but instead of writting the prediction to the image the prediction is recorded. The folder also contains our object detection script for gathering images. The script now takes one photograph instead of a video. Each night the data is transfered to the date folder then processed. The script still uses the same Rpi and DSLR setup as described in the data collection process.

The final piece is creating the actual visualization! Extremely easy: simply aggregate the data that's been recorded over the days and visualize it. While it must be taken with a grain of salt, the overall trend is quite interesting. It truly represents how peoples' behavior has changed as the information around the COVID pandemic has evolved.


<div style="text-align:center"><img src="https://i.imgur.com/uKuNnou.png" /></div>

### If you've made it this far: thank you for reading, stay safe, stay healthy!