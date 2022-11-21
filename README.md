# Ansible Role: Borgbackup Outline

An Ansible Role that backups an instance of getoutline.com to a borgbackup repository. It can also import a directory of ZIP-archives.

This role only runs on the ansible controller.

[[_TOC_]]

## Requirements

* [borg](https://borgbackup.readthedocs.io/en/stable/installation.html)

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### Main Variables

    borgbackup_borgpath: "borg"

The optional variable `borgbackup_borgpath` sets the path to the borg executable.

    borgbackup_sshcommand

The optional variable `borgbackup_sshcommand` sets the command used to connect via SSH to the repository. This can be used to supply additional options to the SSH command.

    borgbackup_repo

The mandatory variable `borgbackup_repo` sets the URL to the borg repository.

    borgbackup_passphrase

The mandatory variable `borgbackup_passphrase` sets the passphrase to interact with the borg repository.

    borgbackup_archivenameformat: "{{ outline_domain }}-{{ _borgbackup_timeformat }}"

The optional variable `borgbackup_archivenameformat` sets the name format used for naming the borg archives.

    outline_domain

The mandatory variable `outline_domain` defines the Outline domain you want to backup (as a string). It usually looks something like `yoursubdomain.getoutline.com`.

    outline_token

The mandatory variable `outline_token` defines the Outline API token used to interact with the Outline API. The user owning this token should be able to view all collections you want to backup.

    outline_timeout: 120

The optional variable `outline_timeout` defines in seconds how long to wait for the export to finish.

    outline_exports_dir

The optional variable `outline_exports_dir` defines the path where downloaded ZIP-archives are stored (when using download or normal mode). In import mode it is also used as source directory where the export files will be expected (in this case it is mandatory).

    outline_delete_old_exports: yes

The optional variable `outline_delete_old_exports` controls whether existing exports should be deleted from Outline.

    outline_exports_pattern: "outlinebackup_yoursubdomain.getoutline.com_%Y-%m-%dT%H%M%S.%f.zip"

The mandatory variable `outline_exports_pattern` defines how to parse the name of the zip file when using import mode.

    # Possible values are `filename`, `atime`, `ctime`, `mtime`
    outline_exports_timesource: mtime

The optional variable `outline_exports_timesource` specifies the way this role determines the timestamp of the specific Outline export archive. `filename` lets you use the time information in the filename of the archive (`outline_exports_pattern` must be defined!). `atime`, `ctime` and `mtime` use the regarding file times of the archive file.

### Background variables

These variables define specific defaults which can be overriden, if wanted.

    _borgbackup_timeformat: "%Y-%m-%dT%H:%M:%S"

The optional variable `_borgbackup_timeformat` time format used by the borg `--timestamp` parameter (a [strptime](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior) string). This probably never needs to change.

    _outline_checkdelay: 7

The optional variable `_outline_checkdelay` defines the delay in seconds inbetween retries when checking if the Outline export has been completed.

    _outline_useragent: "ansible ({{ role_path|basename }} role by maxvalue)"

The optional variable `_outline_useragent` defines the value of the User-Agent header when interacting with the Outline API.

    _outline_timeformat: "%Y-%m-%dT%H:%M:%S.%f%z"

The optional variable `_outline_timeformat` defines the time format used by the Outline API. This pattern is used to parse the timestamps given by the API. This probably never needs to change.

### Internal variables

These are variables internally created by the role. Please do not use them in your code.

    outline_exportfile

This internal variable contains the path to the downloaded Outline export file. This can be used in conjunction with the download mode of the role to get the path of the file.

    _borgbackup_timestamp

The internal mandatory variable `_borgbackup_timestamp` sets the correct timestamp for the borg archive.

    _outline_archive

The internal mandatory variable `_outline_archive` is used to specify the path to the Outline export. Is used in normal, `import` and in `check_import` mode.

    _outline_downloadpath

The internal mandatory variable `_outline_downloadpath` is used to specify the directory path where the downloaded Outline export gets stored, in download and in normal mode.

    _result_borglist

The internal variable `_result_borglist` contains the JSON response from the `borg list` command which is used in import mode. It is used to see which Outline exports are already imported into the borg repository. The same variable name is also used for the same thing in `check_import` mode.

    _result_borglist_raw

The internal variable `_result_borglist_raw` contains the string response from the `borg list` command which is used in import mode. The same variable name is also used for the same thing in `check_import` mode.

    _result_checkexport

The internal variable `_result_checkexport` stores the response by the Outline API and is used to check if the Outline export is already in `complete` state. Later it is also used to get the timestamp of the export, when in normal mode.

    _result_createexport

The internal variable `_result_createexport` stores the response by the Outline API and is used to provide the ID when checking for completion of the export and when downloading the completed export.

    _result_deleteoldoutline

The internal variable `_result_deleteoldoutline` stores the response by the Outline API when deleting an old export. It is used to determine the correct `changed` state for the deletion task.

    _result_outlineexports

The internal variable `_result_outlineexports` stores the sorted list (filename ascending) of Outline export files which should be imported into the borg repository. The directory containing these export files must be specified via the mandatory `outline_exports_dir` variable. The same variable name is also used for the same thing in `check_import` mode.

    _result_outlineexports_unsorted

The internal variable `_result_outlineexports_unsorted` stores the unsorted list of Outline export files which should be imported into the borg repository. The directory containing these export files must be specified via the mandatory `outline_exports_dir` variable. This variable is not used directly, because the content needs to be sorted first.

    _result_tmpdir_borgcheckout

The internal variable `_result_tmpdir_borgcheckout` stores the path to the generated tmp directory for checking out an archive from the borg repository. This is only used in `check_import` mode.

    _result_tmpdir_unzip

The internal variable `_result_tmpdir_unzip` stores the path to the generated tmp directory for unarchiving the zipped Outline export file. This is used in normal, `import` and `check_import` mode.

    _result_tmpdir_zip

The internal variable `_result_tmpdir_zip` stores the path to the generated tmp directory for storing the downloaded Outline export. This is used in normal and `download` mode.

## Dependencies

* The community.general collection: `ansible-galaxy collection install --force community.general`

## Example Playbooks

### Normal usage

    ---
    # This creates an export, downloads it and creates a borg archive
    - hosts: localhost
      roles:
        - role: maxvalue.borgbackup_outline
          vars:
            outline_domain: myprettyoutlineinstance.outline.com
            outline_token: HEREISMYOUTLINETOKEN
            borgbackup_repo: borguser@borghost:22/path/to/borg/repo
            borgbackup_passphrase: HEREISMYBORGPASSPHRASE
    ...

### Import several zipped exports from a directory

    ---
    # This imports all archives in a given directory into a borg repository
    - hosts: localhost
      roles:
        - role:
            name: maxvalue.borgbackup_outline
            tasks_from: import
          vars:
            borgbackup_repo: borguser@borghost:22/path/to/borg/repo
            borgbackup_passphrase: HEREISMYBORGPASSPHRASE
            outline_exports_pattern: outline_myprettyoutlineinstance.outline.com_%Y-%m-%dT%H%M%S.%f.zip
            outline_exports_dir: /the/path/to/the/exports/dir
            outline_exports_timesource: filename
    ...

### Create an export and download it

    ---
    # This creates an export and downloads it
    - hosts: localhost
      roles:
        - role:
            name: maxvalue.borgbackup_outline
            tasks_from: download
            public: yes
          vars:
            outline_domain: myprettyoutlineinstance.outline.com
            outline_token: HEREISMYOUTLINETOKEN
            outline_exports_dir: /the/path/where/to/download/to
      tasks:
        - name: Print path to downloaded Outline export
          debug:
            var: outline_exportfile
    ...

## License

MIT

## Sponsors

You can support the development of this role and other similar roles by donating to one of the accounts below.

* [bymeacoffee](https://www.buymeacoffee.com/publicbetamax)
* [liberapay](https://de.liberapay.com/maxvalue/)
* [ko-fi](https://ko-fi.com/publicbetamax)
* [Patreon](patreon.com/publicbetamax)

## Author Information

This role was created in 2022 by Max Fuxjäger:
* Mastodon: @maxvalue@chaos.social
* Matrix: @ypsilon:matrix.org
