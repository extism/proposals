#  Lifecycle hooks

## Purpose

To provide hosts more insight into plugin resource usage.

## Solution

Provide a set of callbacks that are executed at specific points in the plugin's lifecycle. Where these callbacks are stored
and how they're executed is still up for discussion.

## Considerations

Some of these hooks could be implemented at the kernel-level - this would likely require hosts that use the kernel to implement
additional host functions to thread the configured host hook through. Kernel-level hooks could be used to track things like 
allocations in Extism memory and errors.

Other hooks, specifically around when plugin execution starts and stops, would have to be implemented in each runtime because the
kernel doesn't have that information.

There may also be more runtime-specific ways of implementing these hooks - for example, Wasmtime already provides a memory limiter
that could be used to implement hooks around memory use.  
