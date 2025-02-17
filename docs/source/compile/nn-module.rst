PyTorch 2.0 NNModule Support
============================

**Author**: `Will Constable <https://github.com/wconstab>`_

`torch.compile` has special handling for torch.nn.Module objects, tracing them differently than it traces
arbitrary python classes, with the intent of producing faster code by making assumptions about the structure.

This doc describes some of the tradeoffs or edge cases that come up due to this specialization.

NNModule Hooks Support
----------------------
Previously, `torch.compile` had no support for hooks on nn.Modules, and if hooks were registered
they would simply be ignored in the compiled program. Indeed many users do not
use nn.Module hooks at all, or only use them for debug workflows, but there are valid use cases
for composing nn.Module hooks with `torch.compile`.

Hooks that are orchestrated via nn.Module.__call__ implementation include `_forward_pre_hooks`,
`forward_hooks`, `_backward_pre_hooks`, and `_backward_hooks`, and will be referred to as 'call hooks'.
These hooks are partially supported by `torch.compile` with limitations described below.

Another category of hooks includes `_state_dict_hooks` and its `pre` and `load_` variants, and are still
unsupported by `torch.compile`.

`nn.Module.__call__` Hooks Usage and limitations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
By default, `torch.compile` will trace the contents of `nn.Module.__call__` which means it will encounter
and run forward/pre-forward hooks.  If you install hooks before calling `torch.compile` and then do not remove
or alter the hooks later, your use case should be supported by default.

**skip_nnmodule_hook_guards**
By default, `torch._dynamo.config.skip_nnmodule_hook_guards` is set to True, meaning no guards will be installed
on each nn.Module hook dictionary, improving runtime by reducing guard execution time, at the cost of not noticing
if any hook dict is changed after compilation.

If you want to be able to remove or modify hooks after compilation and have `torch.compile` react appropriately
(by recompiling), then you need to set `skip_nnmodule_hook_guards=False` and expect a runtime penalty for the added
guards.

TODO: confirm if backward/pre_backward hooks are working or not and document accordingly

state_dict Hooks
~~~~~~~~~~~~~~~~
State dict hooks have not yet been supported in `torch.compile`.


TODO: warn_once if graph-breaking on hooks.  warn_once to point to this doc if hooks are present.