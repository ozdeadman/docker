help:
	@cat Makefile

CERTS="${HOME}/.certs"
DOCKER_IMAGE_NAME=kaggle-dstl-v2
DATA?="${HOME}/kaggle/satellite-imagery/input"
GPU?=0
DOCKER_FILE=Dockerfile
DOCKER=GPU=$(GPU) nvidia-docker
BACKEND=tensorflow
PYTHON_VERSION?=3.5
CUDA_VERSION?=8.0
CUDNN_VERSION?=6
TEST=tests/
SRC?="${HOME}/kaggle/satellite-imagery-v2/src"
OLD_SRC?="${HOME}/kaggle/satellite-imagery/latestModels"

build:
	docker build -t $(DOCKER_IMAGE_NAME) --build-arg python_version=$(PYTHON_VERSION) --build-arg cuda_version=$(CUDA_VERSION) --build-arg cudnn_version=$(CUDNN_VERSION) -f $(DOCKER_FILE) .

bash: build
	$(DOCKER) run -it --rm -v $(SRC):/home/keras/src -v $(OLD_SRC):/home/keras/old_src -v $(DATA):/home/keras/data --env KERAS_BACKEND=$(BACKEND) $(DOCKER_IMAGE_NAME) bash

ipython: build
	$(DOCKER) run -it --rm -v $(SRC):/home/keras/src -v $(OLD_SRC):/home/keras/old_src -v $(DATA):/home/keras/data --env KERAS_BACKEND=$(BACKEND) $(DOCKER_IMAGE_NAME) ipython

notebook: build
	$(DOCKER) run -it --rm -v $(SRC):/home/keras/src -v $(OLD_SRC):/home/keras/old_src -v $(DATA):/home/keras/data --net=host --env KERAS_BACKEND=$(BACKEND) $(DOCKER_IMAGE_NAME)

test: build
	$(DOCKER) run -it --rm -v $(SRC):/home/keras/src -v $(OLD_SRC):/home/keras/old_src -v $(DATA):/home/keras/data --env KERAS_BACKEND=$(BACKEND) $(DOCKER_IMAGE_NAME) py.test $(TEST)

