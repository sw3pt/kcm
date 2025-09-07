# Kubernetes and OpenShift Config Manager

Creates per-terminal dependent k8s config files so that you don't switch context in one terminal and fire up commands in the wrong cluster in a different terminal.

Written to be used in zsh. Might also work in bash.

Requirements:
- `kubectl`
- `yq`
- `curl` - only for OpenShift OAuth login

## Usage

```
KCM - Kube Config Manager

Usage: kcm COMMAND

Commands:
  config        All commands related to managing/switching configs
  completion    Print completion scripts
  namespace     Switch Namespace
  oauth         Interact with OpenShift oauth
  setup         Initialize your workspace
  help          Displays this message
```

This can completely replace `oc login` with `kcm oauth login` so the oc client can't fuck up your kubeconfig. Using `oc login` or `oc project` will make your life worse with this because of how they interact with your kubeconfig.

For a better experience, use `kcm namespace` for switching namespaces because that updates a defaults file so you don't land in the `default` namespace every time you open a new shell/switch clusters. zsh completions also auto-complete namespaces dynamically.

## Setup 

ENV Vars:
- `KCM_CONFIG_DIR` - specify where the kube configs should be saved (default: `~/.kube/config-manager`)
- `KCM_CONFIG` - specify where config specifications are located (default: `$KCM_CONFIG_DIR/config-manager.yaml`)

To setup your terminal add the following to your `.zshrc`:
```
source <("$MYZSH"/plugin/kcm/kcm setup terminal)
```
That sets your `$KUBECONFIG` to the correct locations. And creates a kubeconfig for the current terminal.

Optional zsh auto-completion:
```
source <(kcm completion zsh)
```

⚠️ Run `kcm setup config` every time you change something in your config-manager.yaml


## config-manager.yaml

The `type` field is basically just for you. There is one special condition for OpenShift, when specified as "OpenShift-OAuth" you can actually do oauth login to your cluster.

Special Configuration for "OpenShift-OAuth":
- `.username`: Your username

For anything else you can just copy over `.cluster` and `.user` from your typical kubeconfig file, the whole content under each key is simply copied, so all typical flags should be supported. 

Example:
```
configs:
  - type: OpenShift-OAuth
    identifier: crc
    username: <YOUR USERNAME>
    cluster:
      certificate-authority-data: <BASE64 Encoded CA>
      server: https://api.crc.testing:6443
    user:
      token: <REDACTED>
  - type: k8s
    identifier: cluster-a
    cluster:
      certificate-authority-data: <BASE64 Encoded CA>
      server: https://cluster-a.example.com:6443
    user:
      client-certificate-data: <REDACTED>
      client-key-data: <REDACTED>
  - type: k3s
    identifier: cluster-b
    cluster:
      certificate-authority-data: <BASE64 Encoded CA>
      server: https://cluster-b.example.com:6443
    user:
      client-certificate-data: <REDACTED>
      client-key-data: <REDACTED>
last-config: crc
```