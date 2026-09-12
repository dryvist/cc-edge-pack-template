# Changelog

## 1.0.0 (2026-09-12)


### Features

* add review-thread-resolver caller for instant bot-thread resolution ([#20](https://github.com/dryvist/cc-edge-pack-template/issues/20)) ([d64f316](https://github.com/dryvist/cc-edge-pack-template/commit/d64f316366615337cb5c514cc35b1b2c959b5c29))
* **fixtures:** replace toy passthrough sample with Claude/Gemini/Codex shapes ([#11](https://github.com/dryvist/cc-edge-pack-template/issues/11)) ([1cd752a](https://github.com/dryvist/cc-edge-pack-template/commit/1cd752adc3e1bde0e6cb6ada8233cbe3cf8d6bfb))
* full TypeScript pivot — Vitest + Biome + nix-devenv + release-please ([#4](https://github.com/dryvist/cc-edge-pack-template/issues/4)) ([b276140](https://github.com/dryvist/cc-edge-pack-template/commit/b276140468688698cb35f2807230ee57c9c6a4e6))
* initial Cribl pack template with full DRY scaffolding ([2cb9594](https://github.com/dryvist/cc-edge-pack-template/commit/2cb95948a9222c86ddd9b0b284ae8a38058b84ce))
* ship working demo passthrough pipeline + prove harness teeth ([#8](https://github.com/dryvist/cc-edge-pack-template/issues/8)) ([83b1b95](https://github.com/dryvist/cc-edge-pack-template/commit/83b1b951c1e624d94c33a03a4620d5313988d87b))
* validate end-to-end pack test flow ([fb2d8e6](https://github.com/dryvist/cc-edge-pack-template/commit/fb2d8e68b092c8265032e81f243ac43260f1b186))


### Bug Fixes

* **ci:** correct dcarbone/install-yq-action version (v1.3.1, not v1.4.4) ([#5](https://github.com/dryvist/cc-edge-pack-template/issues/5)) ([b4c19bd](https://github.com/dryvist/cc-edge-pack-template/commit/b4c19bd24d133add4370f2e8742f29e370384852))
* **ci:** pass .yamllint.yml as config_file to frenck/action-yamllint ([#6](https://github.com/dryvist/cc-edge-pack-template/issues/6)) ([598c5e6](https://github.com/dryvist/cc-edge-pack-template/commit/598c5e61a3f2e9bc33e263c65c3f4bb9ee7f0940))
* **ci:** retarget reusable-workflow uses: refs to current org homes ([#14](https://github.com/dryvist/cc-edge-pack-template/issues/14)) ([7e3d01f](https://github.com/dryvist/cc-edge-pack-template/commit/7e3d01ff4a43abf9449482aefdae1e0bdbb49fe5))
* **ci:** trigger tests on scripts/ and reusable-workflow changes; sync workflow docs ([#21](https://github.com/dryvist/cc-edge-pack-template/issues/21)) ([f7e55ba](https://github.com/dryvist/cc-edge-pack-template/commit/f7e55baa5a91fa9dfddaa787448f38f9b517c830))
* **release-please:** inherit dryvist/.github org-native caller ([#17](https://github.com/dryvist/cc-edge-pack-template/issues/17)) ([96075e9](https://github.com/dryvist/cc-edge-pack-template/commit/96075e9037f28b2fda75ed73fe7fee4f8de5b263))
* **tarball:** top-level whitelist for createPackTarball ([#7](https://github.com/dryvist/cc-edge-pack-template/issues/7)) ([452f029](https://github.com/dryvist/cc-edge-pack-template/commit/452f029b1735ba963ef55c27838222762d779637))
* **tests:** allow esbuild and lefthook build scripts for pnpm v11 ([#35](https://github.com/dryvist/cc-edge-pack-template/issues/35)) ([9103a83](https://github.com/dryvist/cc-edge-pack-template/commit/9103a8352714a7a4978ab350b190dc7914755b40))
* **tests:** wait for pack to appear in /packs after install ([70a2e0b](https://github.com/dryvist/cc-edge-pack-template/commit/70a2e0b0ceb097e99204dcf15830714207680362))
