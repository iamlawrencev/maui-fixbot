# MAUI fix bot — notes from earlier runs

## Build and test commands that work

- (2026-11-24) NONE yet: `./build.sh --target=dotnet` fails because the environment's network
  policy blocks the SDK hosts (`builds.dotnet.microsoft.com`, `dotnetcli.azureedge.net`,
  `pkgs.dev.azure.com` dnceng feeds → proxy 403 "no rule or network setting allows this host").
  `api.nuget.org` is reachable. No `dotnet` on PATH. Until fixed, no fix can be verified → no PR.
- net11 `global.json` wants SDK 11.0.101-servicing.26567.105 in `./.dotnet`.

## GitHub access in the cloud environment

- `gh auth status`: GH_TOKEN is set but reported invalid; `gh api` works only for `search/*`
  (search/issues returns issue bodies; secondary rate limit trips after ~3-4 searches/min — space them out).
- `gh api repos/dotnet/maui/...` (issues, comments, timeline) → 403 "not enabled for this session".
- `add_repo dotnet/maui` (read) → git clone/fetch only. `add_repo` with push is refused because
  the fork `iamlawrencev/maui` is attached under the same name `maui` → no API/MCP access to
  dotnet/maui, so the bot CANNOT open the upstream PR (best case is `pushed-no-pr` + compare URL).
- WebFetch of `https://github.com/dotnet/maui/issues/<n>` works for reading issue threads and linked PRs.
- `git fetch upstream net11 main --depth=200` works.

## Picking tips

- Before deep-diving, `git log upstream/net11 --grep=<issue>` — several verified issues are already
  fixed on net11 (e.g. #40375 via #40376, #39747 via #39838) but still open.
- Search query that gives a useful shortlist: `is:issue is:open no:assignee label:s/verified -linked:pr`
  minus `platform/*` labels. Few shared-code candidates remain; most open bugs are platform/device.

## Lessons from maintainer feedback

(none yet)

## Needs human

- Sign the .NET Foundation CLA (https://cla.dotnetfoundation.org/) when the CLA bot asks on the first PR.
- Allow .NET SDK download hosts in the cloud environment network policy (builds.dotnet.microsoft.com,
  dotnetcli.azureedge.net / dotnetcli.blob.core.windows.net, pkgs.dev.azure.com) — without it nothing can be built or tested.
- Give the bot a way to open PRs on dotnet/maui: a valid GH_TOKEN with public_repo scope for the fork owner,
  or run the routine in a session whose source is dotnet/maui (fork and upstream can't both be attached: same repo name).
