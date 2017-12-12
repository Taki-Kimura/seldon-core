# How to use seldon wrappers 

### Preliminary steps 

Clone seldon-core, install grpc tools and buld the protobuffers (if not done before) 

* Run ```git clone seldon-core```
* Run ```python -m pip install grpcio-tools==1.1.3```
* Build the protos. You only have to do this once: run ```cd seldon-core/wrappers && make build_protos```

### Wrap model

* Run ```cd python && python wrap_model.py <path_to_your_model_folder> <your_model_name> <your_model_version> <your_docker_repo> --base-image <your_base_image>```

* Run ```cd <path_to_your_model_folder>/build && make build_docker_image```. This will  build a docker image of your model locally ready to be deployed with seldon-core

    
### Notes:


*   \<path_to_your_model_folder>: Your local path to the folder with your model. In the model  folder you need the files

	* \<your_model_name>.py: Needs to include a python class having the same name as the file, i,e. \<your_model_name>, and implementing the  methods \__init__  and predict.
	The following example show the content of a model file model_name.py that load a keras model previusly saved in h5 format.
	
	    	from keras.models import load_model
	
	    	class model_name(object):

	        	def __init__(self):
                    self.model = load_model('saved_model_folder/saved_model.h5) 

		        def predict(self,X,features_names):
		            return self.model.predict(X) #return predictions for batch X

	* requirements.txt: Lists the requirements needed fot you model. For the keras model above the requirements.txt file would be:
	
		    keras==2.0.6 
		    h5py
 	    	
	* saved_model.h5: The file with your saved model. The format of the file depends on the tool you used to create, train and saved the model. In this case is a h5 model crated with keras.
	
* \<your_model_name>: The name of the .py file with your model. Has to me the same as the name of the python class implemented in the file, e.g MnistClassifier.

* \<your_model_version>: The version of your model, e.g.  0.0.

* \<your_docker_repo>: The repository for the image, e.g. seldonio.

* \<your_base_image>: THe base image for the model, default is Python:2.


# Example of usage.

Here we include a step-by-step guide to train, save and wrap a mnist classifier from scratch. The model is in ```seldon-core-plugins/keras_mnist``` and it is builded using keras.

### Preliminary steps

* Have seldon-core and seldon-core-plugins folders in the same directory and install grpc tools (if not done before).
* Run ```git clone seldon-core```
* Run ```git clone seldon-core-plugins```
* Run ```python -m pip install grpcio-tools==1.1.3```

### Train and save keras mnist classifier

* Run ```cd seldon-core-plugins/keras_mnist && python train_mnist.py```. This will train a keras convolutional neural network on mnist dataset for 2 epochs and save the model in the same folder.

### Wrap saved model

* Run ```cd ../../seldon-core/wrappers && make build_protos``` (if not done before).

* Run ```cd python && python wrap_model.py ../../../seldon-core-plugins/keras_mnist MnistClassifier 0.0 seldonio```. This will create the folder build in keras_mnist. The --base-image argument is not specified and the wrapper will use the default base image Python:2.

* Run ```cd ../../../seldon-core-plugins/keras_mnist/build/ && make build_docker_image```. This will create the docker image ```seldonio/mnistclassifier:0.0``` which is ready for deployment with seldon-core.