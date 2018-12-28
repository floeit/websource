---
title: "Config"
date: 2018-05-14T17:05:08+01:00
anchor: "config"
weight: 130
---

### Common Config

All config has a Common section which has the following top level config items:

* `hosts`       - []string - All other floe Hosts
* `base-url`    - string - The api base url,  in case hosting on a sub domain.
* `config-path` - string - Is a path to the config which can be a path to a file in a git repo e.g. git@github.com:floeit/floe.git/build/FLOE.yaml
* `store-type`  - string - Define which type of store to use - memory, local, ec2
* `git-key`     - string - The private key to use with git, if empty then the system installed key is used. e.g. 

```
    git-key: "/home/ubuntu/.ssh/id_floedemo_rsa" 
```

### Flow Config

**A note on the workspace var**
Many field values will expand to include the workspace.  Any `{{ws}}` will be replaced by the absolute workspace path.
Task fields that start `./` (and are not `./...`) will also be replaced by the absolute workspace path, as will any value `.` on its own.

A flow has the following top level config items:

* `id` - string - url friendly ID - computed from the name if not given explicitly.
* `ver`- int    - Flow version, together with an ID form a global compound unique key.
* `name` - string - human friendly name for the flow - will show up in web interface.
* `reuse-space`	- bool - If true then will use the single workspace and will mutex with other instances of this Flow on the same host.
* `host-tags` - ([]string) - Tags that must match the tags on the host, useful for assigning specific flows to specific hosts.
* `resource-tags` - ([]string) - Tags that represent a set of shared resources that should not be accessed by two or more runs. So if any flow has an active run on a host then no other flow can launch a run if the flow has any tags matching the one running.
* `env`     - ([]string) - In the form of key=value environment variable to be set in the context of the command being executed, can include `{{ws}}` to expand to full absolute path - `.` at the start will be treated like `{{ws}}`.

* `flow-file` - string - the reference to a file that can be loaded as the pending run is generated, this file will override the config of the floe - so can be used like a jenkinsfile, three types of reference can be used...
    * `file` - load it from the local file system. e.g. `floes/floe.yaml`
    * `git` - do the shallowest clone of the repo specified and grab the content e.g. `git@github.com:floeit/floe.git/build/FLOE.yaml` in this case if the opts contain a ref then the ref (git ref - e.g. tag, branch etc.) will be used
    *  `fetch` - Fetch a file via http(s) e.g. `https://raw.githubusercontent.com/floeit/floe/redesign/confog.yaml`

### Triggers

Triggers are the things that start a flow off there are a few types of trigger.

* `data`     - Where a web request pushing data to the server may trigger a flow - for example the web interface uses this, to explicitly launch a run.
* `timer`    - A flow can be triggered periodically - as a timer does not contain any repo version info this can only include git 
* `poll-git` - A git repo will be polled for changes matching the parameters. One or more runs may be added to the pending queue.

### Trigger Types

#### data

