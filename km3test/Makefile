IMG_NAME = ph2_km3test
IMG_TAG = 1.0
LOCAL_REG = localhost:5000
GITLAB_REG = git.km3net.de:4567/rbruno/ph2_dom_integration

default:
	@echo "Use"
	@echo "    image: Builds image '$(IMG_NAME):$(IMG_TAG)'"
	@echo "    regpush: Push the image into local registry '$(LOCAL_REG)/$(IMG_NAME):$(IMG_TAG)'"
	@echo "    gitlabimage: Build image tagging it for gitlab registry '$(IMG_NAME):$(IMG_TAG)'"
	@echo "    gitlabpush: Push the gitlab image into gitlab registry '$(GITLAB_REG)/$(IMG_NAME):$(IMG_TAG)'"

image:
	#@docker build --no-cache -t $(IMG_NAME):$(IMG_TAG) .
	@docker build -t $(IMG_NAME):$(IMG_TAG) .

regpush: image
	@docker tag $(IMG_NAME):$(IMG_TAG) $(LOCAL_REG)/$(IMG_NAME):$(IMG_TAG)
	@docker push $(LOCAL_REG)/$(IMG_NAME):$(IMG_TAG)

gitlabimage: image
	@docker tag $(IMG_NAME):$(IMG_TAG) $(GITLAB_REG)/$(IMG_NAME):$(IMG_TAG)

gitlabpush: gitlabimage
	docker push $(GITLAB_REG)/$(IMG_NAME):$(IMG_TAG)

test:
	@docker run -dti\
                --name $(IMG_NAME)_test\
                -h $(IMG_NAME)_test\
                --network host\
                $(IMG_NAME):$(IMG_TAG)
