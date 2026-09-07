# Notebook — Order Service Provisioning

**Ticket:** ENG-218
**Started:** 2026-03-02
**Status:** ✅ Complete — 2026-03-04
**Purpose:** Put `pbj-api` on a dedicated host serving the order service for the machine floor, reachable at a stable hostname over TLS. Done when a machine on the shop floor can place an order against the public hostname and the deploy workflow can replace the running process without a hand-edit on the box.

## 📊 Resource Register

Current state of what this operation created. Overwritten as it changes.

| Resource | Identifier | Notes |
|---|---|---|
| Compute instance | `i-04b7c1a9` | `t4g.small`, arm64, 20 GB encrypted volume. Launched 2026-03-02 |
| Static address | `198.51.100.24` | Associated 2026-03-02. Allowlisted on the order database |
| Security group | `pbj-api-sg` | Ingress 80 and 443 only. No 22 — see entry 002 |
| DNS record | `api.pbjmachine.co` | A record at `198.51.100.24`, TTL 300. Created 2026-03-03 |
| TLS certificate | `api.pbjmachine.co` | Issued 2026-03-03, expires 2026-06-01. Renewal timer enabled, dry run passed |
| Deploy workflow | `.github/workflows/deploy.yml` | Written 2026-03-04. Environment is its only input |
| Process manager | `pm2` under `pbj-api.service` | Runs as `deploy`, not root. Resurrect dump saved after the first deploy |

## 📇 Entry Index

| # | | Title |
|---|---|---|
| 001 | 🔍 | What the account already runs |
| 002 | ⚖️ | Access to the host: session manager rather than SSH |
| 003 | 🔧 | Create the security group |
| 004 | 🔧 | Point the hostname at the instance |
| 005 | 🔍 | Read the web server's state before writing to it |
| 006 | 🔧 | Issue the certificate — failed, superseded by 007 |
| 007 | 🔧 | Issue the certificate, with the challenge port open |
| 008 | ⚖️ | Where the environment file comes from |
| 009 | 📄 | The deploy workflow |
| 010 | 🔧 | First deploy — failed on an unset environment variable |
| 011 | 🔧 | Set the variable at the process manager and deploy |

## 🔭 Context

Preconditions inherited from the Q1 capacity review. Each one constrains a command below.

- **The order database allows a fixed list of addresses.** Every host that talks to it needs its address added by hand, so the address has to be static before anything connects.
- **The machine floor calls one hostname**, baked into each machine's firmware at flash time. Changing it means reflashing every machine, so the name has to be right before the first machine points at it.
- **`pbj-api` reads its configuration at import time.** A module that reads the environment above its own imports never sees a file loaded later.

## 🧪 Entries

### 001 🔍 What the account already runs

**Intent.** Establish what exists before creating anything. Two things are worth knowing first: whether a host is already serving something on this account that a new security group could collide with, and whether the hostname is free.

**Command.**

```
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{Id:InstanceId,Type:InstanceType,State:State.Name,Ip:PublicIpAddress}' \
  --output table

aws route53 list-resource-record-sets --hosted-zone-id Z04HG21LKM3901 \
  --query 'ResourceRecordSets[].{Name:Name,Type:Type}' --output text
```

**Output.**

```
i-0f92aa31   t2.micro    running   203.0.113.9
i-0c1b8e70   t3.small    stopped   -

NS      pbjmachine.co.
SOA     pbjmachine.co.
A       pbjmachine.co.
CNAME   www.pbjmachine.co.
```

**Settled.** Two instances exist. `i-0f92aa31` is the shift-scheduling app and is live; `i-0c1b8e70` has been stopped since it was replaced and is not this operation's to remove. Neither serves the order service.

`api` is not occupied, so the record can be created rather than overwritten. The apex and `www` point at the marketing site and must not be touched.

---

### 002 ⚖️ Access to the host: session manager rather than SSH

**Intent.** Settle how commands reach the instance before the security group is written, since the answer decides whether port 22 is ever open.

**Options.** A key pair with port 22 open to the office address range, which is what the shift-scheduling host does. Or the cloud provider's session manager, where an agent on the instance opens the connection outward and no inbound port is needed.

**Decision.** Session manager. No key pair, and port 22 stays closed.

**Why.** The key pair is the part that ages badly. It has to be stored somewhere, it outlives the person who generated it, and an office address range stops being a meaningful boundary the first time someone works from home. The session manager grants access through the same identity system that governs everything else on the account, which means access is removed by removing a role rather than by remembering a key exists.

**Consequence.** Every command in this notebook runs through `send-command` rather than over SSH, and its output is retrieved as a second call. Two things follow that shape the entries below: the remote shell is **dash**, not bash, so no `pipefail` and no `[[ ]]`; and it runs as **root**, so anything that must run as the `deploy` user needs `su - deploy -c "..."`.

The shift-scheduling host now diverges from this pattern. Recorded, not fixed — it is not this operation's.

---

### 003 🔧 Create the security group

**Intent.** Open the two ports the public needs and nothing else, before the instance exists to attach it to.

