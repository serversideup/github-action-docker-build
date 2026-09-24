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

# About this project
This is a GitHub Action intended to simplify the process for building automated Docker images with GitHub Actions.

### Features:
- ✅ **Stupid simple to use** - just pass in the tags, registry, and credentials and you're good to go
- 🚀 **Customize your docker image names/tags** - easily pass in what you want it to be
- 🤓 **Multi-arch support** - build for multiple architectures
- 📦 **Multi-registry support** - build and push to up to 3 registries simultaneously (Docker Hub, GitHub Container Registry, and private registries)
- 🔀 **Context aware** - great if you have a Dockerfile in a different part of your repo
- ⚡ **Build cache support** - pass `cache-from`/`cache-to` straight through to reuse layers between runs

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
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Build and push Docker image
        uses: serversideup/github-action-docker-build@v6
        with:
          tags: serversideup/financial-freedom:latest
          registry-username: ${{ secrets.DOCKER_HUB_USERNAME }}
          registry-password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
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
          registry-password: ${{ secrets.DOCKER_HUB_TOKEN }}
          
          # Registry 2: GitHub Container Registry
          registry-2: "ghcr.io"
          registry-2-username: ${{ github.actor }}
          registry-2-password: ${{ secrets.GITHUB_TOKEN }}
          
          # Registry 3: Custom Private Registry
          registry-3: "registry.example.com"
          registry-3-username: ${{ secrets.CUSTOM_REGISTRY_USER }}
          registry-3-password: ${{ secrets.CUSTOM_REGISTRY_TOKEN }}
          
          platforms: "linux/amd64,linux/arm64"
```

**💡 Pro tip:** You only need to specify the registries you want to use. Registry 2 and 3 are optional and will be skipped if credentials aren't provided.
## Build Cache Example
GitHub-hosted runners give you a fresh VM on every job, so builds start cold unless you bring a cache with you. Pass `cache-from` and `cache-to` to reuse layers between runs.

The quickest option is GitHub's own Actions cache:

```yml
      - name: Build and push Docker image
        uses: serversideup/github-action-docker-build@v6
        with:
          tags: serversideup/financial-freedom:latest
          registry-username: ${{ secrets.DOCKER_HUB_USERNAME }}
          registry-password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

If you'd rather keep the cache next to your image, store it in your registry instead. This avoids the 10GB Actions cache limit and stays shared across branches:

```yml
name: Docker Publish (Registry Cache)
on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

jobs:
  docker-publish:
    runs-on: ubuntu-24.04
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build and push Docker image
        uses: serversideup/github-action-docker-build@v6
        with:
          tags: ghcr.io/myorg/myapp:latest
          registry: "ghcr.io"
          registry-username: ${{ github.actor }}
          registry-password: ${{ secrets.GITHUB_TOKEN }}
          cache-from: type=registry,ref=ghcr.io/myorg/myapp/buildcache:latest
          cache-to: type=registry,ref=ghcr.io/myorg/myapp/buildcache:latest,mode=max
```

**💡 Pro tip:** `mode=max` caches every layer, including intermediate build stages. Without it you only cache the layers that ship in your final image, which usually misses the expensive ones like `composer install`, `npm ci`, and compile steps.

### Configuration options
**🔀 Input Name**|**📚 Description**|**🛑 Required**|**👉 Default**
:-----:|:-----:|:-----:|:-----:
tags|Enter the tag(s) you would like to name your image with. (example: `myorg/myapp:production`) Use multi-line format for multiple tags.|⚠️ Yes| 
registry|Choose which container image repository to upload to. <a href="https://github.com/docker/login-action#usage">See all options.</a>| |`docker.io`
registry-username|Enter the username to authenticate with your first registry.|⚠️ Yes| 
registry-password|Enter the password or token to authenticate with your registry. (an access token is highly recommended)|⚠️ Yes|  
registry-token (deprecated)| Use `registry-password` instead||
context|The build context directory (the directory containing your Dockerfile and build files).| |`.`
dockerfile|Filename of the Dockerfile within the context that you set.| |`./Dockerfile`
platforms|Comma separated list of <a href="https://github.com/docker-library/official-images#architectures-other-than-amd64">platforms</a>.| |`linux/amd64`
target|The target build stage to build.| |
cache-from|List of external <a href="https://docs.docker.com/build/cache/backends/">cache sources</a> for the build (e.g., `type=gha`). Passed directly to `docker/build-push-action`.| |
cache-to|List of <a href="https://docs.docker.com/build/cache/backends/">cache export destinations</a> for the build (e.g., `type=gha,mode=max`). Passed directly to `docker/build-push-action`.| |

#### If you have more than one registry
**🔀 Input Name**|**📚 Description**|**🛑 Required**|**👉 Default**
:-----:|:-----:|:-----:|:-----:
registry-2|Choose which container image repository to upload to. <a href="https://github.com/docker/login-action#usage">See all options.</a>| |
registry-2-username|Enter the username to authenticate with your second registry.|⚠️ Yes (if you use the 2nd registry)| 
registry-2-password|Enter the token or password to authenticate with your second registry. (an access token is highly recommended)|⚠️ Yes (if you use the 2nd registry)|  
registry-3|Choose which container image repository to upload to. <a href="https://github.com/docker/login-action#usage">See all options.</a>| |
registry-3-username|Enter the username to authenticate with your third registry.|⚠️ Yes (if you use the 3rd registry)| 
registry-3-password|Enter the token or password to authenticate with your third registry. (an access token is highly recommended)|⚠️ Yes (if you use the 3rd registry)|  

