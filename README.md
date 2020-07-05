# Description

Portainer images which manifests contain the correct OS and architecture to be used directly in Docker Swarm stacks, including on Raspberry Pi without the need to use a special tag and to use the option to ignore the platform.

# Usage

For example in a `docker-compose.yml`:

```yaml
version: '3.7'

services:
  portainer:
    image: vitkin/portainer:1.24.0
    volumes:
      - type: bind
        source: /var/run/docker.sock
        target: /var/run/docker.sock
      - type: volume
        source: portainer-data
        target: /data
    command:
      - "-H"
      - "unix:///var/run/docker.sock"
    ports:
      - published: 9000
        target: 9000
        protocol: tcp
        mode: host
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - node.role==manager

volumes:
  portainer-data:
```

# Build process

1. Enable the experimental feature for `docker-cli` by adding `"experimental": "enabled"` to `~/.docker/config.json`.

2. Run the following commands:

  ```bash
  version=1.24.0
  repo=vitkin/portainer

  function manifest_create {
    eval "docker manifest create ${repo}:${version} portainer/portainer:{$1}-${version}"
  }

  function manifest_annotate {
    docker manifest annotate --os $1 --arch $2 ${repo}:${version} portainer/portainer:$1-$2-${version}
  }

  function manifest_push {
    docker manifest push -p ${repo}:${version}
  }  

  manifest_create linux-amd64,linux-arm,linux-arm64,linux-ppc64le,linux-s390x,windows-amd64

  for arch in amd64 arm arm64 ppc64le s390x; do
    manifest_annotate linux ${arch}
  done

  manifest_annotate windows amd64

  manifest_push

  version=${version}-alpine

  manifest_create linux-amd64,linux-arm,linux-arm64

  for arch in amd64 arm arm64; do
    manifest_annotate linux ${arch}
  done

  manifest_push
  ```
