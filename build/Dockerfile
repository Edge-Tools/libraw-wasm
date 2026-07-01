FROM emscripten/emsdk:5.0.7 AS builder

ENV DEBIAN_FRONTEND=noninteractive

ARG VERSION=0.21.4
ENV VERSION=${VERSION}
ARG SOURCE_SHA256=6be43f19397e43214ff56aab056bf3ff4925ca14012ce5a1538a172406a09e63

WORKDIR /src

COPY js /opt/js

ADD --checksum=sha256:${SOURCE_SHA256} \
    https://www.libraw.org/data/LibRaw-${VERSION}.tar.gz libraw.tar.gz
RUN tar -xzf libraw.tar.gz && rm libraw.tar.gz

WORKDIR /src/LibRaw-${VERSION}

ENV LIBRAW_CXXFLAGS="-O2 -fexceptions -std=gnu++17 -I. -w -DLIBRAW_NOTHREADS"

RUN mkdir -p object lib bin \
    && emmake make -f Makefile.dist lib/libraw.a -j"$(nproc)" \
         CC=emcc CXX=em++ AR=emar RANLIB=emranlib \
         CFLAGS="${LIBRAW_CXXFLAGS}" LDADD="" \
    && [ -f lib/libraw.a ] || { echo "FATAL: lib/libraw.a not produced"; exit 1; } \
    && ls -lh lib/libraw.a

ENV LIBRAW_LDFLAGS="\
    --post-js /opt/js/post.js \
    -O2 \
    -fexceptions \
    -sALLOW_MEMORY_GROWTH=1 \
    -sINITIAL_MEMORY=67108864 \
    -sMAXIMUM_MEMORY=1073741824 \
    -sSTACK_SIZE=8388608 \
    -sEXIT_RUNTIME=0 \
    -sFORCE_FILESYSTEM=1 \
    -sEXPORTED_RUNTIME_METHODS=callMain,FS,ENV \
    -sINVOKE_RUN=0 \
    -sMODULARIZE=1 \
    -sEXPORT_ES6=1 \
    -sERROR_ON_UNDEFINED_SYMBOLS=0 \
"

RUN set -eux; \
    mkdir -p /dist; \
    link_sample() { \
      out="$1"; sample="$2"; \
      em++ ${LIBRAW_CXXFLAGS} \
        "samples/${sample}.cpp" lib/libraw.a -lm \
        ${LIBRAW_LDFLAGS} -o "/dist/${out}.mjs"; \
    }; \
    link_sample libraw       dcraw_emu; \
    link_sample raw-identify raw-identify; \
    ls -lh /dist

RUN set -eux; \
    for out in libraw raw-identify; do \
      MJS="/dist/${out}.mjs"; WASM="/dist/${out}.wasm"; \
      [ -f "$MJS" ]  || { echo "FATAL: ${MJS} missing";  exit 1; }; \
      [ -f "$WASM" ] || { echo "FATAL: ${WASM} missing"; exit 1; }; \
      grep -q "export default" "$MJS" \
        || { echo "FATAL: ${out}.mjs is not an ES6 module"; exit 1; }; \
      grep -q "${out}.wasm" "$MJS" \
        || { echo "FATAL: ${out}.mjs lost its wasm reference"; exit 1; }; \
      [ "$(head -c4 "$WASM" | od -An -tx1 | tr -d ' \n')" = "0061736d" ] \
        || { echo "FATAL: ${out}.wasm has no wasm magic"; exit 1; }; \
      [ "$(stat -c%s "$WASM")" -gt 262144 ] \
        || { echo "FATAL: ${out}.wasm is implausibly small"; exit 1; }; \
    done; \
    printf '%s\n' "${VERSION}" > /dist/LIBRAW_VERSION; \
    ls -lh /dist

COPY test/smoke.mjs ./smoke.mjs
COPY test/fixtures/sample.kdc ./sample.kdc
RUN cp /dist/libraw.mjs /dist/libraw.wasm \
       /dist/raw-identify.mjs /dist/raw-identify.wasm ./ \
    && node smoke.mjs

FROM scratch AS exportor
COPY --from=builder /dist /dist
