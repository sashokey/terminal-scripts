# Project maintenance

## Layout and build

- Keep each tool's sources and tests in its own top-level directory. Put installation instructions in the root `README.md`. Keep `README.md` and `AGENTS.md` as the only `.md` files.
- Store GitHub Actions workflows in `.github/workflows/`. Set the tool directory as the working directory of each check after checkout so relative paths resolve independently of neighboring tools. Install runner dependencies before checkout.
- Use the root GPL-3.0 `LICENSE` for project-owned code. Preserve required notices for third-party code, data and resources.
- The installers are standalone Bash scripts and run directly with Bash. They require no build or packaging step. Keep both raw download paths in `README.md` aligned with the entry points in `master`.
- See `README.md` for operating system, architecture, privileges, network, DNS and command prerequisites for each installer.

## Behavior to preserve

- Keep main-server setup on Netplan with `systemd-networkd`. Keep slave support for that backend and static IPv4 with ifupdown. Preserve IPv4 access for provisioning and service checks before SSH removal and reboot.
- Sourcing either installer must only define functions and initialize shell state. Keep provision and maintenance entry points intact, including `--update` on both roles and `--check` on the slave.
- Verify the upstream Xray release checksum and binary version. Validate the generated configuration, ACME path and HTTPS fallback. The slave must use its configured main server and report a successful chain check before setup can remove SSH.
- Preserve the working Xray release if a candidate rejects the configuration. Restore the previous release if the candidate fails its post-restart health check. Keep test assertions for these checks and update them alongside intentional behavior changes.
- Update this document when environment requirements, maintenance rules or important decisions change. Record the reason and how to verify the behavior. Use de-identified examples and omit dates and timestamps.

## Local and CI checks

From the repository root, enter `xray-install/` and run:

```bash
cd xray-install
bash -n setup.sh && bash -n setup-proxy.sh && bash -n tests/integration.sh
python3 -m unittest discover -s tests -v
```

Use Bash 5 and Python 3 for the full suite. Set `BASH_BIN` to select the Bash 5 interpreter when it differs from `bash` on `PATH`. Older Bash skips tests that require Bash 5. CI requires Bash 5 and runs all tests.

Run `bash tests/integration.sh` as root from the tool directory only in a disposable Debian 13 container. Use Bash 5, CA certificates, curl, Git, Nginx, OpenSSL, Python 3 and unzip. The test writes system configuration and uses local listeners. It needs outbound access to GitHub and `https://example.com/`.

The root Xray workflow runs both jobs in `debian:13-slim`. The `test` job checks Bash syntax and runs the behavior suite. The `integration` job downloads and verifies real Xray, checks the Nginx ACME path, and sends HTTPS through Xray with TLS certificate verification. A successful run reaches the public target through the chain, loses the route when either server stops, and recovers when main restarts while slave remains running. Use VMs for host networking, public ACME, SSH removal and reboot checks.

## Manual checks

- Download both installation URLs from `README.md` on a machine that has no checkout. Each download must be a Bash script identical to the corresponding file on `master`. Check each with `bash -n`.
- Provision a clean Debian 13 VM via the main installer using the VM console. Expect an IPv4 route, a valid public certificate, a working VLESS link and reachable HTTPS fallback. After success, expect SSH removal and a reboot.
- Provision a second clean VM with the slave installer. Check both supported network backends in separate runs. Expect the slave link to work through the main server and `bash setup-proxy.sh --check` on the slave console to report the verified chain.
- Repeat setup on a fresh VM with an unmet preflight requirement or a failing service check. Expect setup to stop with an error before SSH removal or reboot. Check the prior Xray binary and HTTPS response after a failed candidate update to confirm the working release is still in use.
- When adding a tool, run both tools' checks from their own directories. Confirm they resolve their own relative paths and keep any temporary tool out of the published changes.

## Releases

1. Publish the intended commit to `master`. Verify that all relevant CI jobs passed for that exact commit and that the documented installation URLs return the intended scripts.
2. Inspect existing GitHub releases and tags. Use the same `vN` name for the release title and Git tag. Start with `v1` and increment `N` by one for every release, including a fix or prerelease. Point the tag at the verified commit.
3. Publish notes covering the changes since the previous release. Confirm the published release and tag identify the tested commit.
