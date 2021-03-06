# On Cloud Configs
`cf-deployment` is developed against `bbl`.
This allows us to delegate the particulars
of the director's [Cloud Config][bosh-docs-cloud-config].
However, `cf-deployment` does not _require_ `bbl`.

This document discusses what `cf-deployment` requires
of any Cloud Config intended to work with it.
The discussion here is not exhaustive,
and is not under test.
Nonetheless, we intend it to support
those wishing to use `cf-deployment` without `bbl`.

## General Resources
The [BOSH docs for Cloud Config][bosh-docs-cloud-config]
are a great starting-place.
You may find the CPI-specific `cloud_properties` references
linked throughout to be useful.

`bbl` has [fixtures][cloud-config-fixtures]
for each `bbl`-supported IaaS.
These can be useful examples.
Anything `cf-deployment` draws from the Cloud Config
will be present among these fixtures.

If you're having trouble with a specific question,
please feel free to join the Cloud Foundry Slack,
and ask us in the `#cf-deployment` channel.

## VM Types
`cf-deployment` uses four VM types:

- minimal: this is sufficient for most things.
- small: our API instances can benefit from some additional resources
- small-highmem: diego cells need the memory to run apps.
- sharedcpu: some instances need only small, occasional resources.

It is important to note that all of these
have a 10 GB Ephemeral disk associated by default.
`cf-deployment` has encountered issues in the past
when attempting to use ephemeral disks
smaller than that.

## VM Extensions
We use VM Extensions to manage
non-default ephemeral disk sizes
and `cloud_properties` related to load balancing.

In particular, we generally require:
- `50GB_ephemeral_disk`
- `100GB_ephemeral_disk`
- `cf-router-network-properties`
- `cf-tcp-router-network-properties`
- `diego-ssh-proxy-network-properties`

While the disk extensions are straightforward,
load balancing is one of the details
that varies most between IaaSs,
so this may be one of the trickier parts
of writing your own Cloud Config.

## Networks
The network name `default`
is used throughout `cf-deployment`.
VMs on this network should be able to reach one another.
If Cloud Foundry is expected to be able to reach the internet,
this network will need some kind of NAT solution.
For example:
- On GCP, we assign an ephemeral external IP
  to each instance.
- On AWS, we have a NAT box
  and a corresponding routing rule
  which sends internet-bound traffic to it.

[bosh-docs-cloud-config]: https://bosh.io/docs/cloud-config.html
[cloud-config-fixtures]: https://github.com/cloudfoundry/bosh-bootloader/tree/master/cloudconfig/fixtures