> [!NOTE]  
> At least one registry's credentials must be provided (either registry 1, 2, or 3).

### Important security notice
Always use encrypted secrets when passing sensitive information. [Learn more here →](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

### Security Disclosures
If you find a security vulnerability, please let us know as soon as possible.

[View Our Responsible Disclosure Policy →](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8)

<!-- serversideup-sponsors -->
## Our Sponsors
All of our software is free and open to the world. None of this can be brought to you without the financial backing of our sponsors.

<p align="center"><a href="https://github.com/sponsors/serversideup"><img src="https://521public.s3.amazonaws.com/serversideup/sponsors/sponsor-box.png" alt="Become a sponsor"></a></p>

### Platinum Sponsors
<a href="https://sevalla.com"><img src="https://serversideup.net/sponsors/sevalla.png" alt="Sevalla" width="500px"></a>

### Silver Sponsors
<a href="https://giga-infosystems.com"><img src="https://serversideup.net/sponsors/giga-infosystems.png" alt="GiGa infosystems" width="200px"></a>

### Infrastructure Sponsors
These companies give us free access to the tools and infrastructure we use to build, test, and ship our open source projects. Their support helps our entire community.

<a href="https://depot.dev"><img src="https://serversideup.net/sponsors/depot.png" alt="Depot" width="250px"></a>&nbsp;&nbsp;<a href="https://hub.docker.com/u/serversideup"><img src="https://serversideup.net/sponsors/docker.png" alt="Docker" width="250px"></a>
<!-- serversideup-sponsors -->

<!-- serversideup-about -->
## About Us
We're [Dan](https://x.com/danpastori) and [Jay](https://x.com/jaydrogers) - a two-person team with a passion for open source products. We created [Server Side Up](https://serversideup.net) to help share what we learn.

<div align="center">

| <div align="center">Dan Pastori</div> | <div align="center">Jay Rogers</div> |
| --- | --- |
| <div align="center"><a href="https://x.com/danpastori"><img src="https://serversideup.net/wp-content/uploads/2023/08/dan.jpg" title="Dan Pastori" width="150px"></a><br /><a href="https://x.com/danpastori"><img src="https://serversideup.net/logos/x.svg" title="X" width="24px"></a><a href="https://github.com/danpastori"><img src="https://serversideup.net/logos/github.svg" title="GitHub" width="24px"></a></div> | <div align="center"><a href="https://x.com/jaydrogers"><img src="https://serversideup.net/wp-content/uploads/2023/08/jay.jpg" title="Jay Rogers" width="150px"></a><br /><a href="https://x.com/jaydrogers"><img src="https://serversideup.net/logos/x.svg" title="X" width="24px"></a><a href="https://github.com/jaydrogers"><img src="https://serversideup.net/logos/github.svg" title="GitHub" width="24px"></a></div> |

</div>

### Find us at:

* **📖 [Blog](https://serversideup.net)** - Get the latest guides and free courses on all things web/mobile development.
* **🙋 [Community](https://community.serversideup.net)** - Get friendly help from our community members.
* **🤵‍♂️ [Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing support from the core contributors.
* **💻 [GitHub](https://github.com/serversideup)** - Check out our other open source projects.
* **📫 [Newsletter](https://serversideup.net/subscribe)** - Skip the algorithms and get quality content right to your inbox.
* **🐥 [X (Twitter)](https://x.com/serversideup)** - You can also follow [Dan](https://x.com/danpastori) and [Jay](https://x.com/jaydrogers).
* **❤️ [Sponsor Us](https://github.com/sponsors/serversideup)** - Please consider sponsoring us so we can create more helpful resources.

## Our Products
If you appreciate this project, be sure to check out our other projects.

### 🛠️ Premium
- **[Self-Host Pro](https://selfhostpro.com)**: Sell self-hosted software in minutes.
- **[Bugflow](https://bugflow.io)**: Get product feedback directly in GitHub, GitLab, and more.
- **[Spin Pro](https://getspin.pro)**: Production-ready Docker templates for shipping quickly.

### 🌍 Open Source
- **[serversideup/php](https://serversideup.net/open-source/docker-php/)**: Supercharged PHP Docker images, based off the official PHP images. <!-- repo:serversideup/docker-php -->
- **[Spin](https://serversideup.net/open-source/spin/)**: Docker Simplified. Deploy Anywhere. Zero Downtime. Any OS. <!-- repo:serversideup/spin -->
- **[Financial Freedom](https://serversideup.net/open-source/financial-freedom/)**: Open source alternative to Mint, YNAB, and more. <!-- repo:serversideup/financial-freedom -->
- **[AmplitudeJS](https://serversideup.net/open-source/amplitudejs/)**: Customize the design of any element of the HTML5 Audio Player. <!-- repo:521dimensions/amplitudejs -->
- **[webext-bridge](https://serversideup.net/open-source/webext-bridge/)**: Messaging in Web Extensions made easy. Batteries included. <!-- repo:serversideup/webext-bridge -->
- **[serversideup/ansible](https://github.com/serversideup/docker-ansible)**: Run Ansible anywhere with a lightweight and powerful Docker image. <!-- repo:serversideup/docker-ansible -->

### 📚 Books
- **[Building Browser Extensions](https://serversideup.net/products/building-multi-platform-browser-extensions/)**: Build browser extensions for Firefox, Chrome, and more.
- **[Ultimate Guide To Building APIs & SPAs](https://serversideup.net/products/ultimate-guide-to-building-apis-and-spas-with-laravel-and-nuxt3/)**: Build web and mobile apps from the same codebase.
<!-- serversideup-about -->
