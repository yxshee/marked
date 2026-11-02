<!-- claude-projects-attribution: {"github_login":"yxshee"} -->
_Requested by **Sir** · [project thread](https://claude.ai/code/project/chan_01FGhb5Fba7ZosnqFBVKeY7t?thread=cmsg_01FGhb5Fba7ZosnqFBVKeY7tB5HtdF6YvT4RNDg9dYg6Fi)_

`JSONComparisonTest` in `yts/test_suite_test.go` decodes `in.json` with `encoding/json`, which turns every JSON number into a `float64`, and compares it with `reflect.DeepEqual` to the value go-yaml loads from `in.yaml`, where integers are `int`. So any case with an integer fails even when go-yaml loads it correctly, for example 229Q:

```
Expected: [map[avg:0.278 hr:65 name:Mark McGwire] ...]
Got     : [map[avg:0.278 hr:65 name:Mark McGwire] ...]
```

(`65` is `float64` on one side and `int` on the other.) Because of that, 38 `JSONComparisonTest` cases sit in `yts/known-failing-tests` and are skipped, so a real regression in those cases wouldn't show up.

This PR converts the integers in the loaded YAML value to `float64` before the comparison (`jsonNumbers`), and removes the 33 cases that now pass from `known-failing-tests`. The other 5 still differ for real reasons and stay listed:

- 565N: `!!binary` loads as the decoded bytes, `in.json` has the base64 text
- UGM3: the date loads as `time.Time`, `in.json` has a string
- 9MQT-01, DK95-01, DK95-06: the load result itself differs (error or different value)

Only the test harness changes; the library is untouched.
