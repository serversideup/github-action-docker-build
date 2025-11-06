<p align="center">
		<img src="https://raw.githubusercontent.com/serversideup/github-action-docker-build/main/.github/readme-header.png" width="1280" alt="Header Image">
</p>
<p align="center">
	<a href="https://github.com/serversideup/github-action-docker-build/blob/main/LICENSE" target="_blank"><img src="https://badgen.net/github/license/serversideup/github-action-docker-build" alt="License"></a>
	<a href="https://github.com/sponsors/serversideup"><img src="https://badgen.net/badge/icon/Support%20Us?label=GitHub%20Sponsors&color=orange" alt="Support us"></a>
  <br />
  <a href="https://community.serversideup.net"><img alt="Discourse users" src="https://img.shields.io/discourse/users?color=blue&server=https%3A%2F%2Fcommunity.serversideup.net"></a>
  <a href="https://serversideup.net/discord"><img alt="Discord" src="https://img.shields.io/discord/910287105714954251?color=blueviolet"></a>
</p>

Hi! We're [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers). We're a two person team with a passion for open source products. We created [Server Side Up](https://serversideup.net) to help share what we learn.

### Find us at:

* 📖 [Blog](https://serversideup.net) - get the latest guides and free courses on all things web/mobile development.
* 🙋 [Community](https://community.serversideup.net) - get friendly help from our community members.
* 🤵‍♂️ [Get Professional Help](https://serversideup.net/get-help) - get guaranteed responses within next business day.
* 💻 [GitHub](https://github.com/serversideup) - check out our other open source projects
* 📫 [Newsletter](https://serversideup.net/subscribe) - skip the algorithms and get quality content right to your inbox
* 🐥 [Twitter](https://twitter.com/serversideup) - you can also follow [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers)
* ❤️ [Sponsor Us](https://github.com/sponsors/serversideup) - please consider sponsoring us so we can create more helpful resources

### Our Sponsors
All of our software is free an open to the world. None of this can be brought to you without the financial backing of our sponsors.

#### Individual Supporters
<!-- supporters --><a href="https://github.com/deligoez"><img src="https://github.com/deligoez.png" width="40px" alt="deligoez" /></a>&nbsp;&nbsp;<a href="https://github.com/alexjustesen"><img src="https://github.com/alexjustesen.png" width="40px" alt="alexjustesen" /></a>&nbsp;&nbsp;<!-- supporters -->

# About this project
This is a GitHub Action intended to simplify the process for building automated Docker images with GitHub Actions.

### Features:
- ✅ Stupid simple to use
- 🚀 Customize your docker image names/tags by easily passing what you want it to be
- 🤓 Multi-arch support
- 🔀 Context aware (great if you have a Docker file in a different part of your repo)
- 📦 **Multi-registry support** - Push to up to 3 registries simultaneously (Docker Hub, GitHub Container Registry, and private registries)

# Usage

## Single Registry Example
Here is a basic example workflow for publishing to a single registry:

```yml
name: Docker Publish (Production Images)
on:
  push:

jobs:
  docker-publish:
    runs-on: ubuntu-24.04
    steps:
      - name: docker-build-action
        uses: serversideup/github-action-docker-build@v6
        with:
          tags: serversideup/financial-freedom:latest
          registry-username: ${{ secrets.DOCKER_HUB_USERNAME }}
          registry-token: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
          platforms: "linux/amd64,linux/arm/v7,linux/arm64/v8"
```

## Multiple Registry Example
You can now push to up to **3 different registries** in a single build! Perfect for publishing to Docker Hub, GitHub Container Registry, and your own private registry simultaneously:

```yml
name: Docker Publish (Multiple Registries)
on:
  push:
    branches:
      - main

jobs:
  docker-publish:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Build and push to multiple registries
        uses: serversideup/github-action-docker-build@v6
        with:
          # Tag with multiple registry prefixes
          tags: |
            docker.io/myorg/myapp:latest
            ghcr.io/myorg/myapp:latest
            registry.example.com/myapp:latest
          
          # Registry 1: Docker Hub
          registry: "docker.io"
          registry-username: ${{ secrets.DOCKER_HUB_USERNAME }}
          registry-token: ${{ secrets.DOCKER_HUB_TOKEN }}
          
          # Registry 2: GitHub Container Registry
          registry-2: "ghcr.io"
          registry-username-2: ${{ github.actor }}
          registry-token-2: ${{ secrets.GITHUB_TOKEN }}
          
          # Registry 3: Custom Private Registry
          registry-3: "registry.example.com"
          registry-username-3: ${{ secrets.CUSTOM_REGISTRY_USER }}
          registry-token-3: ${{ secrets.CUSTOM_REGISTRY_TOKEN }}
          
          platforms: "linux/amd64,linux/arm64"
```

**💡 Pro tip:** You only need to specify the registries you want to use. Registry 2 and 3 are optional and will be skipped if credentials aren't provided.
### Configuration options
**🔀 Input Name**|**📚 Description**|**🛑 Required**|**👉 Default**
:-----:|:-----:|:-----:|:-----:
tags|Enter the tag(s) you would like to name your image with. (example: `myorg/myapp:production`) Use multi-line format for multiple tags.|⚠️ Yes| 
registry|Choose which container image repository to upload to. <a href="https://github.com/docker/login-action#usage">See all options.</a>| |`docker.io`
registry-username|Enter the username to authenticate with your first registry.|⚠️ Yes*| 
registry-token|Enter the token or password to authenticate with your first registry. (an access token is highly recommended)|⚠️ Yes*| 
registry-2|Second container registry (e.g., `ghcr.io`)| | 
registry-username-2|Username for second registry| | 
registry-token-2|Token/password for second registry| | 
registry-3|Third container registry (e.g., `registry.example.com`)| | 
registry-username-3|Username for third registry| | 
registry-token-3|Token/password for third registry| | 
context|The relative path to the Dockerfile.| |`.`
dockerfile|Filename of the Dockerfile within the context that you set.| |`./Dockerfile`
platforms|Comma separated list of <a href="https://github.com/docker-library/official-images#architectures-other-than-amd64">platforms</a>.| |`linux/amd64`
target|The target build stage to build.| |

> [!NOTE]  
> At least one registry's credentials must be provided (either registry 1, 2, or 3).

### Important security notice
Always use encrypted secrets when passing sensitive information. [Learn more here →](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

### Security Disclosures
If you find a security vulnerability, please let us know as soon as possible.

[View Our Responsible Disclosure Policy →](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8)