# Firedrills

Listed here will be a collection of Garden firedrill event that we will
occasionally act out. If you're reading this for the first time then please
don't spoil the fun for yourself by reading on unless you have a particular
role to play in this drill.

## *The one where new apps couldn't be pushed*

### Role: Environment setup

<details>

<summary>Show</summary>

<p>This firedrill will involve abusing the os.RemoveAll bug we found in golang
in order to DoS a diego-cell of all container create "slots". You must:</p>

<ol>
  <li>spin up a Cloud Foundry using a version of Garden vulnerable to this issue</li>
  <li>push 10 harmless apps (doras?) to simulate real apps</li>
  <li>craft a docker image to reproduce the vulnerability</li>
  <li>continue to push and delete apps until pushing eventually fails</li>
  <li>bask in your hackery</li>
  <li>read through the issue report below and get into character as the operator</li>
</ol>

<h4>Resources</h4>

<ul>
  <li><a href="https://www.cloudfoundry.org/blog/cve-2018-11084/">CVE report</a></li>
  <li><a href="https://www.pivotaltracker.com/story/show/159246405">tracker story</a></li>
</ul>

</details>

### Role: garden engineers on-call

An operator of a public offering of cloud foundry is reporting that they are
unable to push new apps to their foundation. The error mentions something about
app limits but they are pretty sure that they only have around 10 apps pushed.
Speak with the operator to get access to the environment and try to figure out
mitigations, root cause and a fix for their problem.
