---
layout: default
title: Chapter 5 - Refactoring
nav_order: 2
parent: Egg timer
has_children: false
---

# Chapter 5 - Low button refactored

Updated to Gio 0.71 as of August 30th 2024

## Goals

The intent of this section is to organize the code better.

## Outline

Up to now, we've built the program by bolting on functionality, bit by bit. This has served us well, allowing us to start with an empty canvas, iterate by changing the minimum amount of lines, while still making meaningful progress.

Going forward however, it's starting to look a little unwieldy. Having all the code inside one big `main()` is can make it harder to understand, and harder to continue building. Hence we'll refactor the program a bit, simply breaking it up into smaller pieces.

> _Refactoring is transforming code in a safe and rapid way is vital to keeping it cheap and easy to modify for future needs._
> [Martin Fowler](https://martinfowler.com/books/refactoring.html)

In other words, no new functionality will be added, but we'll clear the way for better things to come.

## Code

### No 1 - `main()` is too long

Main is too long and does too much. It's better if `main()` starts and controls the program, but apart from that delegates to others. Here's the new one:

```go
func main() {
  go func() {
    // create new window
		w := new(app.Window)
		w.Option(app.Title("Egg timer"))
		w.Option(app.Size(unit.Dp(400), unit.Dp(600)))
    if err := draw(w); err != nil {
      log.Fatal(err)
    }
    os.Exit(0)
  }()
  app.Main()
}
```

Now, inside `main()` we create a window `w` as before, and immediately hand it over to a dedicated function `draw()`.

By storing the result of `draw()` in `err`, we can examine if the execution went well, and we can handle any errors in an orderly fashion.

For that we use [os.Exit()](https://pkg.go.dev/os?utm_source=gopls#Exit) and it's close cousin [log.Fatal(err)](https://pkg.go.dev/log?utm_source=gopls#Fatal). Both come from the standard library and are included as imports.

As mentioned before, the convention is that a zero exit code indicates success, which is what we send from `os.Exit(0)`if _err_ is nil. If not, we call `log.Fatal(err)` which prints the error message en exits with `os.Exit(1)`.

### No 2 - Constraints and Dimensions - A handy shortcut

We talked at length about **Constraints** and **Dimensions** earlier. Since we're using them quite a lot, it's handy to define two shortcuts, `C` and `D`. Constraints are part of the Context.

```go
type C = layout.Context
type D = layout.Dimensions
```

### No 3 - The `draw( )` function

A simplified version of `draw( )` shows the structure.

```go
func draw(w *app.Window) error {
    // ...

    // listen for events in the window.
    for {
        // detect what type of event
        switch e := w.Event().(type) {

        // this is sent when the application should re-render.
        case app.FrameEvent:
          // change typ to e
          gtx := app.NewContext(&ops, e)
            // ...
          // change typ to e
          e.Frame(gtx.Ops)
        // this is sent when the application is closed
        case app.DestroyEvent:
            return e.Err
        }
    }
}
```

As before examine all events. But since all we really care about is the type, let's simplify to `e: = w.Event().(type)`, to detect the type.

- `app.FrameEvent` is handled as before,
- we tidy the case for `app.DestroyEvent`, which returns _nil_ for normal window closures, but _Err_ if something else is the cause. The draw function should only inform what's been detected, not call the `os` directly.

## Comments

Refactoring is a matter of taste, and this is my take on it. If you have different needs, do what's right for your app. The main point is to keep your applications flexible enough to support continued improvements and future needs. Good luck.

---

[Next chapter](06_button_low_margin.md){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View it on GitHub](https://github.com/jonegil/gui-with-gio/tree/main/egg_timer){: .btn .fs-5 .mb-4 .mb-md-0 }
