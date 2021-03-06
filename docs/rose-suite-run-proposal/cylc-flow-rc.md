# Proposal example of a new style `cylc-flow.rc` file

## References
[cylc-admin PR #40](https://github.com/cylc/cylc-admin/pull/40)
[cylc-admin work plan](../proposal-rose-suite-run.md)

## Purpose
As part of the work to transfer functionality from `rose suite run` to cylc
it is proposed that the options available in global, user and suite config
files are brought into alignment. This file sets out a specification for the
combined `cylc-flow.rc` file, although for the foreseeable future `suite.rc`
will also be read in the same way.

## Preamble

To reduce changes required for end users this file format is based on
`suite.rc`: Where aspects of the file are described as unchanged this implies
that file contents are exactly the same as they would be in a `suite.rc`.
Users of the `global.rc` are usually admins or power users and thus changes
to the format of this file are preferable.

It should be noted that although it will be possible to modify all settings in
all contexts, that some settings are more likely to be used in global contexts
and some are more likely to be used in suite configs. It has been proposed
that it ought to be possible for sysadmins to lock some settings in site
configurations.

****

## specification for `cylc-flow.rc`

### 1. Top level sections to come from from `suite.rc`

Most sites will leave these to users (Although I could imagine adding a
copyright message to meta by default, for example, and one user has suggested a
very simple runtime might be added too, for training and debugging purposes.)

These items are:
```ini
[meta]
[scheduling]
[vizualization]
```

* `[cylc]` will be renamed `[general]`

### 2. Top level sections from site configuration

It is likely that most users will continue to have these set by site admins.

#### 2.1 Small changes

* `[authentication]` becomes `[authorization]`

* The `[suite servers]` to be renamed `[suite run platforms]` for consistency
  with job platforms.

* `[test battery]` will be removed entirely.

* `[task events]` will be moved to `[runtime][[root]][[[events]]]`

#### 2.2 `[suite run platforms]`
This is the dictionary key formerly known as ``[suite servers]``. Changed only
for the purpose of keeping the name "platforms" conistent. This is expected to
be set only by system administrators. It should include the former top-level
section `[[suite host self-identification]]`.

#### 2.3 `[cylc]` -> `[general]`

`[cylc]` is be renamed `[general]`.

`global.rc[cylc]` at present contains a subset of the items available in
`suite.rc[cylc]` for this section so it is proposed that the new fill just has
the larger set. These items are:
```ini
[cylc]
  health check interval = 600
  task event mail interval = 300
  [[events]]
```

### 3 Job Platforms and the deprecation of `[runtime][[TASK]][[[job]]]host`

#### 3.1 `[job platforms]`
Many of the options in this section will be very similar to `[hosts]`
It is expected that these will mainly be set at site level, but that
small numbers of power users may wish to over-ride them.

```ini
[job platforms]
  [[example platform]]
    run directory =
    work directory =
    task communication method =
    submission polling intervals =
    execution polling intervals =
    scp command =
    ssh command =
    use login shell =
    login hosts =                         # list of possible login hosts
    batch system =                        # name of batch system
    cylc executable =
    global init-script =
    copyable environment variables =
    retrieve job logs =
    retrieve job logs command =
    retrieve job logs max size =      [[default directives]]

    retrieve job logs retry delays =
    task event handler retry delays =
    tail command template =
    [[batch systems]]
      [[__MANY__]]
        err tailer =
        out tailer =
        err viewer =
        out viewer =
        job name length maximum =
        execution time limit polling intervals =


    [[default directives]]                # This is probably something to do
      --some-directive="directive here!"  # sometime after cylc8
```

#### 3.2 Legacy Hosts behaviour
`[hosts]` will be deprecated but we need to keep many of its settings in
`[job platforms]`. For back compatibility host should re-direct to
`[runtime][[__MANY__]][[[platform]]]`. If the re-mapped `host` is part of a
cluster defined in `[job platforms]` then that job will use that cluster.
If a user wishes to over-ride this they can over-ride the
`[job-platforms][[PLATFORM]]` section.


### 4 Top level sections to merge in a more complex way

#### 4.1 `[[[job]]]` & `[[[remote]]]`
Old `[runtime][[__MANY__]][[[job]]]` & `[[[remote]]]`
sections to be merged and rationalized, being replaced by a new
`[runtime][[__MANY__]][[[job]]]` section.

We should select the platform defined by `[job platforms]`

```ini
[runtime]

[[job]]
  platform =                            
```

I think that we will probably want users to set this in the
[job platforms] section, leaving some of these options here as due-to-be
deprecated back compat over-rides which will give warnings if set?

```ini
    batch system =
    batch submit command template =
    execution polling intervals =
    execution retry delays =
    execution time limit =
    submission polling intervals =
    submission retry delays =
    host =
    owner =
    suite definition directory =
    retrieve job logs =
    retrieve job logs max size =
    retrieve job logs retry delays =
      [[[batch systems]]]
        err tailer =
        out tailer =
        err viewer =
        out viewer =
        job name length maximum =
        execution time limit polling intervals =
```

### Locking down global settings
There should be a mechanism by which system administators can lock global
settings for their sites. This should probably be a `lock=True/False` switches
within those settings that we wish to make lockable. If unset these will
default to `False`. If set to `True` a user who tries to over-ride that setting
will see a warning explaining that the setting has been over-ridden by a site
admin.
