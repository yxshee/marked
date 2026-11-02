<!-- claude-projects-attribution: {"github_login":"yxshee"} -->
_Requested by **Sir** · [project thread](https://claude.ai/code/project/chan_01FGhb5Fba7ZosnqFBVKeY7t?thread=cmsg_01FGhb5Fba7ZosnqFBVKeY7tB5HtdF6YvT4RNDg9dYg6Fi)_

`Compositor.Render()` sizes its canvas with `c.bounds.Dx()` and `c.bounds.Dy()`, but each layer is drawn at its absolute position. When no layer starts at the origin, the canvas is smaller than the area the layers cover, so they get clipped.

```go
c := lipgloss.NewCompositor(lipgloss.NewLayer("AAAA").X(2).Y(1))
fmt.Printf("%q\n", c.Render())
// before: ""
// after:  "\n  AAAA"

c = lipgloss.NewCompositor(
	lipgloss.NewLayer("Box\nBox").X(3).Y(2),
	lipgloss.NewLayer("x").X(1).Y(1).Z(1),
)
fmt.Printf("%q\n", c.Render())
// before: "\n x\n   Bo"
// after:  "\n x\n   Box\n   Box"
```

It only works today when some layer sits at (0, 0), because then the bounds start at the origin and `Dx`/`Dy` happen to equal `Max.X`/`Max.Y`.

The fix sizes the canvas from the origin to `bounds.Max`, clamped at zero. Layers at negative positions are still clipped at the left/top edge, same as before. `Draw`, `Hit` and `Bounds` are unchanged.

I checked the outputs above against v2 by rendering a canvas of `Max.X` by `Max.Y` from outside the package, and added `TestCompositorRenderOffsetLayers` in `canvas_test.go` (single offset layer, two offset layers, and a case with a layer at the origin that already worked).
