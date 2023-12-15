FROM ghcr.io/openfaas/of-watchdog:0.9.13 as watchdog

FROM rust:1.72 as builder

WORKDIR /home/rust

# Copy all the sources. So function is at /home/rust/function
COPY function ./function           

# ENV CARGO_TARGET_DIR=/home/rust/target
# WORKDIR /home/rust/function
#RUN rustup target add x86_64-unknown-linux-gnu
# RUN --mount=type=cache,target=/usr/local/cargo/registry cargo build

RUN cd function && cargo install --path .

FROM debian:bookworm-slim
RUN apt-get update & apt-get install -y extra-runtime-dependencies & rm -rf /var/lib/apt/lists/*
COPY --from=builder /usr/local/cargo/bin/function /usr/bin/function

# FROM alpine:3.17.3 as runner 
# 
# # Install packages and add non-root user
# #RUN apk --no-cache add curl ca-certificates \
#     #&& addgroup -S app && adduser -S -g app app
# RUN apk --no-cache add curl ca-certificates \
#     && addgroup -S app && adduser -S -g app app \
#     && addgroup app root    

ENV USER=app

# Copy of-watchdog binary
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog
# 
# # AE: unable to get the binary to start if moved to /home/app
# COPY --from=builder /home/rust/target/debug/function /usr/bin/
# 
WORKDIR /home/app

ENV fprocess="function"
EXPOSE 8080
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
