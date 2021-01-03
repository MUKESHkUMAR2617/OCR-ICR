# OCR-ICR
## OCR OpenCV in Forms
#### We will first use feature extraction to find the form and then use tesseract to recognize the characters. The good thing about this techniques is that unlike the previous Scanner we did, it will be able to detect the form even if it is not completely in the picture. This is because we are not relying of contour detection. So let’s get started.

## Packages

#### A couple of packages are required to create this project. All of these packages can be installed within the Pycharm IDE in the virtual environment, except for one. We will install the pytesseract via pre-built binary package available on their website ( windows installer) as we will use windows operating system. You can also install it on macos, linux or raspbian (more information can be found on their website).

Once installed we have to get the path of the tesseract executable file that we will link in our python script. If you keep the default installation directory, then it should be in C – Program Files- Tesseract-OCR. So the path will be as follows:

‘C:\\Program Files\\Tesseract-OCR\\tesseract.exe’

Once we have the path now we can create a new project in pycharm and install the necessary packages. Below is the list of the packages that need to be installed for this project:

opencv-pyhton
numpy
pytesseract
os

The first three need to be installed manually whereas the os is available by default. To install you can simply go to

Now we can import these packages and link to our tesseract executable file.

# import cv2
# import numpy as np
# import pytesseract
# import os
 
# pytesseract.pytesseract.tesseract_cmd = 'C:\\Program Files\\Tesseract-OCR\\tesseract.exe'


## Feature Detection Import Image

The first step would be to import the form template which we will refer to as the Query Image. So in other words the query image is the blank form. So rather than getting a form from google, this time I designed it myself and yes its the worst design ever. Well there is a reason for that. Since we want to find the query image in our new images we want some good amount of feature matching. So if we have a very minimalistic design with very little icons and text, it would be very hard to match.

The idea behind the form is to register your self as an awesome person, therefore the Awesomeness Form. Here you will find two types of inputs, Text and Check Box. This will allow us to practice the usage of different types of input fields.

We will us the following line of code to import the image.

     imgQ = cv2.imread('Query.png')

## Feature of Query Image

Once we have the Query Image loaded we will get the features for it, that will later help us find and align the new images/forms. We will use the same method for feature extraction, The idea is to find the features and their descriptors using the ORB detector and later compare them to the new image descriptors.

Now we will initialize our detector . There are many types of detectors available. Some are free and some require license for commercial use. The most common ones include ORB, SIFT and SURF. We will be using the ORB detector since it is free to use . To learn more about different detectors you can visit the opencv documentation.

## Importing User Forms

Now that we have the key points of our query image, we need to import the user filled forms and get the features and descriptors for them as well. To make the process simple we will add all user forms in a folder and write some code to automatically extract all the images. To do this will will need the os package.


## Aligning The Forms
Since the our user forms are not in the correct alignment, we cannot extract the text information yet. So we will first align these images using the key points from both our images.

Given we have a few points in our user form and the location of same points in the query image we can find the relationship between them. This relationship is basically a matrix, and the process of finding it is know as Homography. Using this relationship we can align our user form.

## Text Detection

Now that our forms are aligned we can extract the text and send it for recognition. So the data that we will be getting will be stored in a list, this will allow us to store it in a file later on. At this point it’s good to add a print message with the information of which form is being processed.

## ROI Selector
Now comes the fun part . We will loop through all the Regions of Interest (ROI) i.e the input fields to find the text in them . But here we need to answer two question .

How to get the ROI for the form?
How to handle checkbox input?
For the first question you can simply get the roi from an image editing software like paint or photoshop and write them down in a list. But this seems like an inefficient way to do it. So to solve this problem I have written a simple script that opens the form and asks the user to click on the desired region of interests and saves these values in a list.

