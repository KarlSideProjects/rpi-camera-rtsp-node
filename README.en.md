<div align="center">
  <img src="./docs/assets/rpi-camera-rtsp-node.svg" width="112" alt="rpi-camera-rtsp-node icon">
  <h1>rpi-camera-rtsp-node</h1>
  <p><strong>Turn a Raspberry Pi CSI camera into a controllable, supply-chain-conscious RTSP node.</strong></p>
  <p>
    <a href="https://github.com/KarlSideProjects/rpi-camera-rtsp-node/releases/latest"><img alt="Latest release" src="https://img.shields.io/github/v/release/KarlSideProjects/rpi-camera-rtsp-node?style=flat-square"></a>
    <img alt="Raspberry Pi" src="https://img.shields.io/badge/Raspberry%20Pi-Zero%202%20W-c51a4a?style=flat-square&logo=raspberrypi&logoColor=white">
    <img alt="Streams" src="https://img.shields.io/badge/RTSP%20%2F%20WebRTC%20%2F%20HLS-H.264-2f855a?style=flat-square">
    <a href="./LICENSE"><img alt="License" src="https://img.shields.io/badge/license-non--commercial-blue?style=flat-square"></a>
  </p>
  <p><a href="./README.md">繁體中文</a> | <strong>English</strong></p>
</div>

**An authenticated Raspberry Pi CSI camera stream for existing NVRs, browsers and computer-vision hosts.** The Pi captures and hardware-encodes H.264; downstream tools handle recording and analysis.

This public repository contains documentation and binary releases, **not Node Agent source code**. Install on a supported Pi with a CSI camera to view RTSP, WebRTC or HLS. Without hardware, start with the [camera diagnosis case](./docs/question-and-answer.md); there is no hosted interactive demo.

This is not a complete commercial IP camera. It does not provide recording, inference, central management, a warranty or a country-of-origin guarantee. Compatibility with individual downstream products needs deployment-specific testing.

