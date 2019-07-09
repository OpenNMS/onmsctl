# onmsctl

A CLI tool for OpenNMS.

For now, only provisioning is implemented. Future releases can be able to control foreign source definitions, snmp configuration, send events, reload daemon configuration, list and search nodes, events, alarms, notifications, outages, among other features.

The reason for implemenging a CLI in Go is that the generated binaries are self contained, and for the first time, Windows users will be able to control OpenNMS from the command line.

The current alternative is `provision.pl` which relies on having `Perl` installed with some additional dependencies. This script only controls requisitions, whereas the scope of `onmsctl` is way wider.

## Compilation

1. Make sure to have [GO](https://golang.org/dl/) installed on your system.

2. Make sure to have Go Modules enabled

```bash
export GO111MODULE=on
```

3. Compile the source code for your desired operating system

For Linux:

```bash
GOOS=linux GOARCH=amd64 go build -o onmsctl onmsctl.go
```

For Mac:

```bash
GOOS=darwin GOARCH=amd64 go build -o onmsctl onmsctl.go
```

For Windows:

```bash
GOOS=windows GOARCH=amd64 go build -o onmsctl.exe onmsctl.go
```

For your own operating system, there is no need to specify parameters, as `go build` will be sufficient. Also, you can build targets for any operating system from any operating system, and the generated binary will work on itself, there is no need to install anything on the target device, besides copying the generated binary file.

## Usage

The binary contains help for all commands and subcommands by passing `-h` or `--help`. Everything should be self explanatory.

1. Build a requisition like you would do it with `provision.pl`:

```bash
➜ onmsctl inv req add Local
➜ onmsctl inv node add Local srv01
➜ onmsctl inv intf add Local srv01 10.0.0.1
➜ onmsctl inv svc add Local srv01 10.0.0.1 ICMP
➜ onmsctl inv cat add Local srv01 Servers
➜ onmsctl inv assets set Local srv01 address1 home
➜ onmsctl inv node get Local srv01
nodeLabel: srv01
foreignID: srv01
interfaces:
- ipAddress: 10.0.0.1
  snmpPrimary: S
  status: 1
  services:
  - name: ICMP
categories:
- name: Servers
assets:
- name: address1
  value: home

➜ onmsctl inv req import Local
Importing requisition Local (rescanExisting? true)...
```

2. You can build requisitions in YAML and apply it like K8s workload with `kubectl`:

```bash
➜ cat <<EOF | onmsctl inv req apply -f -
name: Routers
nodes:
- foreignID: router01
  nodeLabel: Router-1
  interfaces:
  - ipAddress: 10.0.0.1
  categories:
  - name: Routers
- foreignID: router02
  nodeLabel: Router-2
  interfaces:
  - ipAddress: 10.0.0.2
  categories:
  - name: Routers
EOF
```

The above also work for individual nodes:

```bash
➜ cat <<EOF | onmsctl inv node apply -f - Local
foreignID: www.opennms.com
interfaces:
- ipAddress: www.opennms.com
categories:
- name: WebSites
EOF

www.opennms.com translates to [34.194.50.139], using the first entry.
Adding node www.opennms.com to requisition Local...

➜ onmsctl inv node get Local www.opennms.com
nodeLabel: www.opennms.com
foreignID: www.opennms.com
interfaces:
- ipAddress: 34.194.50.139
  snmpPrimary: S
  status: 1
categories:
- name: WebSites
```

As you can see, it is possible to specify FQDN instead of IP addresses, and they will be translated into IPs before sending the JSON payload to the ReST end-point for requisitions.

Additionally, for convenience, if the `node-label` is not specified, the `foreign-id` will be used.

To configure the tool, or to avoid specifying the URL, username and password for your OpenNMS server with each request, you can create a file with the following content on `$HOME/.onms/config.yaml` or add the file on any location and create an environment variable called `ONMSCONFIG` with the location of the file:

```yaml
url: demo.opennms.com
username: demo
password: demo
```

Make sure to protect the file, as the credentials are on plain text.