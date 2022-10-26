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
<!-- supporters --><a href="https://github.com/deligoez"><img src="https://github.com/deligoez.png" width="40px" alt="deligoez" /></a>&nbsp;&nbsp;<!-- supporters -->

# About this project
This is a GitHub Action intended to simplify the process for building automated Docker images with GitHub Actions.

### Features:
- ✅ Stupid simple to use
- 🚀 Customize your docker image names/tags by easily passing the action what you want it to be
- 🤓 Multi-arch support
- 🔀 Context aware (great if you have a Docker file in a different part of your repo)

# Usage
Here is an example workflow:

```yml
name: Docker Publish (Production Images)
on:
  push:

jobs:
  docker-publish:
    runs-on: ubuntu-22.04
    steps:
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: docker-build-action
        uses: serversideup/github-action-docker-build@v1.0.0
        with:
          tags: serversideup/financial-freedom:latest
          platforms: "linux/amd64,linux/arm/v7,linux/arm64/v8"
          context: "./docker/"
```
### Configuration options
**🔀 Variable Name**|**📚 Description**|**👉 Default**
:-----:|:-----:|:-----:
tags|Enter the tag you would like to name your image with. (example: `myorg/myapp:production`)|**🚨 Required to be set**
context|The relative path to the Dockerfile.|`.`
platforms|Comma separated list of <a href="https://github.com/docker-library/official-images#architectures-other-than-amd64">platforms</a>.|`linux/amd64`

### Security Disclosures
If you find a security vulnerability, please let us know as soon as possible.

[View Our Responsible Disclosure Policy →](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8)