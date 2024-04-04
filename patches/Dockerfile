FROM golang:alpine as builder

ARG IMAGINARY_VERSION=dev

# Installs libvips + required libraries
RUN \
  apk update && \
  apk add --repository=https://dl-cdn.alpinelinux.org/alpine/edge/testing --no-cache openslide-dev && \
  apk add --no-cache --virtual .dev-deps \
  musl-dev \
  curl meson ninja clang clang gcc g++ \
  expat-dev glib-dev libjpeg-turbo-dev libpng-dev \
  libwebp-dev tiff-dev cgif-dev libxml2-dev poppler-dev \
  swig imagemagick-dev pango-dev openslide-dev cfitsio-dev \
  libgsf-dev fftw-dev orc-dev librsvg-dev libimagequant-dev libheif-dev libexif-dev libarchive-dev libjxl-dev highway-dev && \
  cd /tmp && \
  curl -fsSL https://github.com/libvips/libvips/archive/refs/heads/master.tar.gz | tar -xz && \
  cd /tmp/libvips-master && \
	CC=clang CXX=clang++ CFLAGS="-O3" CXXFLAGS="-O3" meson setup build -Dexamples=false -Dintrospection=disabled && \
  cd build && \
  meson compile -j$(nproc) && \
  meson install

WORKDIR ${GOPATH}/src/github.com/h2non/imaginary

# Cache go modules
ENV GO111MODULE=on

COPY go.mod .
COPY go.sum .

RUN go mod download

# Copy imaginary sources
COPY . .

# Compile imaginary
RUN go build -a \
    -o ${GOPATH}/bin/imaginary \
    -ldflags="-s -w -h -X main.Version=${IMAGINARY_VERSION}" \
    github.com/h2non/imaginary

FROM alpine

ARG IMAGINARY_VERSION

LABEL maintainer="tomas@aparicio.me" \
      org.label-schema.description="Fast, simple, scalable HTTP microservice for high-level image processing with first-class Docker support" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.url="https://github.com/h2non/imaginary" \
      org.label-schema.vcs-url="https://github.com/h2non/imaginary" \
      org.label-schema.version="${IMAGINARY_VERSION}"

COPY --from=builder /usr/local/lib /usr/local/lib
COPY --from=builder /go/bin/imaginary /usr/local/bin/imaginary

# Install runtime dependencies
RUN \
  apk update && \
  apk add --no-cache --repository=https://dl-cdn.alpinelinux.org/alpine/edge/testing/ openslide && \
  apk add --no-cache procps \
    glib expat libjpeg-turbo libpng \
    libwebp tiff cgif poppler-glib libxml2 \
    imagemagick pango cfitsio \
    libgsf fftw orc librsvg libimagequant libheif libexif libarchive libjxl && \
  rm -rf /tmp/* /var/cache/apk/*

# Server port to listen
ENV PORT 9000

# Drop privileges for non-UID mapped environments
USER nobody

# Run the entrypoint command by default when the container starts.
ENTRYPOINT ["/usr/local/bin/imaginary"]

# Expose the server TCP port
EXPOSE ${PORT}
