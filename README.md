# MLOps : machine learning and operations
***This document discusses techniques for implementing and automating continuous integration (CI) of machine learning (ML) system using jenkins***  

This project documentation is for data scientists and ML engineers who want to apply **DevOps** principles to ML systems **(MLOps)**. **MLOps** is an ML engineering culture and practice that aims at unifying ML system development (Dev) and ML system operation (Ops). Practicing MLOps means that you advocate for automation and monitoring at all steps of ML system construction, including integration, testing etc.

# ***INITIAL SETUP***
 * install linux OS(espcially redhat) 
 * install docker in linux.To install [click here](https://docs.docker.com/engine/install/)
 * install jenkins in linux.Information regarding install [click here](https://www.jenkins.io/download/)
 * install git in linux.To install [click here](https://git-scm.com/download/linux)

# ***Setting up Docker images***
 Build the images using the dockerfile given in Dockerfile folder 
 here we created the two images both with different libraries installed, keras and sckit-learn respectively.
 We will be configuring the image in such a way that as soon as the container is launched using these image along with the model it should automatically starts to train the model.
 ![](screenshots/docker file for sckit-learn.png)
 ![](screenshots/docker file for keras.png)
     
     - docker build -t your_image_name:version .
      
# ***JENKINS***  
1) **JOB1**:
Checks the github repository for every one minute and if any developer pushes the code to github, jenkins automatically downloads the code and datasets and then copy it to developer folder in base system.
   
![](screenshots/j11.png)
![](screenshots/j12.png)
![](screenshots/j13.png)


2) **JOB2**:
This job checks the downloaded the code and analyses wheather it's a CNN/ANN/LINEAR_REGRESSION code and lauches respective docker container in linux system for training the model along with the model.
The code is built in such a way that as soon as it is trained, it's final accuracy will be saved in accuracy.txt file in the same folder which will be required later.

       #copy the above code in execute shell
       if [ '$(sudo cat /root/developer/kears.py)|sudo grep Convolution2D' ]
       then
         sudo docker run -t -v  /root/developer/:/hello/ --rm keras:v1 /hello/keras.py 
       elif [ '$(sudo cat /root/developer/kears.py)|sudo grep sklearn' ]
       then
         sudo docker run -t -v /root/developer/:/hello/ --rm   sklearn:v1 /hello/keras.py
       fi
![](screenshots/j21.png)
![](screenshots/j22.png)


3) **JOB3**:
Checks the accuracy of the trained model and if the accuarcy is not over 90% then **JOB3** tweak the architecture of the model by adding the layers or by changing the number of epochs and pushes back the code to github to retrain the model for  better accuracy.This cycle goes until accuracy of 90% is not acchieved.
when the desired accuracy is achieved then **JOB4** is triggered.
        
       sudo cd /root/developer
       requiredacc=0.90    #our required accuracy i.e 90%
       if [ '$accInt -ge $requiredacc' ]
       then 
         exit 0
       else
         #random in between 123
         x=1
         y=2
         DIFF=$((y+x))
         accInt=$(sudo cat /root/developer/accuracy.txt)      #as accuracy will be in a string datatype we will convert it into integer
         while [ $accInt -lt $requiredacc ] 
         do
            case $(echo  $(($(($RANDOM%DIFF))+x)))     #randomly choses a number in between 1,2 and 3. And if its "1" or "3" it will add the layers else if its "2" it will change the number of epochs.
	         in
	       1)
	       sudo sed -i "32i model.add(Convolution2D(48,(3,3),activation='relu'))" keras.py
	       ;;
	       2)
	       sudo sed 's/epoch = 10/epoch = 20/g' keras.py
	       ;;
           3)
	       sudo sed -i "33i model.add(Convolution2D(48,(3,3),activation='relu'))" keras.py
                        sed -i "34i model.add(MaxPoling2D(pool_size=(2,2))" keras.py
	       ;;
	    esac	
         done
       fi
       yes| sudo cp -f /root/developer/keras.py '/root/keras-CNN'
       sudo 'cd /root/keras-CNN'
       sudo git commit keras.py -m "hello"
       sudo git push
       exit 1

![](screenshots/j31.png)


4)**JOB4**:
Notifies the developer that the best model has been impelmented through email.

![](screenshots/j41.png)

5)**JOB5**:
jenkins job5 takes the updated code and trained model weights and  updates it to github repository.




# Finally the build pipeline view of the above jenkins job is:
 
 
![](screenshots/j51.png)
![](screenshots/j51.png)




***In collobration with : madan naik.***
