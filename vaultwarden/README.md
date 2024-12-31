# How to package k9s

This is a simple example of how to package k9s using melange and apko.

## Steps

1. Run the first pipeline to build the package

```bash
docker run --privileged --rm -v "${PWD}":/work --entrypoint=melange --workdir=/work cgr.dev/chainguard/melange build myvault.yaml --arch x86_64 --signing-key melange.rsa --keyring-append melange.rsa.pub --keyring-append https://packages.wolfi.dev/os/wolfi-signing.rsa.pub --repository-append https://packages.wolfi.dev/os --repository-append /work/packages --empty-workspace
```

2. Run the second pipeline to build the apko

```bash
docker run --rm --workdir /work -v ${PWD}:/work cgr.dev/chainguard/apko build apko.yaml myvault:test myvault.tar --arch x86_64 --keyring-append melange.rsa.pub --repository-append /work/packages
````

3. Load the image into docker

```bash
docker load -i myvault.tar
```

4. Run the image

```bash
docker run --detach --name vaultwarden \
  --env DOMAIN="https://vw.domain.tld" \
  --volume $(pwd)/vw-data/:/data/ \
  --restart unless-stopped \
  --publish 80:80 \
  --env WEB_VAULT_ENABLED=false \
  myvault:test-amd64
```