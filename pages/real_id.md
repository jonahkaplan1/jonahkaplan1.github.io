## Real ID Helper

#### Project Overview:
The Real ID Helper uses an image classification model trained through GCP which labels a drivers license or ID card as compliant or not with the [Real ID regulations](https://www.dhs.gov/real-id). Personally, I was very confused when I first heard about Real ID, and it took me far too long to understand if I needed a new license or not. However, after reading up on the subject I quickly realized the simplicity of the new law, and the fact that it just requires a quick look at ones license. 

### Data Gathering
My primary method for gathering training data was, of coure, through google images. Although it should come as no surprise for any CV project that data gathering is the most challenging aspect, this project was particulary hard. Search results were often blurly, duplicates, or altered and unusable. It was challenging to find a sufficient amount of data to get substantial training data. 


https://imgur.com/a/u602D9N


### GCP Training
For the actual training and classification piece I chose to create a custom model using Google Cloud's Vision API. The platform was straightforward enough to use, and though it didnt provide as much training information as I'd like, it was sufficient for my purpose. I chose a binary (single label) classification model, which greatly simplified the problem and improved my training accuracy. In all the model was trained on about 1100 images

https://imgur.com/a/fqt3t7D


### Deploying 
Deploying the model was quite fun, as I realized I could build a fairly straightforward front end and use GCP's automl python clients to access my hosted model. Getting a prediction was as simple as:
```
def get_prediction(content, project_id, model_id):
    logging.warning('reading credentials ')
    # automl_v1beta1 = automl_v1beta1.Credentials.from_service_account_json(key_path)
    credentials = service_account.Credentials.from_service_account_file(key_path)
    logging.warning('setting client')
    prediction_client = automl_v1beta1.PredictionServiceClient(credentials=credentials)
    name = 'projects/{}/locations/us-central1/models/{}'.format(project_id, model_id)
    payload = {'image': {'image_bytes': content }}
    params = {}
    logging.warning('starting client')
    request = prediction_client.predict(name, payload, params)
    logging.warning('returning value from client')
    return request  # waits till request is returned
```

I was also able to spin up a [Dash page](https://plotly.com/dash/) very easily and manage to get it hosted on Heroku. It's not the fastest, but it works! The site can be reached at: [http://www.realidhelper.org/](http://www.realidhelper.org/)

<div style="text-align:center">
	<figure>
		<img src="https://imgur.com/a/SSV0qVB" width="450" />
	    <figcaption>Real ID Helper site where user simply uploads a picture of their license and receives a prediction and our confidence level</figcaption>
	</figure>
	</div>

