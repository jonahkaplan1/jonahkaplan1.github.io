# Face Mask Detection

_Associated Github Repo [HERE](https://github.com/jonahkaplan1/face_mask_detection)_

## Project Summary
This project developed as a way I can apply computer vision to the COVID-19 pandemic. My intention is that by being able to passively quantify mask usage, outbreak predictions can be improved and resourcing of PPE and community education can be improved. While this project is a hyper-local survey, and is no way scientific nor statiscally significant, my hope is that this is a tiny, tiny step in the right direction. 

The project at its basis is the process of gathering data, training a deep learning CV model, and deploying said model in order to categorize mask usage in my neighborhood (Panhandle, San Francisco). This project solves for gathering images of pedestrians (people walking down a sidewalk) used both in the binary classification model and later for developing daily mask usage data. I used my [Raspberry Pi](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/), the [camera module V2](https://www.raspberrypi.org/products/camera-module-v2/), and a [DSLR](https://www.nikonusa.com/en/nikon-products/product-archive/dslr-cameras/d3300.html) to gather images. 

The project highly leverages [pyimagesearch](http://pyimagesearch.com/) models and code throughout the project. A large amount of the code for person detection, face detection, and Rasperry Pi controls are attributed to them. 


## PRIVACY
In any computer vision project privacy is important, let alone one which captures images of people without their direct consent. For that reason the following steps were taken to protect peoples' privacy:
* All images/videos available in the [github repo](https://github.com/jonahkaplan1/face_mask_detection) are either directly from google or of me
* All images/video used for training data are classified, processed, then deleted. Only images of solely faces are used for training. Once training is complete, images were deleted
* All images/video used to generate mask usage data is processed to aggregate daily numbers (Ex: 5/20/20, 60% with mask, 40% without) then deleted directly after. No images are held for more than 24 hours on any machines or SD cards used in the project



## Data Collection
_Folder:_ [data_processing](https://github.com/jonahkaplan1/face_mask_detection/tree/master/data_processing)



## Data Cleaning
_Folder:_ [data_processing](https://github.com/jonahkaplan1/face_mask_detection/tree/master/data_processing)



## Training
_Folder:_ [training]https://github.com/jonahkaplan1/face_mask_detection/tree/master/train)



## Testing
_Folder:_ [testing](https://github.com/jonahkaplan1/face_mask_detection/tree/master/test)



## Deploy + Results
_Folder:_ [deploy](https://github.com/jonahkaplan1/face_mask_detection/tree/master/deploy)