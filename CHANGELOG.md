# Changelog

## 0.1.2 - 2026-06-24

- Run the rendered Batch Connect script with `bash` from `submit.yml.erb` so
  launch does not depend on the staged `script.sh` executable bit.

## 0.1.1 - 2026-06-23

- Use portable ERB control tags in `submit.yml.erb` for compatibility with the
  standard Ruby ERB parser used on Rocky 10.
- Clarify that Open OnDemand 4.x deployments can set app environment variables
  through `pun_custom_env` in `nginx_stage.yml`.

## 0.1.0 - 2026-06-23

- Initial published Batch Connect app layout.
- Launch Selkies/PixelFlux and KDE Plasma from an Apptainer image.
- Add configurable cluster, partition, optional nodelist, image path, and web
  root settings.
