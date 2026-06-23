# Selkies Desktop

Selkies Desktop is an Open OnDemand Batch Connect app that launches a KDE
Plasma desktop on an HPC compute node and streams it through Selkies/PixelFlux.
It is designed for sites that want to distribute the desktop runtime as an
Apptainer image instead of installing a full graphical desktop stack directly on
every compute node.

## Features

- Uses the Open OnDemand `basic` Batch Connect template and the `/rnode/`
  reverse proxy.
- Starts Selkies from inside an Apptainer image.
- Starts a nested KDE Plasma Wayland session on the PixelFlux Wayland socket.
- Supports selectable CPU count, resolution, frame rate, encoder, H.264 CRF, and
  static-frame polish.
- Defaults to WebSocket transport, software CPU encoding, no audio, no
  microphone, no gamepad input, and no local manual-resolution lock.

## Requirements

- Open OnDemand with Batch Connect enabled.
- A Slurm-backed OOD cluster configuration.
- Apptainer available on the compute node.
- A Selkies-capable Apptainer image that provides:
  - `start-selkies` in `PATH`
  - an image runscript or command path that starts the nested KDE desktop
  - Selkies web assets, normally under `/opt/selkies-ood/web`
- Network policy that allows the OOD node proxy to reach the selected compute
  node port.

The companion image definition used during development is maintained separately
from this app repository.

## Installation

Install this repository as a system app:

```bash
cd /var/www/ood/apps/sys
git clone https://github.com/jose-d/bc_selkies_desktop.git
```

Then configure the app for your site by setting environment variables in the
OOD service environment, app wrapper, or another site policy mechanism.

Required site settings:

```bash
export OOD_SELKIES_CLUSTER="your_ood_cluster_name"
export OOD_SELKIES_DEFAULT_PARTITION="interactive"
export OOD_SELKIES_IMAGE="/path/to/rocky10-plasma-selfcontained.sif"
```

Optional settings:

```bash
export OOD_SELKIES_PARTITION_FIXED="false"
export OOD_SELKIES_NODELIST=""
export OOD_SELKIES_WEB_ROOT="/opt/selkies-ood/web"
export OOD_SELKIES_UI_TITLE="Selkies KDE Desktop"
export OOD_SELKIES_WAYLAND_LIB="/opt/selkies-ood/wayland/lib64"
```

Use `OOD_SELKIES_NODELIST` only for canary or test deployments that should pin
sessions to a specific compute node.

## Configuration

The main launch controls are in `form.yml.erb`:

- `bc_queue`
- `selkies_num_cores`
- `selkies_resolution`
- `selkies_framerate`
- `selkies_encoder`
- `selkies_h264_crf`
- `selkies_paint_over_quality`

The runtime contract is in `template/script.sh.erb`. The script starts Selkies,
waits for the PixelFlux Wayland socket, then starts the same Apptainer image
again for the nested KDE desktop.

## Testing

After deployment, submit a short session with two CPU cores and the default
resolution. The session log should show:

- `Starting Selkies PixelFlux server`
- `Waiting for PixelFlux Wayland socket`
- `Starting Rocky 10 KDE Plasma container`
- an active stream line such as `Mode: H264 (CPU)`

The OOD session card should open `/rnode/<host>/<port>/`.

## Troubleshooting

If the session fails before the Connect button appears, check that Slurm can
start the job and that the requested partition or optional nodelist is valid.

If the session log reports `Missing Apptainer image`, set
`OOD_SELKIES_IMAGE` to the path visible from the compute node.

If the browser opens but shows a black screen, inspect the session log for
PixelFlux, Wayland socket, and encoder messages. Start with a lower resolution
or `JPEG` encoder to separate desktop startup problems from H.264 encoder
problems.

If clipboard warnings mention `wl-paste`, install `wl-clipboard` in the image or
disable clipboard support in `template/script.sh.erb`.

## Known Limitations

- This app currently targets the Selkies WebSocket path, not WebRTC.
- Audio, microphone, gamepad input, collaboration, and file transfers are
  disabled by default.
- The shipped form has one KDE/Apptainer backend option. Additional backends can
  be added as site-specific extensions.
