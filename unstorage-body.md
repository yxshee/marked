<!-- claude-projects-attribution: {"github_login":"yxshee"} -->
_Requested by **Sir** · [project thread](https://claude.ai/code/project/chan_01FGhb5Fba7ZosnqFBVKeY7t?thread=cmsg_01FGhb5Fba7ZosnqFBVKeY7tB5HtdF6YvT4RNDg9dYg6Fi)_

`getKeys(base, { maxDepth })` counts depth from the storage root. That's what the "getKeys supports maxDepth with mixed native support" test checks: with a memory root and an `fs` mount at `storage_b`, `maxDepth: 1` returns `storage_b:file_depth1` but not `storage_b:depth1:file_depth2`.

When every mount supports `maxDepth` natively (for example `fs` at the root and another `fs` mounted at `b`), the post-filter is skipped and each driver applies `maxDepth` from its own root. Keys from the mounted driver then come back one level (or more, for nested mount points) deeper than asked for:

```js
const storage = createStorage({ driver: fs({ base: "./root" }) });
storage.mount("b", fs({ base: "./data" })); // ./data has a, sub/b, sub/deep/c
await storage.setItem("top", "x");

await storage.getKeys("", { maxDepth: 0 });
// before: ["b:a", "top"]      (b:a is depth 1)
// after:  ["top"]

await storage.getKeys("", { maxDepth: 1 });
// before: ["b:a", "b:sub:b", "top"]
// after:  ["b:a", "top"]
```

With a memory root (the default), the same calls already give the "after" results, because the non-native root turns on the post-filter. So the result depended on which drivers happen to be mounted.

The fix subtracts the mount point's depth from `maxDepth` before passing it to a mounted driver, and skips the driver when nothing below its mount point can be shallow enough. The root mount gets `maxDepth` unchanged. When the post-filter runs (mixed mounts), it sees the same keys as before, so that path is unchanged.

Checked with unstorage 2.0.0-beta.0 from npm (same `getKeys` code as `main`). Added a test next to the mixed-support one, with `fs` at the root and `fs` at `storage_b`.
