# Changelog

## [2.0.1](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v2.0.0...home-assistant-v2.0.1) (2026-06-23)


### Bug Fixes

* update ArtifactHub README with 2.0.0 docs ([3e0bc42](https://github.com/Sironite/helm-home-assistant/commit/3e0bc422fc545fef5a2019128163f1e9e4fcaa1d))

## [2.0.0](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.2.1...home-assistant-v2.0.0) (2026-06-23)


### ⚠ BREAKING CHANGES

* networkPolicy.enabled is removed. Replace with

### Features

* add Ingress, Kubernetes NetworkPolicy, and custom policy rules ([8556d03](https://github.com/Sironite/helm-home-assistant/commit/8556d0331c8fc1f05c475446fa5458860917f940))

## [1.2.1](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.2.0...home-assistant-v1.2.1) (2026-06-23)


### Bug Fixes

* export secret keyring for helm package --sign ([87718f6](https://github.com/Sironite/helm-home-assistant/commit/87718f627e4ebc818b4dab4abe420f953b100545))
* use legacy OpenPGP keyring format for helm package --sign ([ac8d42c](https://github.com/Sironite/helm-home-assistant/commit/ac8d42c3e213044d6d0ee6c4e78e5ca6b64474d5))

## [1.2.0](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.1.4...home-assistant-v1.2.0) (2026-06-22)


### Features

* add values schema and GPG chart signing ([fed8a4d](https://github.com/Sironite/helm-home-assistant/commit/fed8a4d9f35709e68d00e6ed45c1146324a9c0d7))

## [1.1.4](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.1.3...home-assistant-v1.1.4) (2026-06-22)


### Bug Fixes

* add ArtifactHub icon ([193949e](https://github.com/Sironite/helm-home-assistant/commit/193949ef41b1718f514b21fbe61a0b0320aba35c))

## [1.1.3](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.1.2...home-assistant-v1.1.3) (2026-06-22)


### Bug Fixes

* use valid ArtifactHub category slug (integration-delivery) ([86e76e9](https://github.com/Sironite/helm-home-assistant/commit/86e76e98302c599551cc6c289001fee662f147d2))

## [1.1.2](https://github.com/Sironite/helm-home-assistant/compare/home-assistant-v1.1.1...home-assistant-v1.1.2) (2026-06-22)


### Bug Fixes

* sync authentik proxy tag in values.yaml.example to 2026.5.3 ([167fe56](https://github.com/Sironite/helm-home-assistant/commit/167fe568cf731cd57da9f9404926912a4bd64e7f))

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
