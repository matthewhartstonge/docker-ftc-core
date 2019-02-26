FROM alpine:latest
LABEL maintainer="Matthew Hartstonge <matt@mykro.co.nz>"

ENV BUILD_DIR /tmp
ENV BUILD_JOBS 4
ENV BUILD_PLATFORM x86_64-linux-musl
ENV DB_VERSION 4.8.30.NC
ENV DB_SHASUM 12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef
ENV FTC_VERSION v0.17.0.2

# Add musl monkey patches...
COPY patches /patches/
RUN apk add --no-cache --virtual .build-deps \
        # Build tooling
        autoconf \
        automake \
        curl \
        g++ \
        git \
        gzip \
        libtool \
        make \
        tar \
    # Obtain and build BerkleyDB
    && wget -P ${BUILD_DIR} http://download.oracle.com/berkeley-db/db-${DB_VERSION}.tar.gz \
    && (echo "${DB_SHASUM}  ${BUILD_DIR}/db-${DB_VERSION}.tar.gz" | sha256sum -c) \
    && tar -xvf ${BUILD_DIR}/db-${DB_VERSION}.tar.gz -C ${BUILD_DIR}/ \
    && cd ${BUILD_DIR}/db-${DB_VERSION}/build_unix \
    && sed s/__atomic_compare_exchange/__atomic_compare_exchange_db/g -i ${BUILD_DIR}/db-${DB_VERSION}/dbinc/atomic.h \
    && ../dist/configure \
        --prefix=/usr/ \
        --includedir=/usr/include/db4 \
        --enable-cxx \
        --enable-compat185 \
        --disable-shared \
        --with-pic \
    && make install \
    # Obtain and build Feathercoin
    && git clone --depth 50 https://github.com/feathercoin/feathercoin.git ${BUILD_DIR}/feathercoin \
    && cd ${BUILD_DIR}/feathercoin \
    && git fetch --tags \
    && git checkout ${FTC_VERSION} \
    && sed -i '/AC_PREREQ/a\AR_FLAGS=cr' src/univalue/configure.ac \
    && sed -i '/AX_PROG_CC_FOR_BUILD/a\AR_FLAGS=cr' src/secp256k1/configure.ac \
    && sed -i s:sys/fcntl.h:fcntl.h: src/compat.h \
    && mkdir -p ${BUILD_DIR}/feathercoin/depends/patches/openssl \
    && cp -f /patches/0001-fix-for-musl.patch ${BUILD_DIR}/feathercoin/depends/patches/openssl/0001-fix-for-musl.patch \
    && cp -f /patches/openssl.mk ${BUILD_DIR}/feathercoin/depends/packages/openssl.mk \
    && cp -f /patches/0002-disable-pthread_set_name_np.patch ${BUILD_DIR}/feathercoin/depends/patches/zeromq/0002-disable-pthread_set_name_np.patch \
    # Build required dependencies...
    && cd ${BUILD_DIR}/feathercoin/depends \
    && make HOST=${BUILD_PLATFORM} NO_QT=1 -j${BUILD_JOBS} \
    ## Configure and build Feathercoin
    && cd ${BUILD_DIR}/feathercoin \
    && ./autogen.sh \
    && ./configure \
        --prefix=${BUILD_DIR}/feathercoin/depends/${BUILD_PLATFORM} \
        --disable-shared \
        --disable-asm \
        --enable-static \
        --without-gui \
    && make -j${BUILD_JOBS} \
    && make install \
    # Shrink and install binaries...
    && strip -s $BUILD_DIR/feathercoin/depends/${BUILD_PLATFORM}/bin/feathercoind \
    && strip -s $BUILD_DIR/feathercoin/depends/${BUILD_PLATFORM}/bin/feathercoin-cli \
    && strip -s $BUILD_DIR/feathercoin/depends/${BUILD_PLATFORM}/bin/feathercoin-tx \
    && cp $BUILD_DIR/feathercoin/depends/${BUILD_PLATFORM}/bin/feather* /usr/bin/ \
    # Clean up dev dependencies
    && cd ${BUILD_DIR}/db-${DB_VERSION}/build_unix \
    && make uninstall \
    && rm -Rf ${BUILD_DIR} \
    && apk del .build-deps \
    # Install required libs
    && apk add --no-cache \
        libgcc \
        libstdc++

EXPOSE 9336 9337 19336 19337
CMD ["feathercoind"]