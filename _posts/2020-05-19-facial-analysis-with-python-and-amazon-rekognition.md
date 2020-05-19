---
layout:     post
title:      Facial Analysis with Python and Amazon Rekognition
date:       2020-05-19
summary:    Getting started with Facial Analysis using AWS
categories: beginners python cv aws
---
Cameras are getting smarter, they're changing how we manufacture goods, how we drive our cars and even, for good or bad, how surveillance is being carried out. 

And now it's getting even easier than ever to add this functionality to your solutions; it's becoming more democratised, allowing anyone to add features like facial analysis to their projects. One of the big players driving this is AWS, their goal is to put machine learning into the hands of every developer. For vision, they're doing it with [Amazon Rekognition](https://aws.amazon.com/rekognition/).

# Amazon Rekognition
Rekognition makes it easy to add image and video analysis to your applications using pre-trained models that require no machine learning know-how. It unlocks the ability to identify objects, people, text, and actions in images and videos, as well as detecting inappropriate content. 

One of the most impressive features is the highly accurate facial detection; you can use it to detect, analyze, and compare faces for a whole range of verification, behaviour and public safety use cases. This post will focus on those facial detection features and I'll cover some of the others in the future. 

So, let's get started!

# Pre-requisites
To try out Rekogniton you'll need to an [AWS account](https://docs.aws.amazon.com/rekognition/latest/dg/setting-up.html), the team at AWS have a great guide on getting your account set-up properly with roles for developers; if you're starting fresh, you'll also need to install and configure the [CLI](https://docs.aws.amazon.com/rekognition/latest/dg/setup-awscli-sdk.html) - it'll handle storing credentials for the SDKs to use with Rekognion. For experimenting with Rekognition the [full access role](https://docs.aws.amazon.com/rekognition/latest/dg/security_iam_id-based-policy-examples.html) from the docs will work well, but for anything further, stick to granting the [least possible](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) permissions.

The examples covered in this post should fall under the AWS [Free Tier](https://aws.amazon.com/free/) but it's worth checking out the [pricing page](https://aws.amazon.com/rekognition/pricing/) for any service before getting started.

# Set-up
We'll be using Python3 so we'll need the Python AWS SDK, called `boto3`, to access Rekognition as well as any other AWS services,

First, you need to install boto3 using [PIP](https://pypi.org/project/boto3/);
```bash
pip install boto3
```
Then we can import boto into a new python file and set up a client, which acts as an interface to AWS to start using the facial detection functionality;

```python
import boto3

# Create a Rekognition client
client=boto3.client('rekognition')
```

# Detection
Once you've got a client set up you can call the `detect_faces` method with an image, the response will contain a full batch of info on any faces it detects. There's two ways to load images to Rekognition: byte arrays and S3.

## S3
__The way__ to store blobs on AWS - to use with Rekognition we just specify the bucket and file, pass it to Rekognition through our client and get our response. 

```python
response = client.detect_faces(
    Image={
         'S3Object':{
             'Bucket':bucket-name,
             'Name':photo-file-name
          }
     }
)
```
Using S3 allows easier integration to other services and images can be [uploaded to S3](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/upload-objects.html) anyway you normally would.

## Byte Array
We can also [send images](https://docs.aws.amazon.com/rekognition/latest/dg/images-bytes.html/) straight to Rekognition as a byte array. You can open the files as you would load any file in Python, using the built-ins; 

```python
client=boto3.client('rekognition')

with open("photo.jpg", 'rb') as image:
    response = client.detect_faces(Image={
        'Bytes': image.read()
     })
```

You can also try using webcams and directly firing images to Rekognition, you can access them with OpenCV, this [Gist](https://gist.github.com/Nevin243/c00b1d4622933394ea2280636e5a05de) shows how you can use your Webcam with Rekognition 

## Attribute Option
There's another argument you can add to the request; `Attributes`, you have two options `ALL` or `Default`. `Default` returns BoundingBox, Confidence, Pose, Quality, and Landmarks for any faces in the image, so it _just_ detects the faces. `All`, takes a fraction longer and returns a fuller set of information for any faces, everything from `Default` and more interesting aspects like predicted emotions, age and gender.

```python
response = client.detect_faces(
    Image={
        'S3Object':{
            'Bucket':bucket-name,
            'Name':photo-file-name
        }
    },
    Attributes=['ALL']
)
```

# Response
After making the request, the response contains a set of "face details" for each face it detects in the image (up to around 40 faces). Here's a sample of an `ALL` response;

```python
{
   "FaceDetails": [ 
      { 
         "AgeRange": { 
            "High": number,
            "Low": number
         },
         "Beard": { 
            "Confidence": number,
            "Value": boolean
         },
         "BoundingBox": { 
            "Height": number,
            "Left": number,
            "Top": number,
            "Width": number
         },
         "Confidence": number,
         "Emotions": [ 
            { 
               "Confidence": number,
               "Type": "string"
            }
         ],
         "Eyeglasses": { 
            "Confidence": number,
            "Value": boolean
         },
         "EyesOpen": { 
            "Confidence": number,
            "Value": boolean
         },
         "Gender": { 
            "Confidence": number,
            "Value": "string"
         },
         "Landmarks": [ 
            { 
               "Type": "string",
               "X": number,
               "Y": number
            }
         ],
         "MouthOpen": { 
            "Confidence": number,
            "Value": boolean
         },
         "Mustache": { 
            "Confidence": number,
            "Value": boolean
         },
         "Pose": { 
            "Pitch": number,
            "Roll": number,
            "Yaw": number
         },
         "Quality": { 
            "Brightness": number,
            "Sharpness": number
         },
         "Smile": { 
            "Confidence": number,
            "Value": boolean
         },
         "Sunglasses": { 
            "Confidence": number,
            "Value": boolean
         }
      }
   ],
   "OrientationCorrection": "string"
}
```
A full breakdown of the response can be found in the [docs](https://docs.aws.amazon.com/rekognition/latest/dg/API_DetectFaces.html) but let's take a look at some of the most interesting aspects like landmarks and emotions.

## Landmarks
Landmarks are coordinates for points on the face, they're used for detecting the face itself and are points like different spots on the nose, eyes and jawline. This image from the docs shows roughly where they all are on the face:

![Man's face with facial landmarks noted](https://dev-to-uploads.s3.amazonaws.com/i/74d3g9ww4pzauwwxvz2i.png)
_From AWS docs_

Landmarks can provide a different way of understanding detected faces, to spark some ideas of interesting ways of using landmarks; here's a [rough demo](https://github.com/Nevin243/face-shaper) to work out what a male's face shape might be.

## Face Details
The details returned with the `ALL` attributes call can be really useful, these are aspects like estimated age range, predicted gender, if they've facial hair and if the person is wearing glasses. 

Rekognition returns a dict that contains a boolean for the aspect and the confidence in the analysis. Here's a snippet for parsing the response to show the dict for detecting glasses on the faces in the scene:

```python
for faceDetail in response['FaceDetails']:
    print(faceDetail['Eyeglasses'])

# Example Output
# {'Value': False, 'Confidence': 98.8663558959961}
```

Detecting aspects like glasses or age allows us to do better understand demographics to tailor services during testing or verify information about who our users could be!

## Emotions
One of the most interesting aspects of Rekognition is the emotional analysis it can do. Understanding your user with the other details is useful but being able to see and understand exactly how they're reacting can create open up some really interesting opportunities.

This snippet parses the emotions of the response and displays them all,  confidence indicates how likely the face is displaying that emotion:

```python
for faceDetail in response['FaceDetails']:
    print('Emotions: \t Confidence\n')
    for emotion in faceDetail['Emotions']:

    print(str(emotion['Type']) + '\t\t' + str(emotion['Confidence']))
    print('\n')

# Sample Output
# Emotions:     Confidence
# HAPPY		0.051588401198387146
# ANGRY		7.0355730056762695
# SAD		0.04484181851148605
# DISGUSTED	0.11442316323518753
# CONFUSED	0.2994600534439087
# FEAR		0.013996988534927368
# CALM		90.5924301147461
# SURPRISED	1.8476886749267578
```

This on can be quite fun to play around with trying to see what gets a 'Disgusted' response rather than 'Calm' but it can be really useful, being able to even get a rough sentiment read on someone interacting with your application can be really valuable. 

# Streaming, Comparing and Videos
Rekognition features are a lot broader than what's been covered here but when it comes to faces, you can stream pictures in with [kinesis](https://aws.amazon.com/kinesis/), pass batches of images or create collections of images to sample and analyse. 
You can use [compare faces] (https://docs.aws.amazon.com/rekognition/latest/dg/API_CompareFaces.html)among these data collections, check if the faces are celebrities, or even begin to track movement across images and video. 

To take any of these further you'll want to take a proper look at the [docs](https://docs.aws.amazon.com/rekognition/index.html)!

---

Hopefully, this has shown you that when used responsibly it can be a fantastic tool and how to start experimenting with facial detection and analysis! 

Want to see some really impressive Rekognition example projects to check out? There's a bunch at the official AWS samples [repo](https://github.com/aws-samples?q=rekognition&type=&language=).

If you want to learn more about AWS in general? A great place to start is their Certs here's a [post](https://marcnev.in/a-journey-through-the-amazon) I wrote after getting my first!

