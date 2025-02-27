FROM registry.ci.openshift.org/stolostron/builder:go1.20-linux AS builder

WORKDIR /workspace
# copy go tests into build image
COPY go.sum go.mod ./
COPY ./tests ./tests

# compile go tests in build image
RUN go install github.com/onsi/ginkgo/ginkgo@v1.14.2 && go mod vendor && ginkgo build ./tests/pkg/tests/

# create new docker image to hold built artifacts
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest

# run as root
USER root

# expose env vars for runtime
ENV KUBECONFIG "/opt/.kube/config"
ENV IMPORT_KUBECONFIG "/opt/.kube/import-kubeconfig"
ENV OPTIONS "/resources/options.yaml"
ENV REPORT_FILE "/results/results.xml"
ENV GINKGO_DEFAULT_FLAGS "-slowSpecThreshold=120 -timeout 7200s"
ENV GINKGO_NODES "1"
ENV GINKGO_FLAGS=""
ENV GINKGO_FOCUS=""
ENV GINKGO_SKIP="Integration"
ENV SKIP_INTEGRATION_CASES="true"
ENV IS_CANARY_ENV="true"

# install ginkgo into built image
COPY --from=builder /go/bin/ /usr/local/bin

# oc exists in the base image. copy oc into built image
COPY --from=builder /usr/local/bin/oc /usr/local/bin/oc
RUN oc version

WORKDIR /workspace/opt/tests/
# copy compiled tests into built image
COPY --from=builder /workspace/tests/pkg/tests/tests.test ./observability-e2e-test.test
COPY ./examples /examples
COPY --from=builder /workspace/tests/format-results.sh .

VOLUME /results


# execute compiled ginkgo tests
CMD ["/bin/bash", "-c", "ginkgo --v --focus=${GINKGO_FOCUS} --skip=${GINKGO_SKIP} -nodes=${GINKGO_NODES} --reportFile=${REPORT_FILE} -x -debug -trace observability-e2e-test.test -- -v=3 ; ./format-results.sh ${REPORT_FILE}"]