It also allows us to define wether the input field is Text based or Checkbox. This solves our problem 2 as well. Below is the code for Roi Selector.

      import cv2
      import random

      path= 'Query.pdf'
      scale = 0.4

      circles = []
      counter = 0
      counter2 = 0
      point1=[]
      point2=[]
      myPoints = []
      myColor=[]
      def mousePoints(event,x,y,flags,params):
          global counter,point1,point2,counter2,circles,myColor
          if event == cv2.EVENT_LBUTTONDOWN:
              if counter==0:
                  point1=int(x//scale),int(y//scale);
                  counter +=1
                  myColor = (random.randint(0,2)*200,random.randint(0,2)*200,random.randint(0,2)*200 )
              elif counter ==1:
                  point2=int(x//scale),int(y//scale)
                  type = input('Enter Type')
                  name = input ('Enter Name ')
                  myPoints.append([point1,point2,type,name])
                  counter=0
              circles.append([x,y,myColor])
              counter2 += 1

      img = cv2.imread(path)
      img = cv2.resize(img, (0, 0), None, scale, scale)

      while True:
          # To Display points
          for x,y,color in circles:
              cv2.circle(img,(x,y),3,color,cv2.FILLED)
          cv2.imshow("Original Image ", img)
          cv2.setMouseCallback("Original Image ", mousePoints)
          if cv2.waitKey(1) &amp; 0xFF == ord('s'):
              print(myPoints) 
       
       
We can add a new list by the name roi and place this output in it.

    roi = [[(102, 977), (682, 1079), ‘text’, ‘Name’],
    [(742, 979), (1319, 1069), ‘text’, ‘Phone’],
    [(99, 1152), (144, 1199), ‘box’, ‘Sign’],
    [(742, 1149), (789, 1197), ‘box’, ‘Allergic’],
    [(102, 1419), (679, 1509), ‘text’, ‘Email’],
    [(742, 1419), (1317, 1512), ‘text’, ‘Id’],
    [(102, 1594), (672, 1684), ‘text’, ‘City’],
    [(744, 1589), (1327, 1682), ‘text’, ‘Country’]]


Now we can loop through each of these rois and find the relevant infromaiton. To make sure there are not mistakes in the roi we will display all of the regions. To do this we will first create rectangles on our mask image and then blend it with the original image.

     for x,r in enumerate(roi):
          
        # For displaying the rois
        cv2.rectangle(imgMask, (r[0][0], r[0][1]), (r[1][0], r[1][1]), (0, 
        255,0), cv2.FILLED)
        imgShow = cv2.addWeighted(imgShow, 0.99, imgMask, 0.1, 0)

# OCR with Pytesseract
Regions of Interest Cropped
Now we will check if the roi is text based or checkbox, since they will have different processing method.

For the text we will input it to our pytesseract function. And once we get the result we will append it to the myData list. We can also apply some preprocessing techniques here to enhance the recognition process, but since our images are queit clear, we don’t need that.

     if r[2] == 'text':
            print('{}: {}'.format(r[3],pytesseract.image_to_string(imgCrop)))
            myData.append(pytesseract.image_to_string(imgCrop))

## Check Box Input
For the checkbox we have a few simple steps. First we will convert the image to gray scale . Then we will convert it into a binary image using thresholding. Now we can simply count the number of non zero pixel and compare it to a threshold to declare it checked or not checked. Lastly we can append this information to our myData list.

      if r[2] == 'box':
        imgWarpGray = cv2.cvtColor(imgCrop, cv2.COLOR_BGR2GRAY)  
        imgThresh = cv2.threshold(imgWarpGray, 170, 255, 
        cv2.THRESH_BINARY_INV)[1]  # APPLY THRESHOLD AND INVERSE
        totalPixels = cv2.countNonZero(imgThresh)
        if totalPixels>minThreshold:totalPixels=1
        else:totalPixels=0
        print(f'{r[3]}: {totalPixels}')
        myData.append(totalPixels)


## Saving Data to File
Here is an optional part where we save the data in a file. We can create a table with all the information in it of different forms. Note that this has to be outside the roi for loop since we want to store all of the info of a given form at once i.e not region by region.

      with open('DataOutput.csv','a+') as f:
              for data in myData:
                  f.write(str(data)+',')
              f.write('\n')

        
 
        
