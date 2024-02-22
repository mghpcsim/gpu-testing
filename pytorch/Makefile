PATH_STORAGE ?= "/scratch"

NAME_NGC=pytorch:23.06-py3
NAME_DATASET=object_detection
NAME_CONFIG=g4dn.4xlarge
NAME_MODEL=ssd_amp
TIME_OUT=3000
PATH_OUTPUT=~/results/${NAME_CONFIG}/PyTorch_SSD_AMP


install:
	docker pull nvcr.io/nvidia/${NAME_NGC} && \
	cd ${PATH_STORAGE} && \
	wget https://raw.githubusercontent.com/mghpcsim/gpu-testing/master/pytorch/setup.sh && \
	chmod +x setup.sh && \
	./setup.sh ${NAME_NGC}


create_data:
	cd ${PATH_STORAGE}/deeplearning-benchmark/pytorch && \
	docker run --gpus all --rm -t --shm-size=31g \
		-v ${PATH_STORAGE}/DeepLearningExamples/PyTorch:/workspace/benchmark \
		-v ${PATH_STORAGE}/data:/data \
		-v ${PATH_STORAGE}/deeplearning-benchmark/pytorch/scripts:/scripts \
		nvcr.io/nvidia/${NAME_NGC} \
		/bin/bash -c "cp -r /scripts/* /workspace;  ./run_prepare.sh ${NAME_DATASET}"


benchmark:
	cd ${PATH_STORAGE}/deeplearning-benchmark/pytorch && \
	docker run \
		--rm --shm-size=31g \
		--gpus all \
		-v ${PATH_STORAGE}/DeepLearningExamples/PyTorch:/workspace/benchmark \
		-v ${PATH_STORAGE}/data:/data \
		-v ${PATH_STORAGE}/deeplearning-benchmark/pytorch/scripts:/scripts \
		-v ${PATH_STORAGE}/deeplearning-benchmark/pytorch/results:/results \
		nvcr.io/nvidia/${NAME_NGC} \
		/bin/bash -c "cp -r /scripts/* /workspace; ./run_benchmark.sh ${NAME_CONFIG} ${NAME_MODEL} ${TIME_OUT}" && \
	grep -r '^Training performance = ' ${PATH_STORAGE}/deeplearning-benchmark/pytorch/${PATH_OUTPUT}/*.txt

	
