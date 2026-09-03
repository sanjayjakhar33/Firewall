# Execution Environment

AWX must run an execution environment containing the collection. This repository provides `execution-environment.yml`, which installs the pinned `requirements.yml` collection into the AWX EE base image.

On a build host with `ansible-builder`:

```bash
python3 -m pip install ansible-builder
ansible-builder build --file execution-environment.yml --tag fortigate-awx-ee:2.6.0
```

Push the image to a private registry reachable by AWX, then select it on each FortiGate job template. If your AWX installation supports building EEs through its project mechanism, keep `requirements.yml` as the source of truth. Record the final image digest with the tested FortiOS and AWX versions.
