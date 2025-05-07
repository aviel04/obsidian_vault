# Cool Commands
### Popup Window
```bash
display-popup -w "#{window_id}" -h 5 -w 60 -x C -y C -d '#{pane_cwd}'
```
we can config it in the `~/.tmux.conf` config like so:
```bash
bind-key o display-popup -w "#{window_id}" -h 5 -w 60 -x C -y C -d '#{pane_cwd}'
```
---
# Leader Built-in Binds
`C-b` (Leader)

`C-b c`: Create a new window.
Description: Opens a new window within the current session.
`C-b n`: Next window.
Description: Switches to the next window in the current session.
`C-b p`: Previous window.
Description: Switches to the previous window in the current session.
`C-b "`: Split pane horizontally.
Description: Divides the current pane into two side-by-side panes.
`C-b %`: Split pane vertically.
Description: Divides the current pane into two panes stacked vertically.
`C-b s`: Choose session.
Description: Displays a list of available sessions to switch to.
`C-b w`: Choose window.
Description: Displays a list of windows in the current session to switch to.
`C-b [number]`: Select window by number.
Description: Switches directly to the window with the specified number.
`C-b <Space>`: Arrange panes.
Description: Cycles through different pane layouts.
`C-b x`: Kill Pane
Description: Closes the Current pane