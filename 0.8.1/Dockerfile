FROM arm32v6/alpine:3.6
MAINTAINER Jeff Mitchell <jeff@hashicorp.com> (@jefferai)

COPY ./qemu-arm-static /usr/bin/qemu-arm-static

# This is the release of Vault to pull in.
ENV VAULT_VERSION=0.8.1

ENV GOSU_VERSION="1.7"
ENV	GOSU_ARCHITECTURE="armhf"
ENV GOSU_DOWNLOAD_URL="https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$GOSU_ARCHITECTURE"
ENV	GOSU_DOWNLOAD_SIG="https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$GOSU_ARCHITECTURE.asc"
ENV	GOSU_DOWNLOAD_KEY="0x036A9C25BF357DD4"

RUN ["/usr/bin/qemu-arm-static", "/bin/sh", "-c", " \
buildDeps='curl gnupg' HOME='/root' && \
set -x && \
apk add --update $buildDeps && \
gpg-agent --daemon && \
gpg --keyserver pgp.mit.edu --recv-keys $GOSU_DOWNLOAD_KEY && \
echo \"trusted-key $GOSU_DOWNLOAD_KEY\" >> /root/.gnupg/gpg.conf && \
curl -sSL \"$GOSU_DOWNLOAD_URL\" > gosu-$GOSU_ARCHITECTURE && \
curl -sSL \"$GOSU_DOWNLOAD_SIG\" > gosu-$GOSU_ARCHITECTURE.asc && \
gpg --verify gosu-$GOSU_ARCHITECTURE.asc && \
rm -f gosu-$GOSU_ARCHITECTURE.asc && \
mv gosu-$GOSU_ARCHITECTURE /usr/bin/gosu && \
chmod +x /usr/bin/gosu && \
apk del --purge $buildDeps && \
rm -rf /root/.gnupg && \
rm -rf /var/cache/apk/*"]

# Create a vault user and group first so the IDs get set the same way,
# even as the rest of this may change over time.
RUN ["/usr/bin/qemu-arm-static", "/bin/sh", "-c", "\
    addgroup vault && \
    adduser -S -G vault vault"]

# Set up certificates, our base tools, and Vault.
RUN ["/usr/bin/qemu-arm-static", "/bin/sh", "-c", "\
    apk add --no-cache ca-certificates gnupg openssl libcap dumb-init && \
    gpg --keyserver pgp.mit.edu --recv-keys 91A6E7F85D05C65630BEF18951852D87348FFC4C && \
    mkdir -p /tmp/build && \
    cd /tmp/build && \
    wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_arm.zip && \
    wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_SHA256SUMS && \
    wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_SHA256SUMS.sig && \
    gpg --batch --verify vault_${VAULT_VERSION}_SHA256SUMS.sig vault_${VAULT_VERSION}_SHA256SUMS && \
    grep vault_${VAULT_VERSION}_linux_arm.zip vault_${VAULT_VERSION}_SHA256SUMS | sha256sum -c && \
    unzip -d /bin vault_${VAULT_VERSION}_linux_arm.zip && \
    cd /tmp && \
    rm -rf /tmp/build && \
    apk del gnupg openssl && \
    rm -rf /root/.gnupg"]

# /vault/logs is made available to use as a location to store audit logs, if
# desired; /vault/file is made available to use as a location with the file
# storage backend, if desired; the server will be started with /vault/config as
# the configuration directory so you can add additional config files in that
# location.
RUN ["/usr/bin/qemu-arm-static", "/bin/sh", "-c", "\
    mkdir -p /vault/logs && \
    mkdir -p /vault/file && \
    mkdir -p /vault/config && \
    chown -R vault:vault /vault && \
    cd /bin && \
    ln -s /usr/bin/gosu && \
    ln -s /usr/bin/dumb-init"]

# Expose the logs directory as a volume since there's potentially long-running
# state in there
VOLUME /vault/logs

# Expose the file directory as a volume since there's potentially long-running
# state in there
VOLUME /vault/file

# 8200/tcp is the primary interface that applications use to interact with
# Vault.
EXPOSE 8200

# The entry point script uses dumb-init as the top-level process to reap any
# zombie processes created by Vault sub-processes.
#
# For production derivatives of this container, you shoud add the IPC_LOCK
# capability so that Vault can mlock memory.
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]

# By default you'll get a single-node development server that stores everything
# in RAM and bootstraps itself. Don't use this configuration for production.
CMD ["server", "-dev"]