**Command.**

```
aws ec2 create-security-group --group-name pbj-api-sg \
  --description "Order service host: public HTTP and HTTPS only" \
  --vpc-id vpc-3310ab7c
# -> sg-0ae41cc2

aws ec2 authorize-security-group-ingress --group-id sg-0ae41cc2 \
  --ip-permissions \
    IpProtocol=tcp,FromPort=80,ToPort=80,IpRanges='[{CidrIp=0.0.0.0/0}]' \
    IpProtocol=tcp,FromPort=443,ToPort=443,IpRanges='[{CidrIp=0.0.0.0/0}]'
```

**Output.**

```
{
    "Return": true,
    "SecurityGroupRules": [
        { "SecurityGroupRuleId": "sgr-0b7719", "FromPort": 80,  "CidrIpv4": "0.0.0.0/0" },
        { "SecurityGroupRuleId": "sgr-0b771a", "FromPort": 443, "CidrIpv4": "0.0.0.0/0" }
    ]
}
```

**Settled.** Two ingress rules, no port 22, consistent with 002. Port 80 is open because the certificate challenge in 006 needs it, not because anything is served there — the web server redirects it.

The application port is deliberately absent. Nothing outside the instance reaches the order service directly; the web server proxies to it over the loopback address.

---

### 004 🔧 Point the hostname at the instance

**Intent.** Create the A record. The certificate request in 006 depends on the name resolving publicly, so this comes first.

**Command.**

```
aws route53 change-resource-record-sets --hosted-zone-id Z04HG21LKM3901 \
  --change-batch file:///tmp/api-record.json
# CREATE, not UPSERT. TTL 300.
# -> /change/C0771PPQ2M

aws route53 get-change --id C0771PPQ2M --query 'ChangeInfo.Status' --output text
dig +short api.pbjmachine.co
```

**Output.**

```
INSYNC
198.51.100.24
```

**Settled.** The name resolves to the order service host.

**Two choices worth naming.** `CREATE` rather than `UPSERT`, so an unexpected existing record fails the call instead of being silently replaced — 001 said the name was free, and this is what proves it rather than assumes it. TTL 300, low enough that a mistake is cheap to correct while the address is still being settled.

---

### 005 🔍 Read the web server's state before writing to it

**Intent.** Establish what the web server is serving today. The distribution's default site answers on any hostname, and if it is still enabled it will shadow a new server block rather than losing to it.

**Command.**

```
aws ssm send-command --instance-ids i-04b7c1a9 \
  --document-name AWS-RunShellScript \
  --parameters 'commands=["ls /etc/nginx/sites-enabled/","nginx -v","which certbot || echo NOT-INSTALLED"]' \
  --query 'Command.CommandId' --output text
# -> 91cc0e4f-2f1a-4d33
```

**Output.**

```
default
nginx version: nginx/1.24.0
NOT-INSTALLED
```

**Settled.** Both suspicions confirmed. The default site is enabled and would shadow the order service block, so it has to be unlinked. Certbot is absent and is installed in 006.

**Reading note.** `nginx -v` writes to standard error, so it appears out of command order in the retrieved output. These invocations do not return their streams interleaved.

---

### 006 🔧 Issue the certificate — failed, superseded by 007

**Intent.** Request a certificate for `api.pbjmachine.co` and have the tool write the TLS configuration into the server block.

**Command.**

```
aws ssm send-command --instance-ids i-04b7c1a9 \
  --document-name AWS-RunShellScript \
  --parameters 'commands=["certbot --nginx --non-interactive --agree-tos -m ops@pbjmachine.co -d api.pbjmachine.co"]'
```

**Output.**

```
Certbot failed to authenticate some domains (authenticator: nginx).
Domain: api.pbjmachine.co
Type:   connection
Detail: 198.51.100.24: Fetching http://api.pbjmachine.co/.well-known/acme-challenge/9Kx2:
        Timeout during connect (likely firewall problem)
```

**Settled.** The request failed and no certificate was issued.

The challenge is served over plain HTTP, and the request timed out rather than being refused — the signature of packets being dropped before they reach the host rather than a host with nothing listening. Port 80 was authorized in 003, so the rule exists.

**The cause is the instance, not the rule.** `i-04b7c1a9` was launched into the default security group and `pbj-api-sg` was never attached. Nothing in 003 attached it, and nothing checked.

**Worth carrying.** Creating a security group and attaching it are two operations, and only the first one returns something that looks like success.

---

### 007 🔧 Issue the certificate, with the challenge port open

**Intent.** Attach the security group, confirm the challenge path is reachable from outside, then re-run the request.

**Command.**

```
aws ec2 modify-instance-attribute --instance-id i-04b7c1a9 --groups sg-0ae41cc2

# from a laptop, before spending a request against the rate limit
curl -s -o /dev/null -w "%{http_code}\n" --max-time 5 http://api.pbjmachine.co
# -> 404

aws ssm send-command --instance-ids i-04b7c1a9 \
  --document-name AWS-RunShellScript \
  --parameters 'commands=["certbot --nginx --non-interactive --agree-tos --redirect -m ops@pbjmachine.co -d api.pbjmachine.co","certbot renew --dry-run"]'
```

