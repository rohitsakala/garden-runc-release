# Firedrills

Listed here will be a collection of Garden firedrill event that we will
occasionally act out. If you're reading this for the first time then please
don't spoil the fun for yourself by reading on unless you have a particular
role to play in this drill.

## *The one where new apps couldn't be pushed*

### Role: Environment setup

<details>

This firedrill will involve abusing the os.RemoveAll bug we found in golang
in order to DoS a diego-cell of all container create "slots". You must:

1. spin up a Cloud Foundry using a version of Garden vulnerable to this issue
1. push 10 harmless apps (doras?) to simulate real apps
1. craft a docker image to reproduce the vulnerability
1. continue to push and delete apps until pushing eventually fails
1. bask in your hackery
1. read through the issue report below and get into character as the operator

</details>

### Role: garden engineers on-call

An operator of a public offering of cloud foundry is reporting that they are
unable to push new apps to their foundation. The error mentions something about
app limits but they are pretty sure that they only have around 10 apps pushed.
Speak with the operator to get access to the environment and try to figure out
mitigations, root cause and a fix for their problem.
