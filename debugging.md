# Debugging

## Reporting Issues

Any issues or bugs should be reported by opening
[a github issue](https://github.com/cloudfoundry/garden-runc-release/issues/new).

Please fill out the template and add any other information you think useful.
Logs are greatly appreciated.

If you would like to report sensitive issues then feel free to reach out to
team members on slack directly and we'll create a private tracker ticket for
that issue if necessary. We do strongly encourage you to consider opening a
public github issue.

## Gathering Logs

You can use garden
[ordnance-survery](https://github.com/cloudfoundry/garden-runc-release/blob/develop/scripts/ordnance-survey)
tool to gather most of the logs for general purpose debugging. The script can be
run directly from any machine with internet access like so:

```bash
curl bit.ly/garden-ordnance-survey -sSfL | bash
```
