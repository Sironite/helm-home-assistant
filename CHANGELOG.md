# Changelog

## [1.1.1](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.1.0...home-assistant-v1.1.1) (2026-06-22)


### Bug Fixes

* **ci:** add workflow_dispatch to trigger initial chart publish ([0146488](https://github.com/Sironite/helm-home-assistant/commit/0146488c3d383336dd7c0a912fd4ff927503e4fa))
* **ci:** match release-please helm tag format (home-assistant-v*) ([6976f61](https://github.com/Sironite/helm-home-assistant/commit/6976f619ae423d48b33401985125a1d477811fc6))

## [1.1.0](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.0.0...home-assistant-v1.1.0) (2026-06-22)


### Features

* initial release of home-assistant Helm chart ([b9c9e1a](https://github.com/Sironite/helm-home-assistant/commit/b9c9e1ae2d49c3e00ae2c210492d6d82cc8fbc9f))


### Bug Fixes

* **ci:** remove gitleaks (org license required), add --verify=false to helm-unittest install ([18988b9](https://github.com/Sironite/helm-home-assistant/commit/18988b96c777d858875024e76424284e3c84e0f9))
* **ci:** suppress KSV014 Trivy finding — LSIO s6-overlay incompatible with readOnlyRootFilesystem ([6411808](https://github.com/Sironite/helm-home-assistant/commit/641180880e5315f9373688fc3645daebf64e0f53))
