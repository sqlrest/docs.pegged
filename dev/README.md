# pegged — contributor notes

- **Thin-shell rule**: all lifecycle behavior belongs in [go-pgdocker](https://github.com/gomatic/go-pgdocker). pegged owns only flag/env binding, port resolution, and output. A feature that needs daemon logic goes in the library first.
- Layout mirrors `gomatic/template.cli`: `internal/app/commands/<cmd>` (urfave/cli wiring) 1:1 with `internal/domain/<cmd>` (`Run(ctx, logger, cfg, args...)`), shared plumbing in `internal/manage` (manager construction seam, port resolution, value validation), sentinels in `internal/constants`.
- Tests run entirely against per-package `stubEngine` fakes implementing `pgdocker.Engine`; `internal/manage` is tested through `DOCKER_HOST` pointed at an httptest daemon. No test touches a real daemon.
- Gate: `make check` / `make ci` — stickler/yze strict, gocognit ≤ 7, exactly 100.0% coverage. Zero prior-employer trace: before every push, confirm the tree and history carry no prior-employer identifiers (the identifier set lives in the private home policy, not here).
