---
feature: (fill me in with a unique ident, my_awesome_feature)
start-date: 2025-01-03
author: Victor Engmark
co-authors: (find a buddy later to help out with the RFC)
shepherd-team: (names, to be nominated and accepted by RFC steering committee)
shepherd-leader: (name to be appointed by RFC steering committee)
related-issues: (will contain links to implementation PRs)
---

# Summary
[summary]: #summary

Warn NixOS users about manual configuration which no side effects. For example, if `services.nginx.enable` is `false` and `services.nginx.virtualHosts` is something other than the default value, the `services.nginx.virtualHosts` value has no effect on the system derivation. It would probably also be worth notifying when side effects might be _unexpected,_ such as creating files in the Nix store, but not using the files in the relevant programs.

# Motivation
[motivation]: #motivation

This would enable users to learn of configuration issues at build time, rather than at runtime. For example, consider an nginx admin who wants to migrate to NixOS:

1. They might start by [searching for `nginx` options](https://search.nixos.org/options?query=nginx).
2. They set the relevant options they recognise from their own configuration.
3. They apply the configuration.
4. No nginx server starts because `services.nginx.enable = false` by default.

Figuring out what went wrong, how to fix it, building the new configuration, and re-checking that it worked could take quite a while.

If instead they got a warning at step 3 saying that their configuration will only be applied once `services.nginx.enable = true`, they might fix it with minimal interruption to their workflow.

Why are we doing this? What use cases does it support? What is the expected outcome?

# Detailed design
[design]: #detailed-design

For simple cases such as services this could be implemented by changing the common `config = mkIf cfg.enable { … }` pattern to

```nix
config = if cfg.enable then
  { … };
else
  {
    warnings = someFunction {
      inherit cfg;
      opts = options.services.some-service;
    };
  }
```

We might want to have a build flag to skip these checks. 

This is the core, normative part of the RFC.
Explain the design in enough detail for somebody familiar with the ecosystem to understand, and implement.
This should get into specifics and corner-cases.
Yet, this section should also be terse, avoiding redundancy even at the cost of clarity.

# Examples and Interactions
[examples-and-interactions]: #examples-and-interactions

This section illustrates the detailed design.
This section should clarify all confusion the reader has from the previous sections.
It is especially important to counterbalance the desired terseness of the detailed design;
if you feel your detailed design is rudely short, consider making this section longer instead.

# Drawbacks
[drawbacks]: #drawbacks

False positives can happen, for example if someone combines a manual setup with files generated in the Nix store.

Built time overhead. 

# Alternatives
[alternatives]: #alternatives

We could change modules to do the semantic equivalent of enabling a configuration any time there are non-default values in that part of the configuration. For example, the nginx module might set `services.nginx.enable = lib.mkForce true` if any of the other `services.nginx.` options are set to non-default values. But this could result in surprising behaviour when defaults change.

What other designs have been considered? What is the impact of not doing this?
For each design decision made, discuss possible alternatives and compare them to the chosen solution.
The reader should be convinced that this is indeed the best possible solution for the problem at hand.

# Prior art
[prior-art]: #prior-art

You are unlikely to be the first one to tackle this problem.
Try to dig up earlier discussions around the topic or prior attempts at improving things.
Summarize, discuss what was good or bad, and compare to the current proposal.
If applicable, have a look at what other projects and communities are doing.
You may also discuss related work here, although some of that might be better located in other sections.

[Original discussion](https://matrix.to/#/!RRerllqmbATpmbJgCn:nixos.org/$cCm6NZbOKYEXMLrgmJGH2_y7LEx7fJsl4wPWe-k0BOo?via=nixos.org&via=matrix.org&via=tchncs.de)

# Unresolved questions
[unresolved]: #unresolved-questions

What parts of the design are still TBD or unknowns?

# Future work
[future]: #future-work

What future work, if any, would be implied or impacted by this feature without being directly part of the work?
