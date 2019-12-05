FROM nem-catapult-toolchain as builder

ARG WORK_PATH=/tmp
ARG TMP_DIR=/tmp
ARG LIB_DIR=/usr/local
ARG WORKDIR=/opt/catapult
ARG CATAPULT_VERSION=0.9.0.1

RUN mkdir -p ${WORK_PATH} \
  && mkdir -p ${WORK_PATH}/catapult-server \
  && apt-get update -y \
  && apt-get install ninja-build apt-utils -y

# Uncomment to pull directly from the private repo
# RUN if [ ! -d ${TMP_DIR}/catapult-server-financials ]; then cd ${TMP_DIR} \
#  && git clone https://github.com/SentryLink/catapult-server-financials.git -b v${CATAPULT_VERSION} --depth 1; fi

COPY . ${TMP_DIR}/catapult-server/

WORKDIR ${WORK_PATH}

RUN cd ${TMP_DIR}/catapult-server && \
  sed -i 's/$ENV{BOOST_ROOT}/${BOOST_ROOT}/' CMakeLists.txt && \
  mkdir -p _build && cd _build \
  && cmake -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CXX_FLAGS="-pthread" \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DBOOST_ROOT=${LIB_DIR} \
		-G Ninja \
    .. \
  && ninja publish && ninja -j4

RUN mkdir -p ${TMP_DIR}/deps && mkdir -p ${TMP_DIR}/localdep \
	&& cp /usr/lib/x86_64-linux-gnu/libcrypto* ${TMP_DIR}/deps \
	&& cp /usr/lib/x86_64-linux-gnu/libgflags* ${TMP_DIR}/deps \
	&& cp /usr/lib/x86_64-linux-gnu/libsasl2* ${TMP_DIR}/deps \
	&& cp /usr/lib/x86_64-linux-gnu/libsnappy* ${TMP_DIR}/deps \
	&& cp /usr/lib/x86_64-linux-gnu/libssl* ${TMP_DIR}/deps \
	&& cp /usr/lib/x86_64-linux-gnu/libstdc* ${TMP_DIR}/deps \
	&& cp /usr/local/lib/libboost_date_time* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libboost_filesystem* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libboost_log* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libboost_program_options* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libboost_thread* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libbson* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libmongo* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/librocksdb* ${TMP_DIR}/localdep \
	&& cp /usr/local/lib/libzmq* ${TMP_DIR}/localdep

RUN mkdir -p ${WORKDIR}/bin/boost \
	&& cp ${TMP_DIR}/catapult-server/_build/bin/catapult* ${WORKDIR}/bin/ \
	&& cp ${TMP_DIR}/catapult-server/_build/bin/libcatapult* ${WORKDIR}/bin/ \
	&& cp ${TMP_DIR}/catapult-server/_build/bin/libextension* ${WORKDIR}/bin/ \
	&& cp -r ${TMP_DIR}/catapult-server/_build/bin/boost ${WORKDIR}/bin/ \
	&& cp -r ${TMP_DIR}/catapult-server/resources ${WORKDIR}/

FROM ubuntu:18.04

LABEL MAINTAINER="Alex Firmani <jafirmani@gmail.com>"
LABEL CATAPULT_VERSION=${CATAPULT_VERSION}

COPY --from=builder /opt/catapult /catapult
COPY --from=builder /tmp/deps /usr/lib/x86_64-linux-gnu
COPY --from=builder /tmp/localdep /usr/local/lib

WORKDIR /catapult

EXPOSE 7900 7901 7902

CMD ["/bin/bash"]
