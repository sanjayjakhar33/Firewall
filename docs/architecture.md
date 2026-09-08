# Architecture

The lab uses a bridged VMware network: the FortiGate VM and Ubuntu VM receive addresses on a reachable management LAN. AWX runs on Ubuntu and launches Ansible jobs using `ansible.builtin.uri` against the FortiOS HTTPS REST API.

Keep the FortiGate administrative interface reachable only from the Ubuntu VM or a dedicated lab management subnet. Bridging is not isolation; the upstream LAN can see both guests.

The repository separates inventory, reusable roles, playbook entry points, and AWX configuration documentation so future UAT/production inventories can be added without changing the automation logic.