[Get started](#get-started) • [Use cases](#use-cases) • [Architecture](#architecture)
• [Supply chain](#supply-chain-boundary) • [Commercial licensing](#commercial-licensing)
• [Q&A](#qa-and-troubleshooting)

## What you get

- **Hardware H.264 source**: the Pi only captures, hardware-encodes, and serves.
- **Multi-protocol output**: RTSP, WebRTC, and HLS from one stream path.
- **Low-resource node**: designed for Raspberry Pi Zero 2 W, with a constrained Pi Zero profile.
- **Consumer-side AI/NVR**: inference, recording, alerting, fan-out, and central management stay off-node.
- **Supply-chain-conscious deployment**: works with integrator-led verification through distributors such as RS, DigiKey, and Mouser.

## Get started

Run this on the Raspberry Pi:

```bash
curl -fsSL https://github.com/KarlSideProjects/rpi-camera-rtsp-node/releases/latest/download/install.sh | \
  bash -s -- \
    --read-username viewer \
    --read-password '<your-password>'
```

The installer deploys the Node binary, bundled MediaMTX, config, and `systemd`
service. You choose the stream password; the installer does not generate or
print it later.

Default stream path: `cam`.

```text
RTSP:   rtsp://<user>:<pass>@<pi-host>.local:8554/cam
WebRTC: http://<pi-host>.local:8889/cam/
HLS:    http://<pi-host>.local:8888/cam/index.m3u8
```

Test RTSP from another computer:

```bash
ffplay "rtsp://viewer:<read-password>@<node-host>:8554/cam"
```

## Use cases

| Use case | Value |
| --- | --- |
| SI / low-voltage / security integration | A controllable RTSP source for NVR and AI systems |
| Commercial and public-sector evaluation | Documented installation and integrator-owned hardware verification |
| AI/NVR pipelines | A standard RTSP entry point for downstream integration |
| Household and non-profit use | Low-cost, self-hosted, reinstallable Raspberry Pi camera node |

This project is not a fit when you need a full warranty, PoE enclosure, cloud
management, vendor SLA, or an already-delivered central fleet platform.
Fleet Layer / central management is roadmap, not a current capability.

## Architecture

MediaMTX owns the media plane. The Node Agent only renders config, manages the
service, advertises mDNS, and checks camera status. It does not read, transcode,
or process video frames.

```text
CSI camera
   |
   v
Raspberry Pi Node
   |  hardware H.264 encode
   v
RTSP / WebRTC / HLS
   |
   +--> go2rtc / NVR / Frigate
   +--> YOLO / AI-CV host
   +--> internal dashboard
```

go2rtc example:

```yaml
streams:
  rpi_camera:
    - rtsp://viewer:<read-password>@<node-host>:8554/cam
```

## Supply-chain boundary

This project can support deployments where the integrator buys hardware through
standard distributors such as RS, DigiKey, Mouser, or similar channels, then
verifies fixed part numbers, country-of-origin evidence, receiving labels, and
BOM records.

> [!NOTE]
> Hardware origin, part numbers, receiving labels, procurement evidence, and BOM
> compliance are the integrator's responsibility. This project does not provide
> public-procurement, security-compliance, or country-of-origin certification.

Read the checklist: [Supply-chain self-verification checklist](./docs/supply-chain-verification.zh-TW.md).

## Hardware profiles

| Profile | Hardware | Stream profile | Purpose |
| --- | --- | --- | --- |
| Reference | Raspberry Pi Zero 2 W, 64-bit OS | H.264 1280x720 around 30 fps | Release acceptance target |
| Constrained | Original Pi Zero / Zero W, ARMv6 | H.264 640x480 around 15 fps | Low-resource supported profile |
| Integration proxy | Pi 3B | Same verifier flow, non-reference | Day-to-day integration testing |

The OS must see the CSI camera through `rpicam-hello --list-cameras` or
`libcamera-hello --list-cameras`.

## Commercial licensing

Personal, household, non-profit educational, and other non-commercial use may be
free under the public license terms. Commercial deployment, paid procurement or system-integration projects,
managed-service delivery, and enterprise internal operations require a separate
written commercial license. Noncommercial research and teaching are welcome
under the applicable version of the LICENSE.

- Non-sensitive general questions:
  [Commercial inquiry](https://github.com/KarlSideProjects/rpi-camera-rtsp-node/issues/new?template=commercial-license.yml)
- Pricing, procurement, NDA, site topology, customer information, or deployment
  details: <jhihweijhan@gmail.com>

> [!WARNING]
> Do not post credentials, RTSP URLs, private network topology, camera locations,
> customer names, procurement-sensitive information, or NDA-covered information
> in public GitHub issues.

## Q&A and troubleshooting

- [Question and Answer](./docs/question-and-answer.md)
- [繁體中文 Q&A](./docs/question-and-answer.zh-TW.md)

Common diagnostics:

```bash
rpicam-hello --list-cameras
sudo systemctl status rpi-camera-mediamtx.service
```

## Evidence and learning extensions

As checked on 2026-09-16, [v0.1.17](https://github.com/KarlSideProjects/rpi-camera-rtsp-node/releases/tag/v0.1.17) provides the installer, ARM64/ARMHF/ARMv6/ARMv6 legacy bundles and a release manifest. Asset availability does not establish fresh hardware or performance acceptance; this documentation update did not repeat live Pi tests.

The Q&A documents a Pi 3B/OV5647 investigation that separated power, device-tree activation and sensor response. Its unresolved hardware hypotheses are not proof of a repaired camera. This is a useful basis for controlled diagnostic exercises; no teaching-outcome study is reported.

The custom [noncommercial research LICENSE](./LICENSE) permits noncommercial research, modification, and redistribution of covered material. It does not promise private source access. Existing binary releases retain their accompanying terms; see [scope and version notices](./LICENSING.md). Bundled MediaMTX retains its independent MIT license; this does not make the whole product open source. See the [Traditional Chinese overview](./README.md) for the integrated project guide.
