# Debug your [GitHub Actions](https://github.com/features/actions) With SSH

This GitHub Action enables direct interaction with the host system running your GitHub Actions via SSH, utilizing [upterm](https://upterm.dev/) and [tmux](https://github.com/tmux/tmux/wiki). This setup facilitates real-time GitHub Actions debugging and allows seamless workflow continuation.

## Features

- **Interactive Debugging**: Gain SSH access to the GitHub Actions runner to diagnose and resolve real-time issues.
- **Workflow Control**: Resume workflows post-debugging without complete restarts, saving time and preserving state.

## Supported Operating Systems

- **Linux**
- **macOS**
- **Windows**: Unsupported (actions will skip to avoid pipeline failures).

## Getting Started

To set up an `upterm` session within your GitHub Actions workflow, use this example:

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup upterm session
      uses: owenthereal/action-upterm@main
```

Access the SSH connection string in the `Checks` tab of your Pull Request.

## Use Registered Public SSH Keys

To enhance security, you can restrict access to the `upterm` session to specific authorized GitHub profiles. First, ensure you have [added an SSH key to your GitHub profile](https://docs.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account).

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup upterm session
      uses: owenthereal/action-upterm@main
      with:
        limit-access-to-actor: true # Restrict to the user who triggered the workflow
        limit-access-to-users: githubuser1,githubuser2 # Specific authorized users only
```

If your registered public SSH key differs from your default private SSH key, specify the path manually: `ssh -i <path-to-private-key> <upterm-connection-string>`.

## Use Custom Upterm Server

To host your own Upterm server, follow the instructions for [deployment across various cloud providers](https://github.com/owenthereal/upterm?tab=readme-ov-file#hammer_and_wrench-deployment).
Configure the Upterm server with the `upterm-server` input parameter:

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup upterm session
      uses: owenthereal/action-upterm@main
      with:
        ## Use the deployed Upterm server via Websocket or SSH
        upterm-server: wss://YOUR_HEROKU_APP_URL
```

## Shut Down the Server if No User Connects

If no user connects, the server automatically shuts down after a specified time. This feature is handy for deploying `action-upterm` to provide a debug shell on job failure without unnecessarily prolonging pipeline operation.

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup upterm session
      uses: owenthereal/action-upterm@main
      if: ${{ failure() }}
      with:
        ## Shut down the server if unconnected after 5 minutes.
        wait-timeout-minutes: 5
```

## Continue a Workflow

To resume your workflow within an `upterm` session, create an empty file named `continue`:

```bash
touch continue
# or
sudo touch /continue
```

Press `C-b` followed by `d` (tmux detach command keys) to detach from the terminal without resuming the workflow.

## Usage Tips

### Resizing the tmux Window

After connecting via SSH:

- Press `control-b`, then type `:resize-window -A` and press `<enter>`

This will resize the console to the full width and height of the connected terminal.
([Learn more](https://unix.stackexchange.com/a/570015))