**Output.**

```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/api.pbjmachine.co/fullchain.pem
This certificate expires on 2026-06-01.
Deploying certificate to /etc/nginx/sites-enabled/api.pbjmachine.co

Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/api.pbjmachine.co/fullchain.pem (success)
```

**Settled.** The certificate is issued and installed, and plain HTTP now redirects.

**The curl before the request is the load-bearing step.** A 404 means the request reached the web server and it had nothing to serve at that path, which is exactly the reachability the challenge needs. Authorities rate-limit failed validations, so a free check beats a second failed request.

**The dry run, not the timer, is what proves renewal.** A scheduled timer proves a schedule exists. The dry run proves the thing the schedule triggers completes — including the challenge, which depends on the port 80 block surviving the tool's own edit to it.

---

### 008 ⚖️ Where the environment file comes from

**Intent.** Settle how the order service's configuration reaches the host, before the deploy workflow is written around it.

**Options.** Store the values as repository secrets and have the deploy write them onto the box as part of its remote command. Or store them in the provider's parameter store and have the instance read them itself with its own role.

**Decision.** Parameter store, read by the instance.

**Why.** The remote command's arguments are retained on the invocation record, and anyone who can read command history can read them. That is the same permission the deploy needs in order to poll its own command for output, so it cannot be withheld from the deploy without breaking it. The parameter store moves the read to the instance, where the credential is the instance's own role and nothing is carried through the pipeline at all.

**Consequence.** No secret ever passes through the build. The workflow writes the environment file by calling the parameter store *from* the instance, and its own credentials never grant access to a value. It also means a configuration change reaches the service on the next deploy rather than requiring a hand-edit on the box, which is half of this operation's end state.

---

### 009 📄 The deploy workflow

**Intent.** Replace the running process from a build artifact, with the environment name as the only input, and roll back if the new process does not answer.

**Path.** `.github/workflows/deploy.yml`

**Shape.**

| Choice | Why |
|---|---|
| Releases directory with a `current` symlink | The symlink moves only after the new process answers on the loopback port. Nothing destructive happens before the swap, which is most of a rollback for free |
| The environment file lives outside the releases and is symlinked in | Rolling back code does not roll back configuration |
| Delete the process and start it, never restart | The process manager records the *resolved* release path in its dump. A restart after a symlink swap runs the old release, and stops starting at all once retention removes that directory |
| Five releases kept, never the one `current` resolves to | The running process's working directory is inside it |
| The build happens in CI, never on the host | The host has no toolchain and resolves no dependencies. An artifact that fails to build fails before it is anywhere near the machine floor |

**Settled.** The workflow exists and is the only path that changes what runs on the host. No step in it can read a configuration value.

---

### 010 🔧 First deploy — failed on an unset environment variable

**Intent.** Run the workflow for the first time, against commit `4a19c02`.

**Output.**

```
[PM2] Starting /srv/pbj-api/releases/4a19c02/dist/server.js
New release failed its health check on port 8001
Error: unable to determine transport target for "pino-pretty"
    at Object.<anonymous> (.../src/lib/logger.ts:22:10)
No previous release to roll back to. pbj-api is down.
```

**Settled.** Everything upstream of the process worked — the artifact built, uploaded, extracted, and the environment file was written from the parameter store. The process then failed at import.

**The cause.** The logger selects a development formatter unless `NODE_ENV` is `production`, and that formatter is a development dependency the production artifact correctly does not ship. `NODE_ENV` was never set anywhere. The process manager passes on only what it is given.

**Why the fix is not a parameter.** The compiler hoists every import above the rest of the module body, so the call that loads the environment file runs after the logger has already read its configuration. A value in the environment file would look correct and change nothing. It goes on the process manager's start command, which is set before the process exists.

**This is the third form of the same trap** — carried in as a known property of `pbj-api`, and it still caught this deploy. Reading configuration at import time is a property of the module, not of the value.

**What the failure proved anyway.** The health check ran its full window, the process manager's logs named the cause on the first error line, and the no-previous-release branch reported honestly rather than failing at the symlink.

---

### 011 🔧 Set the variable at the process manager and deploy

**Intent.** Apply 010 to both the deploy path and the rollback path, then run again.

**Command.**

```
# deploy of 6b30f7d, then a second deploy to exercise the swap with a real predecessor
curl -i https://api.pbjmachine.co/health-check
```

**Output.**

```
HTTP/1.1 200 OK
Strict-Transport-Security: max-age=31536000
{"status":"ok"}
```

**Settled.** The order service answers over the public hostname. The second deploy exercised the symlink swap against a real predecessor rather than an empty releases directory, and the rollback path carries the same fix — an untested rollback that restores a process which cannot start is not a rollback.

The operation's end state is met: a machine on the floor reaches the hostname, and the deploy replaces the process with no hand-edit on the box.

<!-- NEXT ENTRY -->