Data triggers set up a http endpoint on the host that responds to a `POST` request on the `/data` url (e.g. http://localhost:8080/build/api/push/data )

Options:

* `cmd`     - Use this if you are running an executable that only depends on the 

### Tasks

All tasks have the following top level fields:

* `id`     - (string) A url friendly identity for this node, it has to be unique within a flow. If an id is not give then the name will be used to generate the ID.
* `name`   - (string) A human friendly name to display in the web interface. If a name is not given then one will be generated from the id (either a `name` or `id` must be given).
* `class`  - There are only two task classes:  
    * `task`  - A standard task does something - this is the default, and does not need to be in the config explicitly.
    * `merge` - A merge task waits for all or one of a list of events.
* `listen` - (string) The event tag that will trigger this 

Standard tasks (class `task`) have the following fields.

* `ignore-fail` - Only ever send an event tag containing the `good`postfix, even if the node failed. Can't be used in conjunction with `use-status`.
* `type`        - The specific type of task to execute.
    * `end`          - Special task that when reached positively indicates the flow ended.
    * `data`         - Accepts data from the web API, and is used to create web forms.
    * `timer`        - Waits a certain amount of time before firing its success event.
    * `exec`         - The main work horse, execute commands directly or via invoking a shell.
    * `fetch`        - Downloads a file over http(s).
    * `git-checkout` - Checkout a git repo
* `good`        - ([]int) The array of exit status codes considered a success. Default is `0` (an array of this one value)
* `use-status`  - (bool) If true then rather emit an event on task end containing the postfix `good` or `bad` use the actual exit code.
* `opts`        - (map) The variable map of options as needed by each `type`.

Merge tasks (class `merge`) have the following fields.

* `wait` - ([]string) - Array of event tags to wait for.
* `type` - The type of merge.
    * `all` - Wait for all events in the wait array.
    * `any` - Wait for any of the events in the wait array.

### Task Types

The specific task types and associated options. 

#### exec

The most common type of task node - executes a command, e.g. runs a mke command or a bash script.

Options:

* `cmd`     - Use this if you are running an executable that only depends on the binary, and does not use shell expansions in the args like '*', '.' etc.
* `shell`   - Use this if you are running something that requires the shell, e.g. bash scripts, or other sell extensions like '*', '.' etc. This version is generally more forgiving than 'cmd'.
* `args`    - An array of command line arguments - for simple arguments these can be included space delimited in the `cmd` or `shell` lines, if there are quote enclosed arguments then use this args array for them.
* `sub-dir` - The sub directory (relative to the run workspace) to execute the command in.
* `env`     - ([]string) - In the form of key=value environment variable to be set in the context of the command being executed.
* `git-key` - (string) - the path to a private key to use with this command. If you are mutating a remote repo (that is not the repo already used) using the command `git` then you probably want to include a specific key here (in Github a per repo deploy key may be in order.) This will override the common config git-key, and will be the setting used for all subsequent nodes, until the next occurrence of .

#### fetch

Downloads and caches a file from the web.

Options:

* `url`           - The URL to get the file from.
* `checksum`      - The checksum to validate the file.
* `checksum-algo` - What algorithm to use to compute the checksum `sha256`, `sha1` or `md5` are supported.
* `location`      - Where to link the file once downloaded - can use `{{ws}}` substitution. Relative paths will be relative to the workspace folder for the run in any case. If no location is given it will be linked to the root of the workspace. If the location ends in `/` (or `\` on some systems) then the file will be named as the download name, but linked to the location specified.

Example
-------

The following is the example config taken from the git repo...

```yaml


common:
    base-url: "/build/api" 
    store-type: local
    hosts:                          # defines the cluster topology
        - http://127.0.0.1:8080
        # - http://127.0.0.1:8090

flows:
    - id: build-project               # the name of this flow
      name: Build Project
      ver: 1
      reuse-space: false              # reuse the workspace (false) - if true /single used 
      resource-tags: [couchbase, nic] # resource labels that any other flows cant share
      host-tags: [linux, go, couch]   # all these tags must match the tags on any host for it to be able to run there

      triggers:                       # external events to subscribe to
        - name: push                  # name of this trigger
          type: git-push              # the type of this trigger
          opts:
            url: blah.blah            # which url to monitor

        - name: start
          type: data
          opts:
            form:
              title: Start
              fields:
                - id: ref
                  prompt: Ref (branch or hash)
                  type: text
                  value: master
                - id: from-ref
                  prompt: Ref to merge (branch or hash)
                  type: text 
            
      tasks: 
        - name: Checkout             # the name of this node 
          type: git-merge            # the task type 
          listen: trigger.good       # the event tag that triggers this node
          good: [0]                  # define what the good statuses are, default [0]
          ignore-fail: false         # if true only emit good
          opts:
            sub-dir: src/github.com/floeit
            url: git@github.com:floeit/floe.git
        
        - name: Echo
          type: exec
          listen: task.checkout.good
          opts:
            cmd: "echo"
            args: ["dan is a goon"]

        - name: Build                
          type: exec
          listen: task.checkout.good    
          opts:
            cmd: "ls"        # the command to execute 
            args: ["-lrt", "/"]

        - name: Test
          type: exec                 # execute a command
          listen: task.build.good    
          opts:
            shell: "sleep 10"         # the command to execute 

        - name: Sign Off
          type: data
          listen: task.build.good    # for a data node this event has to have occured before the data node can accept data
          opts:
            form:
              title: Sign off Manual Testing
              fields:
                - id: tests_passed
                  prompt: Did the manual testing pass?
                  type: bool
                - id: to_hash
                  prompt: To Branch (or hash)
                  type: string

        - name: Wait For Tests and Sign Off
          class: merge
          id: signed
          type: all
          wait: [task.echo.good, task.test.good, task.sign-off.good]

        - name: Final Task
          type: exec                 # execute a command
          listen: merge.signed.good
          opts:
            cmd: "ls"         # the command to execute 

        - name: complete
          listen: task.final-task.good
          type: end                 # getting here means the flow was a success 'end' is the special definitive end event

    - id: danmux
      ver: 1
      
      triggers:
        - name: start
          type: data
          opts:
            form:
              title: Start
              fields:
                - id: ref # ref is a magic name that the git-checkout task type will use
                  prompt: Git Ref (branch or hash etc)
                  type: text
        
        - name: Every 5 Minutes
          type: timer                # fires every period seconds
          opts:
            period: 300              # how often to fire in seconds

      flow-file: floe.yml            # get the actual flow from this file

```