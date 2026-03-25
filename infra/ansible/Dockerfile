FROM alpine:3.19

# On installe Ansible et le client Docker
RUN apk update && \
    apk add --no-cache \
    ansible \
    docker-cli \
    python3 \
    py3-pip \
    openssh-client

WORKDIR /ansible
CMD ["sleep", "infinity"]
