# herdr-keep

Run terminal apps (lazygit, btop, yazi, …) so [Herdr](https://herdr.dev)
starts them again in their panes after a Herdr restart or a reboot, the way
it resumes coding agents.

```bash
herdr-keep lazygit
```

Quit the app and it stays quit. Leave it running when Herdr goes down, and it
comes back in the same pane.

## Install

```bash
herdr plugin install tlpinney/herdr-keep
```

The plugin's startup hook links `herdr-keep` into `~/.local/bin` when Herdr
starts, because Herdr only resumes commands it finds on `PATH`. It never
replaces a file it did not create. To set it up before Herdr's next start, run
`bin/herdr-keep install` from a clone of this repository.

## Keep apps without typing herdr-keep

List the apps in the plugin's `apps` file, one command name per line (`#`
starts a comment), in `herdr plugin config-dir tlpinney.keep`:

```text
lazygit
btop
yazi
```

and turn them into aliases in your shell's startup file (`~/.zshrc`,
`~/.bashrc`):

```bash
command -v herdr-keep >/dev/null && eval "$(herdr-keep aliases)"
```

Now `lazygit` runs as `herdr-keep lazygit`.

## How it works

`herdr-keep CMD [ARGS...]` registers its pane with Herdr as a custom agent
(`pane report-agent --source custom:keep`) whose resume command is
`herdr-keep CMD [ARGS...]`, then runs the app. Herdr saves that command with
the session. After a restart, once a client attaches, Herdr runs it in the
restored pane, which registers again.

When the app exits on its own (you quit it, or Ctrl-C), `herdr-keep` releases
the pane (`pane release-agent`), so a quit app is not brought back. When Herdr
stops, the app and `herdr-keep` go down together before the release, and the
registration stays in the saved session.

Outside Herdr, `herdr-keep CMD` just runs `CMD`.

## Limits

- **The app starts fresh.** Herdr brings back the command, not the app's
  state. For lazygit or btop that is all there is; an editor needs its own
  session (`nvim -S`, or a session plugin) to reopen its files.
- **It shows as an agent.** The pane appears in Herdr's agent list as the
  app's name with the state label `kept`, and state `unknown`, so it never
  counts as working, blocked or done and never notifies.
- **Herdr's rules for resume commands.** The first word must be a command on
  `PATH` (hence the link in `~/.local/bin`), and no argument may contain an
  apostrophe or a control character. Such a command still runs, but is not
  kept.
- **One agent per pane.** Keeping an app in a pane takes over that pane's
  agent slot until the app exits.

## License

Apache-2.0.
