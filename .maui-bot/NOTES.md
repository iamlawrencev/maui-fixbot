# MAUI fix bot — notes from earlier runs

## Build and test commands that work

- (2026-11-24) NONE yet: `./build.sh --target=dotnet` fails because the environment's network
  policy blocks the SDK hosts (`builds.dotnet.microsoft.com`, `dotnetcli.azureedge.net`,
  `pkgs.dev.azure.com` dnceng feeds → proxy 403 "no rule or network setting allows this host").
  `api.nuget.org` is reachable. No `dotnet` on PATH. Until fixed, no fix can be verified → no PR.
- (2026-11-24, run 2) SDK hosts now reachable: `dotnet-install.sh --version 11.0.101-servicing.26567.105`
  (from builds.dotnet.microsoft.com) installs fine; pkgs.dev.azure.com dnceng feed returns 200.
  Not yet tried: `./build.sh --target=dotnet` and an actual test-project build.
- net11 `global.json` wants SDK 11.0.101-servicing.26567.105 in `./.dotnet`.

## GitHub access in the cloud environment

- `gh auth status`: GH_TOKEN is set but reported invalid; `gh api` works only for `search/*`
  (search/issues returns issue bodies; secondary rate limit trips after ~3-4 searches/min — space them out).
- `gh api repos/dotnet/maui/...` (issues, comments, timeline) → 403 "not enabled for this session".
- `add_repo dotnet/maui` (read) → git clone/fetch only. `add_repo` with push is refused because
  the fork `iamlawrencev/maui` is attached under the same name `maui` → no API/MCP access to
  dotnet/maui, so the bot CANNOT open the upstream PR (best case is `pushed-no-pr` + compare URL).
- WebFetch of `https://github.com/dotnet/maui/issues/<n>` works for reading issue threads and linked PRs.
- `git fetch upstream net11 main --depth=200` worked in run 1. In run 2 (2026-11-24, after the
  "[Forward fixes] net11 to main (#40577)" merge) upstream had NO literal `net11` branch
  (`git ls-remote upstream refs/heads/net11` empty), only `net11.0`, `candidate/11.0.1xx-sr*` and
  `inflight/11.0.1xx-sr2..4`. AGENTS.md forbids guessing/using those as a .NET 11 fix base → run stopped.
  Always check `git ls-remote upstream refs/heads/net11` first.
- Fork renamed to `iamlawrencev/maui-fixbot`, but `add_repo dotnet/maui access:push` still refused:
  "you need push access to dotnet/maui" → still no way to open the upstream PR (pushed-no-pr only).

## Picking tips

- Before deep-diving, `git log upstream/net11 --grep=<issue>` — several verified issues are already
  fixed on net11 (e.g. #40375 via #40376, #39747 via #39838) but still open.
- Search query that gives a useful shortlist: `is:issue is:open no:assignee label:s/verified -linked:pr`
  minus `platform/*` labels. Few shared-code candidates remain; most open bugs are platform/device.

## Lessons from maintainer feedback

(none yet)

## Needs human

- (2026-11-24) Upstream `dotnet/maui` has no literal `net11` branch right now, which AGENTS.md requires as the
  review base for .NET 11 fixes. Confirm with maintainers which base to use (or wait for `net11` to reappear);
  until then the bot stops before fixing anything.

- Sign the .NET Foundation CLA (https://cla.dotnetfoundation.org/) when the CLA bot asks on the first PR.
- ~~Allow .NET SDK download hosts~~ — resolved as of 2026-11-24 run 2 (SDK installs).
- Give the bot a way to open PRs on dotnet/maui: a valid GH_TOKEN with public_repo scope for the fork owner.
  Renaming the fork did not help: add_repo(dotnet/maui, push) is refused because the account lacks push access to dotnet/maui.
