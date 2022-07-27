# Quarto Examples with Docker

This repository contains supporting material ofr the following blog posts on the _Hosting Data Apps_ ([hosting.analythium.io](https://hosting.analythium.io/)) website:

- [How to Set Up Quarto with Docker, Part 1: Static Content]()

[Quarto](https://quarto.org/) is is an open-source scientific and technical publishing system built on [Pandoc](https://pandoc.org/).

The examples in this repository focus on R related Quarto documents. We review the different options for dockerizing static and interactive Quarto documents.

- [Quarto Examples with Docker](#quarto-examples-with-docker)
  - [Create a Quarto parent image](#create-a-quarto-parent-image)
  - [Render a static file](#render-a-static-file)
  - [Render a static project](#render-a-static-project)
    - [Website](#website)
      - [Option 1: local rendering](#option-1-local-rendering)
      - [Option 2: render inside Docker](#option-2-render-inside-docker)
    - [Book](#book)
      - [Option 1: local rendering](#option-1-local-rendering-1)
      - [Option 2: render inside Docker](#option-2-render-inside-docker-1)
  - [Render and interactive file with widgets](#render-and-interactive-file-with-widgets)

## Create a Quarto parent image

We build a parent image with Quarto installed so that we can use this image in subsequent `FROM` instructions.

The `Dockerfile.base` is based on a Ubuntu based image.

This part in the Dockerfile will [install](https://docs.rstudio.com/resources/install-quarto/#quarto-deb-file-install) the Quarto command line tool:

```dockerfile
...
RUN curl -LO https://quarto.org/download/latest/quarto-linux-amd64.deb
RUN gdebi --non-interactive quarto-linux-amd64.deb
...
```

If you want a specific version, use:

```dockerfile
...
ARG QUARTO_VERSION="0.9.522"
RUN curl -o quarto-linux-amd64.deb -L https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.deb
RUN gdebi --non-interactive quarto-linux-amd64.deb
...
```

Now we can build the image:

```bash
docker build \
    -f Dockerfile.base \
    -t analythium/r2u-quarto:20.04 .
```

Run the container interactively to check the installation:

```bash
docker run -it --rm analythium/r2u-quarto:20.04 bash
```

Type `quarto check`, you should see check marks:

```bash
root@d8377016be7f:/# quarto check

[✓] Checking Quarto installation......OK
      Version: 1.0.36
      Path: /opt/quarto/bin

[✓] Checking basic markdown render....OK

[✓] Checking Python 3 installation....OK
      Version: 3.8.10
      Path: /usr/bin/python3
      Jupyter: (None)

      Jupyter is not available in this Python installation.
      Install with python3 -m pip install jupyter

[✓] Checking R installation...........OK
      Version: 4.2.1
      Path: /usr/lib/R
      LibPaths:
        - /usr/local/lib/R/site-library
        - /usr/lib/R/site-library
        - /usr/lib/R/library
      rmarkdown: 2.14

[✓] Checking Knitr engine render......OK
```

Type `exit` to quit the session.

## Render a static file

Render the R example for [air quality](https://quarto.org/docs/computations/r.html):

```bash
docker build \
    -f Dockerfile.static-file \
    -t analythium/quarto:static-file .

docker run -p 8080:8080 analythium/quarto:static-file
```

## Render a static project

Projects contain multiple files, like [website](https://quarto.org/docs/reference/projects/websites.html) and [books](https://quarto.org/docs/reference/projects/books.html).

### Website

Use the following command to create a website template locally: `quarto create-project static-website --type website`.

#### Option 1: local rendering

Render the project as `quarto render static-website --output-dir output`.

Then use this Dockerfile:

```bash
FROM ghcr.io/openfaas/of-watchdog:0.9.6 AS watchdog
FROM alpine:latest
RUN mkdir /app
COPY static-website/output /app
COPY --from=watchdog /fwatchdog .
ENV mode="static"
ENV static_path="/app"
HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1
CMD ["./fwatchdog"]
```

#### Option 2: render inside Docker

```bash
docker build \
    -f Dockerfile.static-website \
    -t analythium/quarto:static-website .

docker run -p 8080:8080 analythium/quarto:static-website
```

### Book

Use the following command to create a book template locally: `quarto create-project static-book --type book`.

#### Option 1: local rendering

Render the project as `quarto render static-book --output-dir output`.

Then use this Dockerfile:

```bash
FROM ghcr.io/openfaas/of-watchdog:0.9.6 AS watchdog
FROM alpine:latest
RUN mkdir /app
COPY static-book/output /app
COPY --from=watchdog /fwatchdog .
ENV mode="static"
ENV static_path="/app"
HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1
CMD ["./fwatchdog"]
```

#### Option 2: render inside Docker

Note: we need a LaTeX installation, so we add `quarto install tool tinytex` to the Dockerfile:

```bash
docker build \
    -f Dockerfile.static-book \
    -t analythium/quarto:static-book .

docker run -p 8080:8080 analythium/quarto:static-book
```

## Render and interactive file with widgets

Render an [interactive](https://quarto.org/docs/interactive/widgets/htmlwidgets.html) but static file:

```bash
docker build \
    -f Dockerfile.static-widget \
    -t analythium/quarto:static-widget .

docker run -p 8080:8080 analythium/quarto:static-widget
```
