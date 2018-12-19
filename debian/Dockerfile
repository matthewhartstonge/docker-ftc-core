# The Debian build uses a multi-stage build as debian deps clean up gets gnarly
# with apt and the forced upgrade to perl.
# As such, the flow is broken up to better utilize build caching, as the only
# thing that should change is the version of feathercoin!
FROM debian:9-slim AS buildr
LABEL maintainer Matthew Hartstonge <matt@mykro.co.nz>

ENV BUILD_DIR /tmp
ENV BUILD_JOBS 4
ENV BUILD_PACKAGES autoconf automake binutils-gold bzip2 bsdmainutils ca-certificates cmake curl git g++-multilib libtool make patch pkg-config python3 wget
ENV BUILD_PLATFORM x86_64-linux-gnu
ENV DB_VERSION 4.8.30.NC
ENV DB_SHASUM 12edc0df75bf9abd7f82f821795bcee50f42cb2e5f76a6a281b85732798364ef

RUN apt-get update && apt-get install --no-install-recommends --no-upgrade -y ${BUILD_PACKAGES} \
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
    && make install

# Obtain and build Feathercoin
ENV FTC_VERSION v0.17.0.1
RUN git clone --depth 50 https://github.com/feathercoin/feathercoin.git ${BUILD_DIR}/feathercoin \
    && cd ${BUILD_DIR}/feathercoin \
    && git fetch --tags \
    && git checkout ${FTC_VERSION} \
    && sed -i '/AC_PREREQ/a\AR_FLAGS=cr' src/univalue/configure.ac \
    && sed -i '/AX_PROG_CC_FOR_BUILD/a\AR_FLAGS=cr' src/secp256k1/configure.ac \
    && sed -i s:sys/fcntl.h:fcntl.h: src/compat.h \
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
    && strip -s ${BUILD_DIR}/feathercoin/depends/${BUILD_PLATFORM}/bin/feathercoind \
    && strip -s ${BUILD_DIR}/feathercoin/depends/${BUILD_PLATFORM}/bin/feathercoin-cli \
    && strip -s ${BUILD_DIR}/feathercoin/depends/${BUILD_PLATFORM}/bin/feathercoin-tx \
    && mkdir /dist \
    && cp ${BUILD_DIR}/feathercoin/depends/${BUILD_PLATFORM}/bin/feather* /dist/

# Deploy container
FROM debian:9-slim
COPY --from=buildr /dist /usr/local/bin
EXPOSE 9336 9337 19336 19337
CMD ["feathercoind"]
