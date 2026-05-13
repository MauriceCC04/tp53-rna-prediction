rewrite this readme

to run:
install the libraries

i have made it runnable on kaggle with the 2xT4 gpu setup

these are the following attached kaggle datasets as to not upload the data:
ccle:
https://kaggle.com/datasets/0caa0d93ad3c5539a0e438c7d5912de8f74db23c35005e980565a063040797a1

tcga:
https://kaggle.com/datasets/1afc69a244bb09cbd7cb88dfc472a2d173433b6cc7f1d56d93d7da12bab0fdb9

the eda notebooks can run fine locally but the modelling notebooks need to be run via some gpu setup
doesnt have to be kaggle

additionally, because of this, you must edit the file paths in the notebooks to point to the correct location of the datasets on your local machine or kaggle environment.
the modelling notebook portion of the notebooks should be fine from kaggle, but if you run the eda you must download it to whatever environment you run it on.

this was done this way because this project was split into several notebooks but combined into one for submission