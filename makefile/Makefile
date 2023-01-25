VERSION=`cat VERSION.txt | tr -d '\n'`
CURRENT_BRANCH=`git rev-parse --abbrev-ref HEAD | tr -d '\n'`
BUILD_PLATFORMS=linux/amd64,linux/arm64/v8
ORIGIN_ORG=`git config --get remote.origin.url | sed -E 's/^git@.*:([[:graph:]]*)\/([[:graph:]]*).git/\1/' | tr -d '\n'`
ORIGIN_REPO=`git config --get remote.origin.url | sed -E 's/^git@.*:([[:graph:]]*)\/([[:graph:]]*).git/\2/' | tr -d '\n'`
UPSTREAM_ORG=`git config --get remote.upstream.url | sed -E 's/^git@.*:([[:graph:]]*)\/([[:graph:]]*).git/\1/' | tr -d '\n'`
UPSTREAM_REPO=`git config --get remote.upstream.url | sed -E 's/^git@.*:([[:graph:]]*)\/([[:graph:]]*).git/\2/' | tr -d '\n'`

require-clean-tree:
	if git diff --quiet HEAD; then \
  		echo 'Tree is clean'; \
  	else \
  		echo 'Uncommitted changes...aborting'; \
  		exit 1; \
	fi

promotion-alpha: require-clean-tree
	git checkout main && \
	git pull && \
	export ALPHA_VERSION="$(VERSION)" && \
	export PR_BRANCH="promote-v${ALPHA_VERSION}" && \
	git checkout -b "${PR_BRANCH}" && \
	echo "Promoting v${ALPHA_VERSION}..." && \
	BUILDRUNNER_BUMP_TYPE=alpha buildrunner -f buildrunner/buildrunner.yaml -s bump-version && \
	export MINOR_VERSION="$(VERSION)" && \
	echo "Promoted v${ALPHA_VERSION} to v${MINOR_VERSION}" && \
	git add . && \
	git commit -m "Promote v${ALPHA_VERSION} to v${MINOR_VERSION}" && \
	git push -u origin HEAD && \
	gh pr create --web --base main --head ${PR_BRANCH} --fill --draft

github-release: require-clean-tree
	git checkout main && \
	git pull && \
	gh release create --web --draft --generate-notes --target main --title "Release v$(VERSION)" "v$(VERSION)"

github-image: require-clean-tree
	git pull upstream HEAD && \
	export IMAGE_NAME="ghcr.io/$(UPSTREAM_ORG)/$(UPSTREAM_REPO)" && \
	export IMAGE_TAG="${IMAGE_NAME}:$(VERSION)" && \
	echo Building ${IMAGE_TAG} && \
	echo "${GITHUB_CR_PAT}" | docker login ghcr.io -u $(UPSTREAM_ORG) --password-stdin && \
	docker buildx create --use && \
	docker buildx build . \
	  --push \
	  --platform ${BUILD_PLATFORMS} \
	  --tag ${IMAGE_TAG} && \
	git checkout main && \
	git pull upstream main

github-branch: require-clean-tree
	git checkout main && \
	git pull && \
	export CURRENT_VERSION="$(VERSION)" && \
	export VERSION_RELEASE_BRANCH="$(echo -n "$CURRENT_VERSION" | sed -E 's/(^[[:digit:]]+\.[[:digit:]]+\.)[[:digit:]]+a?$/\1x/')" && \
	git checkout -b "${VERSION_RELEASE_BRANCH}" && \
	git push -u origin HEAD

version-alpha: require-clean-tree
	git checkout main && \
	git pull && \
	BUILDRUNNER_BUMP_TYPE=minor buildrunner -f buildrunner/buildrunner.yaml -s bump-version && \
	export PR_BRANCH="move-main-to-v$(VERSION)" && \
	git checkout -b "${PR_BRANCH}" && \
	git add . && \
	git cm "Move main to v$(VERSION)" && \
	git push -u origin HEAD && \
	gh pr create --web --base main --head ${PR_BRANCH} --fill --draft

promotion-patch: require-clean-tree
	git pull upstream HEAD && \
	export BASE_BRANCH="$(CURRENT_BRANCH)" && \
	export BASE_VERSION="$(VERSION)" && \
	echo "Patching v${BASE_VERSION}..." && \
    BUILDRUNNER_BUMP_TYPE=patch buildrunner -f buildrunner/buildrunner.yaml -s bump-version && \
    BUILDRUNNER_BUMP_TYPE=alpha buildrunner -f buildrunner/buildrunner.yaml -s bump-version && \
	export PATCH_VERSION="$(VERSION)" && \
	export PR_BRANCH="patch-v${PATCH_VERSION}" && \
	git checkout -b ${PR_BRANCH} && \
	echo "Patched v${BASE_VERSION} to v${PATCH_VERSION}" && \
	git add . && \
	git commit -m "Patch v${BASE_VERSION} to v${PATCH_VERSION}" && \
	git push -u origin HEAD && \
	gh pr create --web --base ${BASE_BRANCH} --head ${PR_BRANCH} --fill --draft && \
	git checkout $BASE_BRANCH

github-release-patch: require-clean-tree
	git pull upstream HEAD && \
	export PATCH_VERSION="$(VERSION)" && \
	gh release create --web --draft --generate-notes --target $(CURRENT_BRANCH) --title "Patch v${PATCH_VERSION}" "v${PATCH_VERSION}"

pull-request-main: require-clean-tree
	git push -u origin HEAD && \
	gh pr create --base main --fill --draft

venv:
	python3 -m venv .venv && \
	source .venv/bin/activate && \
	pip install --upgrade pip build setuptools wheel && \
	pip install -r requirements.txt

build-image:
	export IMAGE_NAME="ghcr.io/$(UPSTREAM_ORG)/$(UPSTREAM_REPO)" && \
	export IMAGE_TAG="${IMAGE_NAME}:$(VERSION)" && \
	echo Building ${IMAGE_TAG} && \
	docker build . \
	  --tag ${IMAGE_TAG}